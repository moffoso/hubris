I'll be following https://kubernetes.io/docs/tasks documentation

### Kubernetes
installing with native package management
```bash
sudo apt update 
sudo apt-get install -y ca-certificates curl
```

download the google-cloud public signing key
```bash
sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
```

add kubernetes apt repository
```bash
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

update the apt package index and install kubectl
```bash
sudo apt update
```

![unsigned_repository_kubernetes](IMG/unsigned_repository_kubernetes.png)
The key isn't found, this is due to the download location of the key not being where the docs say they are. According to a github post running following curl should fix it
```bash
sudo curl -fsSLo /etc/apt/keyrings/kubernetes-archive-keyring.gpg https://dl.k8s.io/apt/doc/apt-key.gpg
```

and finally install kubectl, kubeadm and kubelet
```bash
sudo apt install -y kubeadm kubectl kubelet
```

marking them so they dont upgrade automatically
```bash
sudo apt-mark hold kubectl kubeadm kubelet
```

### Docker
installing docker
```bash
sudo apt install docker.io
```

start docker and check if it is running
```bash
sudo systemctl start docker
sudo systemctl status docker
```

### Deploying a kubeadm cluster
#### configuring docker for kubeadm
configure docker to use overlay2 storage and systemd
```bash
sudo mkdir -p /etc/docker
cat <<EOF | sudo tee /etc/docker/daemon.json
{
	"exec-opts": ["native.cgroupdriver=systemd"],
	"log-driver": "json-file",
	"log-opts": {"max-size": "100m"},
	"storage-driver": "overlay2"
}
EOF
```

restart docker.service to apply the config
```bash
sudo systemctl restart docker
```

download cri-dockerd latest from https://github.com/Mirantis/cri-dockerd/releases
```bash
sudo curl -sSLo cri-dockerd_0.3.2.3-0.ubuntu-jammy_amd64.deb \
https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.2/cri-dockerd_0.3.2.3-0.ubuntu-jammy_amd64.deb
```

install cr-dockerd
```bash
sudo dpkg -i cri-dockerd_0.3.2.3-0.ubuntu-jammy_amd64.deb
```

memory swap seems to cause issues
```bash
# see if swap is enabled
swapon --show

# turn off swap
sudo swapoff -a

# disable swap
sudo sed -i -e '/swap/d' /etc/fstab
```

enable net.bridge.bridge-nf-call-iptables
```bash
sudo su
/etc/sysctl.d/bridge.conf
net.bridge.bridge-nf-call-iptables = 1 # changed from 0 to 1
```

create the cluster
```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --cri-socket=unix:///var/run/cri-dockerd.sock
```

configure kubectl to access the cluster
```bash
sudo mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config 
```

As this is a single-node cluster, the pods have to be untainted. Otherwise the pods will be stuck in a pending state
```bash
kubectl describe node vault | grep Taints # list taints
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

#### Installing a CNI (Cluster Network Interface)
I'll be installing flannel for this purpose
```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

### Deploying the Web UI
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml 
```

This setup allows the dashboard to be accessed from localost:8001, but i want to be able to access it over the hosts ip address. To achieve this i need to make following change to the kubernetes-dashboard service.
Change `type: ClusterIP` to `type: NodePort`
```bash
KUBE_EDITOR="nano" kubectl -n kubernetes-dashboard edit service kubernetes-dashboard # i want to use nano
```

then check which port the dashboard was exposed on
```bash
kubectl -n kubernetes-dashboard get service kubernetes-dashboard
```
https://[node-ip]:[node-port] is how you acces the dashboard

### Authentication
create a service account, rolbinding and token
```bash
kubectl create serviceaccount dan

kubectl create clusterrolebinding dan-binding --clusterrole cluster-admin --serviceaccount default:dan

kubectl create token dan

# create a token thats valid for 1 year
kubectl create token dan --duration=8760h
```
the token has to then be copy pasted

### Installing Helm
As detailed in https://helm.sh/docs/intro/install/ From Apt
```bash
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null

sudo apt-get install apt-transport-https --yes

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list

sudo apt-get update

sudo apt-get install helm
```

### Setting up persistent storage
made a directory in /media/vault127/kubestor01 called kdata01
create an indexfile to test access
sudo nano /media/vault127/kubestor01/kdata01/index.html
```bash
sudo sh -c "echo 'Hello from Kubernetes storage' > /media/vault127/kubestor01/kdata01/index.html"
# test
cat /media/vault127/kubestor01/kdata01/index.html
```

create the persistent volume
```yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: hp-volume01
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/media/vault127/kubestor01/kdata01"
```

```bash
kubectl apply -f /kubernetes_administratiion/storage-volumes/hostPath-volumes/hP-volume01.yml
```

get information about the persistent volume (pv)
```bash
kubectl get pv hp-volume01
```

create persistent volume claim
```yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: hp-volume01-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

```bash
kubectl apply -f hP-volume01-claim.yml 
```

get information about the persistent volume claim (pvc)
```bash
kubectl get pvc hp-volume01-claim
```
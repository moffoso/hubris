# Install and configure Kubernetes v1.28

Created 2 Virtual Machines

| name  | IP-adress    |
| ----- | ------------ |
| k8s-dev-master | 10.130.5.130 |
| k8s-dev-worker   | 10.130.5.131  |

### Configure ssh rsa key authentification
```bash
sudo copy-ssh-id master@10.130.5.130
sudo copy-ssh-id worker@10.130.5.131
```

### Install kubectl, kubeadm and kubelet
1. Update apt index to include Kubernetes apt repository
```bash
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl
```

2. download the public signing key
```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

3. add the v1.28 repository
```bash
# This overwrites any existing config in /etc/apt/sources.list.d/kubernetes.list
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

4. install the Kubernetes resources
```bash
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
# Mark the resources so they are not updated automatically
sudo apt-mark hold kubelet kubeadm kubectl
```

### Install containerd
I will be using containerd, docker works just aswell if you prefer
```bash
sudo apt install containerd -y
```

### Kernelmodules for preflight checks
1. turn off swap
```bash
cd /etc/fstab
# comment out the swapfile, save and exit
sudo swapoff -a
swapon --show
```

2. enable the kernelmodule [br_netfilter](/Wiki/Bugs/100-KVM/102-Libvirt_brnetfilter) and ipv4 forwarding
```bash
sudo modprobe overlay  
sudo modprobe br_netfilter  
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf  
overlay  
br_netfilter  
EOF
```

3. Configure containerd to use systemd as cgroup driver
```bash
sudo mkdir -p /etc/containerd  
sudo containerd config default | sudo tee /etc/containerd/config.toml  
sudo nano /etc/containerd/config.toml
#find the [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options] section and change systemdcgroup to trueSystemdCgroup = true
```

### Create the controle-plane with kubeadm
```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --cri-socket=unix:///var/run/containerd/containerd.sock
```

1. configure kubectl to access the cluster
```bash
sudo mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Install cilium as CNI
install cilium cli
```bash
CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/main/stable.txt)
CLI_ARCH=amd64
if [ "$(uname -m)" = "aarch64" ]; then CLI_ARCH=arm64; fi
curl -L --fail --remote-name-all https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
sha256sum --check cilium-linux-${CLI_ARCH}.tar.gz.sha256sum
sudo tar xzvfC cilium-linux-${CLI_ARCH}.tar.gz /usr/local/bin
rm cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
```

### Join worker to the cluster
in the control-plane get the join command and paste the generated command in the worker
```bash
kubeadm token create --print-join-command
```

### Add role worker to node
```bash
kubect label node k8s-dev-worker node-role.kubernetes.io/worker=worker
```


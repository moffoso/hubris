### k8s-dev-lb
##### Setup
Install nginx and nginx-extras
```bash
sudo apt install nginx nginx-extras
```

modify `/etc/nginx/nginx.conf` and add the following lines:
```bash
stream {
	upstream apiserver_read {
		# k8s-master-01
		server [master-01 ip-address]:6443;
		# k8s-master-02
		server [master-02 ip-address]:6443;
	}
	server {
		listen 6443;
		proxy_pass apiserver_read;
	}
}
```

### k8s-master-01
##### Initializing master
Using kubeadm we'll initialize the first control plane
```bash
sudo kubeadm init \
    --control-plane-endpoint "192.168.140.107:6443" \
    --upload-certs \
    --pod-network-cidr 10.244.0.0/16 \
    --apiserver-advertise-address=10.130.5.115
```

Output after initializing
```bash
Your Kubernetes control-plane has initialized successfully!To start using your cluster, you need to run the following as a regular user:

mkdir -p $HOME/.kube  
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config  
sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.  
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:  
[https://kubernetes.io/docs/concepts/cluster-administration/addons/](https://kubernetes.io/docs/concepts/cluster-administration/addons/)

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join 192.168.140.107:6443 --token xz...b0n \  
        --discovery-token-ca-cert-hash sha256:63...4 \  
        --control-plane --certificate-key 4ds...e
        
Please note that the certificate-key gives access to cluster sensitive data, keep it secret!  
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use  
"kubeadm init phase upload-certs --upload-certs" 

to reload certs afterward.Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.140.107:6443 --token xz...7b0n \  
        --discovery-token-ca-cert-hash sha256:63...4
```

> [!infobox]+
> If following preflight check fail use the commands appended below to resolve the issue
> `/proc/sys/net/bridge/bridge-nf-call-iptables does not exist`
> `/proc/sys/net/bridge/bridge-nf-call-iptables does not exist`

```bash
modprobe br_netfilter
echo '1' > /proc/sys/net/ipv4/ip_forward 
```

Configure kubectl profile
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Install CNI calico
```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```
### k8s-master-02
##### Join node as control plane
```bash
  kubeadm join 192.168.140.107:6443 --token xz...b0n \  
        --discovery-token-ca-cert-hash sha256:63...4 \  
        --control-plane --certificate-key 4ds...e
```
### k8s-worker-01
##### Join node as worker
```bash
kubeadm join 192.168.140.107:6443 --token xz...7b0n \  
        --discovery-token-ca-cert-hash sha256:63...4
```
### k8s-worker-02
##### Join node as worker
```bash
kubeadm join 192.168.140.107:6443 --token xz...7b0n \  
        --discovery-token-ca-cert-hash sha256:63...4
```



### client
After copying the kube config to the client you will get following error:
```bash
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /home/dan/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /home/dan/.kube/config
```

The fix is to change the permissions as follows:
```bash
chmod go-r ~/.kube/config
```



### client
After copying the kube config to the client you will get following error:
```bash
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /home/dan/.kube/config
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /home/dan/.kube/config
```

The fix is to change the permissions as follows:
```bash
chmod go-r ~/.kube/config
```

### Firewall rules
Here are the rules I used on the microk8s cluster nodes (using ufw):
```bash
sudo ufw allow 6443/tcp
sudo ufw allow 2379:2380/tcp
sudo ufw allow 10250/tcp
sudo ufw allow 10259/tcp
sudo ufw allow 10257/tcp
sudo ufw allow 10250/tcp
sudo ufw allow 30000:32767/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

As the new nodes are using debian 12 bookworm, I'll make use of the already built in nf_tables. As I used the netinst debian 12 image it doesnt have firewalld installed per default. To make sure this image version has nf_tables i ran the following command:
```bash
lsmod | grep -i nf_tables
``` 

Now I install firewalld
```bash
sudo apt install firewalld
```

On the masternodes allow these ports (includes all relevant microservices)
```bash
sudo firewall-cmd --permanent --add-port=22/tcp # allow ssh
sudo firewall-cmd --permanent --add-port=6443/tcp  
sudo firewall-cmd --permanent --add-port=2379-2380/tcp  
sudo firewall-cmd --permanent --add-port=10250/tcp  
sudo firewall-cmd --permanent --add-port=10251/tcp  
sudo firewall-cmd --permanent --add-port=10252/tcp  
sudo firewall-cmd --permanent --add-port=10255/tcp  
sudo firewall-cmd --permanent --add-port=8472/udp  
sudo firewall-cmd --add-masquerade --permanent
sudo firewall-cmd --permanent --add-port=30000-32767/tcp
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --permanent --add-port=443/tcp
```

restart firewalld
```bash
sudo systemctl restart firewalld
```

On the worker nodes add these rules
```bash
sudo firewall-cmd --permanent --add-port=22/tcp # allow ssh
sudo firewall-cmd --permanent --add-port=10250/tcp  
sudo firewall-cmd --permanent --add-port=10255/tcp  
sudo firewall-cmd --permanent --add-port=8472/udp  
sudo firewall-cmd --permanent --add-port=30000-32767/tcp 
sudo firewall-cmd --add-masquerade --permanent 
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --permanent --add-port=443/tcp
```

restart firewalld
```bash
sudo systemctl restart firewalld
```
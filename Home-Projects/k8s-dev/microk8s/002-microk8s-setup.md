### Create 2 Virtual Machines
during installation select microk8s

| name           | IP-adress    |
| -------------- | ------------ |
| k8s-dev-master | 10.130.5.133 |
| k8s-dev-worker | 10.130.5.134 |

### harden ssh access
```bash
sshd -T
```

In my case the Output was: /etc/ssh/sshd_config.d/50-cloud-init.conf

>[!warning]
> ```/etc/ssh/sshd_config.d/50-cloud-init.conf ```
> is a symlink for cloud.init
> https://cloudinit.readthedocs.io/en/21.1/topics/faq.html

add following lines to the file
```bash
PasswordAuthentication no
PermitRootLogin no
ChallengeResponseAuthentication no
UsePAM no
```

test the config with -o PubkeyAuthentication=no
```bash
ssh master@10.130.5.133 -o PubkeyAuthentication=no
```
### give user permissions
```bash
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube
newgrp microk8s
su - $USER
```

### add node to cluster
1. add the worker node to the control-plane node host file
```bash
sudo nano /etc/hosts

# add the following entry
k8s-dev-worker 10.130.5.134
```

2. get the add-node command from the control-plane node
```bash
microk8s add-node
```

3. add the node to the cluster with the generated command
```bash
microk8s join 10.130.5.133:25000/[token] --worker
```

### kubectl config
1. display config in the terminal
```
microk8s config
```

2. paste content in ~/.kube/config

### firewall
install ufw on both nodes
```bash
sudo apt install ufw
```

install rsyslog on both nodes for logging
```bash
sudo apt install rsyslog
```

create firewall rules
```bash
# ssh
sudo ufw allow 22/tcp
# services on default host interface
sudo ufw allow 16443/tcp # API server
sudo ufw allow 10250/tcp # kubelet
sudo ufw allow 10255/tcp # kubelet
sudo ufw allow 25000/tcp # cluster-agent
sudo ufw allow 12379/tcp # etcd
sudo ufw allow 10257/tcp # kube-controller
sudo ufw allow 10259/tcp # kube-scheduler
sudo ufw allow 19001/tcp # dqlite
sudo ufw allow 4789/udp # calico
```

create ipaddresspool
```yaml
# addresspool.yaml
---
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: custom-addresspool
  namespace: metallb-system
spec: 
  addresses:
  - 192.168.1.1-192.168.1.100
```


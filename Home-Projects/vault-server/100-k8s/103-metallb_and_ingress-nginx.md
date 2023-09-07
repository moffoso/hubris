<https://metallb.universe.tf/installation/
installed with helm
```bash
helm repo add metallb https://metallb.github.io/metallb
helm install metallb metallb/metallb --namespace metallb-system --create-namespace
# Output:
Now you can configure it via its CRs. Please refer to the metallb official docs
on how to use the CRs.
```

https://kubernetes.github.io/ingress-nginx/deploy/#quick-start
installed with helm
```bash
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

preflight-check
```bash
kubectl get pods --namespace=ingress-nginx
```

https://docs.okd.io/latest/networking/metallb/metallb-configure-address-pools.html
configure the ipaddresspool
```bash
ApiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default
  namespace: metallb-system
spec:
  addresses:
  - 10.130.5.200-10.130.5.210
  autoAssign: true
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default
  namespace: metallb-system
spec:
  ipAddressPools:
    - default
```

check status of pods and services 
```bash
kubectl get pods,services --all-namespaces
# note following service entry
ingress-nginx          service/ingress-nginx-controller             LoadBalancer   10.111.31.17    10.130.5.200   80:31789/TCP,443:30611/TCP   14m
```

### troubleshooting dns 
check if dns pods are running
```bash
kubectl -n kube-system get pods -l k8s-app=kube-dns
# expected output
NAME                       READY   STATUS    RESTARTS   AGE
coredns-799dffd9c4-6jhlz   1/1     Running   0          76m
```

check if dns service is present with the correct cluster-ip
```bash
kubectl -n kube-system get svc -l k8s-app=kube-dns
# expected output
NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)         AGE
service/kube-dns   ClusterIP   10.43.0.10   <none>        53/UDP,53/TCP   4m13s
```

check if domain names are resolving
```bash
kubectl run -it --rm --restart=Never busybox --image=busybox:1.28 -- nslookup kubernetes.default
# expected output
Server:    10.43.0.10
Address 1: 10.43.0.10 kube-dns.kube-system.svc.cluster.local

Name:      kubernetes.default
Address 1: 10.43.0.1 kubernetes.default.svc.cluster.local
pod "busybox" deleted
```

check if external names are resolving
```bash
kubectl run -it --rm --restart=Never busybox --image=busybox:1.28 -- nslookup www.google.com
# expected output
Server:    10.43.0.10
Address 1: 10.43.0.10 kube-dns.kube-system.svc.cluster.local

Name:      www.google.com
Address 1: 2a00:1450:4009:80b::2004 lhr35s04-in-x04.1e100.net
Address 2: 216.58.211.100 ams15s32-in-f4.1e100.net
pod "busybox" deleted
```

cant resolve "www.google.com"

so i connected to the coredns pods with following command
```bash
kubectl debug -it coredns-6b68c94794-lk5t9 -n kube-system --image=busybox:1.28 --target=coredns
```
from here i checked cat /etc/resolv.conf and ping www.google.ch and these worked without issue
i foudn the issue with following tool
```bash
kubectl run network --image wbitt/network-multitool
kubectl exec -i -t network -- sh
nslookup www.google.ch
```
this resulted in an error "unexpected source"

solution to the problem...
i reset the cluster
```bash
sudo -i
kubeadm reset
iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
ip link set cni0 down
ip link delete cni0 type bridge
systemctl stop kubelet
systemctl stop docker
iptables --flush
iptables -tnat --flush
systemctl start kubelet
systemctl start docker
exit
```
and configured it from anew
now it works...
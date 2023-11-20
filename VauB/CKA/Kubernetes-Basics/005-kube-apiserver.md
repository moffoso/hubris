Primary management componant in Kubernetes
When you run a `kubectl` command it communicates with the kube-apiserver. Let's have a look at how sending a get request or creating a pod actually happens.
##### Example kubectl get
```bash
kubectl get nodes
NAME            STATUS   ROLES           AGE   VERSION
k8s-master-01   Ready    control-plane   23h   v1.28.3
k8s-worker-01   Ready    <none>          23h   v1.28.4
k8s-worker-02   Ready    <none>          23h   v1.28.4
```
![](kubectl_get_nodes.png)
##### Example kubectl create
```bash
kubectl create pod
Pod created!
```
![](kuebctl_create_pod.png)
1. Authenticates user by using the certificate and key stored in ~/.kube/config
2. Validates the request before updating the ETCD (see Admission Controller)
3. Enters the data for the new pod in the ETCD
4. Returns that the ETCD has been updated with `Pod created!`
5. Schedules Pod on worker-node according to metadata
6. Passes information from Scheduler to kubelet on the specified node
7. Kubelet creates pod in CR
8. Kubelet returns status of the pod to the API

### kube-apiserver options
##### cluster built using kubeadm
The kube-apiserver will be a pod in the kube-system namespace. The pod definition file is located here: `/etc/kubernetes/manifests/kube-apiserver.yaml`
##### manually installed cluster
The kube-apiserver will run as a service in systemd, the options can be viewed here:
`/etc/systemd/system/kube-apiserver.service`




#### Container Orchestration
Hardwarefailures dont cause the Application to go down -> Multiple instances of the application running on different Nodes
User traffic is balanced among the containers -> During increased load  more instances of the application are deployed/ taken down if the load decreases

#### Architecture
See [Kubernetes](VauB/Introduction-Kubernetes/Docker/Kubernetes.md) 
Master vs Worker Nodes

Master | Worker | Desc
--------|------------- | -------------
</> Kube-apiserver | </> kubelet (agent) | the agent gathers information about the Worker Nodes
etcd keyvalue storage | | the gathered information is stored in etcd
controller | | 
scheduler | | 

#### Pods
Kubernetes cant deploy directly to Worker Nodes, instead the smallest object in Kubernetes is a Pod. A pod encapsulates Worker nodes for a specific application. A pod is a single instance of an application. Pods have a 1 - 1 relationship with containers. You cannot increase the number of containers in an existing pod. 
An example for this is a LAMP stack server. Within the Pod you'll have your Apache Container, a PHP Container and a SQL Container. All of these containers share the same network space and storage within the container and can communicate. If the pod is destroyed all containers in the pod are destroyed with it.

When you deploy a container with kubectl it automatically deploys a pod
```bash
kubectl run nginx --image=mginx
# inspect the pods
kubectl get pods
# more details about pod
kubectl describe pod nginx
```
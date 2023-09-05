get all
```bash
kubectl get all
```

create resource as soon as command is run
-> --dry-run
test the command but do not create resource
-> --dry-run:client

### PODs
get pods
```bash
kubectl get pods
```

get pod details
```bash
kubectl describe pod [podname]
```

create a pod
```bash
kubectl run [podname] --image=[imagename]
# append a label for example "tier: frontend"
kubectl run [podname] --image=[imagename] --labels=tier=db
# expose container port
kubectl run [podname] --image=[imagename] --port=[port]
```

create pod with definition file
```bash
kubectl create -f [definition file]
# create pod is a specific namespace
kubectl create -f [definition file] --namespace=[namespace]
```

edit pod properties directly
```bash
kubectl edit pod [podname]
```

extract pod definition file
```bash
kubectl get pod [podname] -o yaml > pod-defintion.yml
```

### Service
```bash
#imperativ way to create a service to expose the app on port 6754
kubectl expose pod redis --port=6754 --name redis-service
```

### ReplicaSet
get information on the replicationcontroller
```bash
kubectl get replicationcontroller
```

get information on the ReplicaSet
```bash
kubectl get replicaset
```

scale replicaset 
```bash
# after modifying defintion file replicas: [number of replicas]
kubectl replace -f [replicaset definition file]

# use scale command, note these wont modify the definition file
kubectl scale --replicas=[number of replicas] [replicaset definition file]
# or use the replicaset name
kubectl scale --replicas=[number of replicas] replicaset [replicaset name]
```

delete replicaset
```bash
# this will also delete all underlying PODs
kubectl delete replicaset [replicaset name]
```

### Deployment
create deployment with definition file
```bash
kubectl create -f [deployment definition file]
```

get deployment
```bash
kubectl get deployments
```

imperativ command
```bash
kubectl create deployment [deployment name] --image=[imagename] --replicas=[number of replicas]
```

### Namespace
create namespace
```bash
kubectl create namespace [namespace name]
```

if you want to get a pod from a specific namespace append --namespace=
```bash
kubectl get pods --namespace=[namespace]
```

change context in config to a specific namespace
```bash
kubectl config set-context $(kubectl config current-context) --namespace=[namespace]
```
per default the namespace is always "default"

get pods in all namespaces
```bash
kubectl get pods --all-namespaces
```

### Misc
change kubectl default editor
```bash
export KUBE_EDITOR=nano
```

### Formatting output with kubectl
```bash
kubectl [command] [kubernetes object] [name] -o [output format] [output filename]
```
-  -o json -> output a json formatted API object
-  -o name -> output only the resource name
-  -o wide -> output in the plain-text format with any additional information
-  -0 yaml -> output a yaml formatted API object



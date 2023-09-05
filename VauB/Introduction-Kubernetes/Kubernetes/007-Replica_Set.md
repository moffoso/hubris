### Replication controller
Ensures that a specified number of pods are running at all times. Acts as a loadbalancer.
```yaml
# rc-definition.yml
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  type: frontend
spec:
  template:
  # the content of the pod-defintion.yml is used as a template here
    metadata:
     name: myapp-pod
     labels:
       app: myapp
       type: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3
```
See [YAML](VauB/Introduction-Kubernetes/Kubernetes/YAML.md)

Create the replication controller
```bash
kubectl create -f rc-definition.yml
```

To view the list of replication controllers and the number of pods they are controlling
```bash
kubectl get replicationcontroller
```

to view the pods
```bash
kubectl get pods
```

### Replica set
```yaml
# replicaset-definition.yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    apps: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        - name: nginx-container
          image: nginx
  replicas: 3
  selector: # helps specifiy which pods belong to it with labels
    matchlabels:
      type: front-end
```
 
 run the replica set
```bash
kubectl create -f replicaset-definition.yml
```

View a list of replica sets
```bash
kubectl get replicaset
```

view the pods is the same as with replication controller

### Labels and Selectors
Replica set can be used to monitor existing pods by defining the matchlabels parameter.

### Scaling Replica set
Lets scale the replica set to 6 pods instead of 3. First modify the replicaset-definition.yml and change the line "replicas: 3" to "replicas: 6". Then run this command.
```bash
kubectl replace -f replicaset-definition.yml
```

To directly scale the number of pods without modifying the yaml use following command.
```bash
kubectl scale --replicas=6 -f replicaset-definition.yml
# alternativly use the type and name in the metadata under "kind: ReplicaSet"
kubectl scale --replicas=6 replicaset myapp-replicaset
```
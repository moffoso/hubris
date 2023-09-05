Kind | Version
--------------|-------------
POD | v1
Service | v1
ReplicaSet | apps/v1
Deployment | apps/v1

### POD
pod-definition.yml
```yml
apiVersion: v1
kind: Pod
metadata: # dictionary
  name: myapp-pod # stringvalue
  labels: # dictionary
    app: myapp
    type: front-end
spec:
  containers: # list/array
    - name: nginx-container
      image: ngnix
```
kubernetes expects certain parameters under metadata, labels can be whatever

### Replica Set
#### ReplicationController
rc-definition.yml
```bash
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: front-end
spec:
  template: # contains the information from the pod-definition.yml metadata
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

-> kubectl create -f rc.defintion.yml

get information on the replicationcontroller
```bash
kubectl get replicationcontroller
```

#### Replicaset
the difference here is the apiVersion and "selector" with the selector pods that were not created by the replicaset can be managed -> identified by matchLabels
replicaset-definition.yml
```bash
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
    type: front-end
spec:
  template: # contains the information from the pod-definition.yml metadata
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
  selector:
    matchLabels:
      type: front-end
```

create is the same as with ReplicationController

get information on the ReplicaSet
```bash
kubectl get replicaset
```

### Deployment
deployment-definition.yml
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
    app: myapp
    type: front-end
spec:
  template: # contains the information from the pod-definition.yml metadata
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
  selector:
    matchLabels:
      type: front-end
```

### Namespace
namespace-development.yml
```bash
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

### Resource Quota
limit resources to a namespace
compute-quota.yml
```bash
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```
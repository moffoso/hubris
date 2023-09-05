### Creating a deployment
Here we create a deployment-definition.yml
```yaml
# deployment-definition.yml
apiVersion: apps/v1
kind: Deployment
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
        - name: myapp
          image: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx
  replicas: 3
  selector: # helps specifiy which pods belong to it with labels
    matchlabels:
      type: front-end
```

run the deployment-definition.yml
```bash
kubectl create -f deployment-definition.yml
```

view the list of deployments
```bash
kubectl get deployments
```

the deployment automatically creates a replica set, to view it use
```bash
kubectl get replicaset
```

### Using an existing deployment yaml
If for example following 3 attributes of your deployment should be modified but not changed in the yml proceed as follows.
- Name: httpd-frontend
- Replicas: 3
- Image: httpd:2.4-alpine
```bash
kubectl create deployment httpd-frontend --image=httpd:2.4-alpine --replicas=3
```

verfiy if that the deployment go to a ready state
```bash
kubectl get deploy
```

### Rollout and Versioning
Whenever an update of an application occurs, for example nginx:1.7.0 to nginx:1.7.1 a new Revision is created.  This helps keep track of the changes.

Check the rollout status
```bash
kubectl rollout status deplyoment/myapp-deployment
```

see revision history
```bash
kubectl rollout history deployment/myapp-deployment
```

### Recreate vs Rolling Update
#### Recreate
Destroys every old version instance and then brings the new version instances up.
This leads to a downtime.

#### Rolling Update
Once old version instance is destroyed and replaced by a new instance. This iterates until all instances are of the new version. This results in no downtime and is the default for Kubernetes if no parameter is given.

### Update a deployment
Modify the deployment-defintion.yml
```yaml
# deployment-definition.yml
apiVersion: apps/v1
kind: Deployment
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
        - name: myapp
          image: front-end
    spec:
      containers:
        - name: nginx-container
          image: nginx:1.7.1 # added the new version tag here
  replicas: 3
  selector: # helps specifiy which pods belong to it with labels
    matchlabels:
      type: front-end
```

then run kubectl apply
```bash
kubectl apply -f deployment-defintion.yml
```

alternatively we can set the image of the deployment without modifying the yaml
```yaml
# careful this method is not persistent to the defining yaml
kubectl set image deployment/myapp-deployment \ nginx-container=nginx:1.7.1
```

To see detailed information about the deployment use
```bash
kubectl deployment myapp-deployment
# StrategyType will say either recreate or rolling update
```

### Rollback
```bash
kubectl rollout undo deployment/myapp-deployment
```

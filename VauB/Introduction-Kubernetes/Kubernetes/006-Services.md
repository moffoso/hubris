### NodePort
Listen to a port on a node and forward requests to the corresponding pod.

![NodePort Service](IMG/NodePort-Service.png)

Create the service defining yaml
```yaml
#service-definition.yml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service

spec:
  type: NodePort
  ports:
    - targetPort: 80
      port: 80
      nodePort: 30008
  selector: # to link the service to the correct pod use the corresponding labels
    app: myapp
    type: front-end 
```

create the service
```bash
kubectl create -f service-definition.yml
```

lists services
```bash
kubectl get services
```

When the service finds the label on multiple pods it will automatically select all 3 to forward requests to them.
If the Pods are on multiple clusters it will automatically creates a service that spans all clusters and maps the port the same.

### Cluster IP
Creates a virtual network between for example a set of front-end and back-end servers.

![ClusterIP_Service](IMG/ClusterIP_Service.png)
Create defining yaml
```bash
# service-definition.yml
apiVersion: v1
kind: Service
metadata:
  name: back-end
  ports:
    - targetPort: 80
      port: 80
  
  selector:
    name: myapp
    type: back-end
```

create the service
```bash
kubectl create -f service-definition.yml
```

check the service
```bash
kubectl get services
```

### Loadbalancer
Handles requests by sending them to the pod itself directly.

![Example_votingapp_loadbalancer](IMG/Example_votingapp_loadbalancer.png)

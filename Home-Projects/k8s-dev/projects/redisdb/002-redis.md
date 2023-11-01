### Deploy redis
add the repo
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

create redis-values.yaml by copying the default values yaml from Artifact Hub, in my case the only thing I modified was the persistence values -> storageClassName: vault127-hostpath

For setting up the hostPath provisioner check: https://microk8s.io/docs/addon-hostpath-storage

Export the generated password
```bash
export REDIS_PASSWORD=$(kubectl get secret --namespace test-redis test-redis -o jsonpath="{.data.redis-password}" | base64 -d)
echo $REDIS_PASSWORD
```

Port-forward the database to localhost
```bash
kubectl port-forward --namespace test-redis svc/test-redis-master 6379:6379
```
Create the redis-client pod to query the database directly
```bash
kubectl run --namespace test-redis redis-client --restart='Never'  --env REDIS_PASSWORD=$REDIS_PASSWORD  --image docker.io/bitnami/redis:7.2.2-debian-11-r0 --command -- sleep infinity
```

Execute into the redis-client
```bash 
 kubectl exec --tty -i redis-client \
   --namespace test-redis -- bash
```

Authenticate with the redis-db within the pod
```bash
REDISCLI_AUTH="insert your exported REDIS_PASSWORD here" redis-cli -h test-redis-master
```
### Interacting with go
add package github.com/redis/go-redis/v9
```bash
 go get github.com/redis/go-redis/v9  
```  

then the function main.go can be executed
```bash
go run main.go
```




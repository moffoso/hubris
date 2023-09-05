### General
view all created kubernetes objects
```bash
kubectl get all
```

Update a pod
```bash
kubectl edit pod [podname]
# make changes save and exit
# Output
"/tmp/kubectl-edit-[podid].yaml" 128L, 3983B
# to use the edit-file to recreate the pod
kubectl replace --force -f /tmp/kubectl-edit-[podid].yaml
# Output
pod [podname] deleted
pod/[podname] replaced
```

### Replica set
create a replica set according to the defining yaml
```bash
kubectl create -f replicaset-definition.yml
```

get a list of replica sets
```bash
kubectl get replicaset
```

delete a replica set
```bash
kubectl delete replicaset myapp-replicaset # this deletes all underlying Pods too
```

use an update defining yaml
```bash
kubcectl replace -f replicaset-definition.yml
```

directly scale the number of pods without modifying the yaml 
```bash
kubectl scale --replicas=6 -f replicaset-definition.yml
# alternativly use the type and name in the metadata under "kind: ReplicaSet"
kubectl scale --replicas=6 replicaset myapp-replicaset
```

### Deployment
rollout status
```bash
kubectl rollout status deployment/myapp-deployment
```

see revision history
```bash
kubectl rollout history deployment/myapp-deployment
```

create a deplyoment
```bash
kubectl create -f deployment-defintion.yml
```

see a list of deployments
```bash
kubectl get deployments
```

update a deployment
```bash
# after modifying the yaml
kubectl apply -f deployment-definition.yml
# non persistent change
kubectl set image deployment/myapp-deployment nginx=nginx:1.7.1
```

rollback to an earlier revision
```bash
kubectl rollout undo deployment/myapp-deployment
```

### command & args
In Kubernetes the command field overrides the ENTRYPOINT instruction and the args field overrides the CMD instruction of a Dockerfile. Ex:
```yaml
# pod-definition.yml

apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      command: ["sleep2.0"]
      args["10"]
```

### ENV Variables
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec:
  containers:
    - name simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080
      env:
        - name: APP_COLOR # Plain Key Value
          value: pink
```

Other methods of declaring ENV Variables:
```bash
# ConfigMap
env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
# Secrets
env:
  - name: APP_COLOR
    valueFrom:
      secretKeyRef:
```

### ConfigMap
Used to pass configurationdata in the form of key value pairs
##### Method 1: Imperative
```bash
k create configmap \
	[config-name] --from-literal=APP_COLOR=blue \ # key value pair
	              --from-literal=APP_MOD=prod     # APP_COLOR & blue
```

configmap from file
```bash
k create configmap
	[config-name] --from-file=[path to file]
```

##### Method 2: Declarative
```yaml
# config-map.yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```

### Secrets
##### Method 1: Imperative
```bash
k create secret generic \
	[secret-name] --from-literal=DB_Host=mysql \
	              --from-literal=DB_User=root
	              --from-literal=DB_Password=passwd
```

configmap from file
```bash
k create secret generic
	[secret-name] --from-file=[path to file]
```

##### Method 2: Declarative
```yaml
# secret-data.yaml

apiVersion: v1
kind: Secret
metadata:
  name: app-secret
data:
  DB_Host: bXlzcWw= # base64 encrypted 'mysql'
  DB_User: cm9vdA== # base64 encrypted 'root'
  DB_Password: cGFzc3dk # base64 encrypted 'passwd'
```

##### injecting env variables such as secrets into pod
```yaml
# pod-definition.yml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080
      envFrom:
        - secretRef:
            name: app-secret
```

### Docker Security
##### Process Isolation
Host has a namespace and the containers have a namespace. All processes running in a container are running the host, but in their own namespace. Docker containers can only see within their own namespace while the host can see all processes as if they were their own. 
Example with Ubuntu sleeper container:
```bash
docker run ubuntu sleep 3600

# from within the container
ps aux
# Output
USER   PID  %CPU  %MEM   VSZ  RSS  TTY STAT  START  TIME  COMMAND
root     1   0.0   0.0  4528  828  ?    Ss   03:06  0:00  sleep 3600

# from the host perspective
ps aux
USER   PID  %CPU  %MEM   VSZ  RSS  TTY STAT  START  TIME  COMMAND
dan   3307   0.1   0.1  7437  721  ?    Ss   09:06  0:00  sshd:dan@1
dan   3308   0.0   0.0  6456  345  ?    Ss   09:06  0:00  -bash
root  3309   0.0   0.0  3453  884  ?    Ss   09:06  0:00  docker
root  3310   1.0   0.0  4528  828  ?    Ss   09:06  0:00  sleep 3600
```

##### Linux capabilities
By default docker runs commands in containers as root. To change this use the user option
```bash
docker run --user=1000 ubuntu sleep 3600
ps aux
# Output
USER   PID  %CPU  %MEM   VSZ  RSS  TTY STAT  START  TIME  COMMAND
1000     1   0.0   0.0  4528  828  ?    Ss   03:06  0:00  sleep 3600
```

define the user in the Dockerfile
```yaml
FROM ubuntu

USER 1000
```

Linux capabilities can be found under /usr/include/linux/capability.h
Add linux capabilities to docker root for a specific container
```bash
docker run --cap-add [capability-name] [image]
```

remove linux cap
```bash
docker run --cap-drop [capability-name] [image]
```

### Security Context
In kubernetes if you configure the security settings at pod level, it will override the settings in the container. If you configure it in the pod and container it will override the settings of the pod.
adding security context to pod definition
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  securityContext: # pod level
    runAsUser: 1000
  containers:
    - name: ubuntu
      image: ubuntu
      command: ["sleep", "3600"]
      securityContext: # container level
        runAsUser: 1000
        capabilities:
          add: ["MAC_ADMIN"]
```
### Serviceaccount
using a different serviceaccount in a pod definition. The pod has to be deleted and recreated except for in a deployment -> this automatically causes a new rollout
```bash
# pod-definition.yml
apiVersion: v1
kind: Pod
metadata:
  name: my-kubernetes-dashboard
spec:
  containers:
    - name: my-kubernetes-dashboard
      image: my-kubernetes-dashboard
  serviceAccountName: dashboard-sa
```

using a secret to assign a token to a serviceaccount
```bash
# secret-definition.yml
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: mysecret
  annotations:
    kubernetes.io/service-account.name: dashboard-sa
```


### Resource Requests
The Task Scheduler looks for a node with the amount of resources available as described in the resource request of a pod definition. These resources are guaranteed to the pod
```yaml
# pod-definition.yml
apiVersion:
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080
      resources:
        requests:
          memory: "4Gi"
          cpu: 2
```

Resource definitions
- 1 CPU -> 1 vCPU or 1 Hyperthread
- Memory -> Defined in either Gigabytes or Gibibytes etc. (M/Mi ...)

##### limits
the container is throttled if it tries to surpass given limits, these are detailed in the pod definition
```yaml
# pod-definition.yml
apiVersion:
kind: Pod
metadata:
  name: simple-webapp-color
  labels:
    name: simple-webapp-color
spec:
  containers:
    - name: simple-webapp-color
      image: simple-webapp-color
      ports:
        - containerPort: 8080
      resources:
        requests:
          memory: "2Gi"
          cpu: 1
        limits:
          memory: "4Gi"
          cpu: 2
```

a container can use more memory than the limit, but if it does this constantly it will be terminated with the log OOM (Out Of Memory)
If memory is to be retrieved from a pod for the use of another the pod has to be killed

##### Default Behavior
No limits are set by default -> pods can use as many resources as they need

##### Limit Ranges
Configuring a default resource request/limit for all pods in a namespace
```yaml
# limit-range-cpu.yml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-resource-constraint-default
spec:
  limits:
    - default:
        cpu: 500m
      defaultRequest:
        cpu: 500m
      max:
        cpu: "1"
      min:
        cpu: 100m
      type: Container
```

analog for memory
```yaml
# limit-range-memory.yml
apiVersion: v1
kind: LimitRange
metadata:
  name: memory-resource-constraint-default
spec:
  limits:
    - default:
        cpu: 1Gi
      defaultRequest:
        cpu: 1Gi
      max:
        cpu: 1Gi
      min:
        cpu: 500Mi
      type: Container
```

##### Resource Quota
Setting a hard limit for a namespace that doesn't allow all pods together to use more than a certain quota of resources
```yaml
# resource-quota.yml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: default-resource-quota
spec:
  hard:
    requests.cpu: 4
    requests.memory: 4Gi
    limits.cpu: 10
    limits.memory: 10Gi
```

### Taints and Tolerations
By Default no pods tolerate a taint and cannot be placed on a tainted node.
Taints are set on nodes and alterations on pods.

##### Taints
```bash
k taint nodes [node-name] [key]=[value]:[taint-effect]
```

taint-effects
- NoSchedule -> Pods will not be scheduled on the node
- PreferNoSchedule -> System will try to avaid placing a pod on the node
- NoExecute -> existing pods will be evicted if they dont tolerate the taint

##### Tolerations
```yaml
# pod-definition.yml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: nginx-container
      image: nginx
  tolerations:
    - key: "[key]"
      operator: "equal"
      value: "[value]"
      effect: "[taint-effect]"
```

### Node Selectors
To make sure that processing intensive pods are assigned to a specific node, define the property nodeSelector in a pod definition. This will point to the label of a given node
```yaml
# pod-definition.yml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: data-processor
      image: data-processor
  nodeSelector:
    size: Large
```

to label a node
```bash
k label nodes [node-name] [label-key]:[label-value]
# Example
k label nodes node01 size=Large 
```

### Node Affinity
This feature allows pods to be placed in a node that matches one of the listed selectors
```yaml
# pod-definition.yml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: data-processor
      image: data-processor
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIngnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: size
                operator: In
                values:
                  - Large
                  - Medium
```

operators
- In -> schedule the pod in a node with this label
- NotIn -> schedule the pod in a node without this label
- Exists -> simple check if the label exists and place it there if it does

##### Node Affinity Types
Available in Kubernetes:
requriedDuringSchedulingIgnoredDuringExecution
preferredDuringSchedulingIgnoredDuringExecution

Planned implementation:
requiredDuringSchedulingRequiredDuringExecution

Definitions:
- requiredDuringScheduling -> pod created in a node with matching label
- preferredDuringScheduling -> if possible in node with matching label
- IgnoredDuringExecution -> changes to Node Affinity have no impact after scheduling was completed

### Multi-Container Pods
It is possible to have multiple containers within a pod, these are appended under the container section of a pod definition
```yaml
# pod-definition-yml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp
      ports:
        - containerPort: 8080
    - name: log-agent
      image: log-agent
```

##### Patterns
- SIDECAR -> example: logging service to collect logs from a webserver
- ADAPTER -> example: convert the logs to a common format
- AMBASSADOR -> communicate to different dbs at different stages

### Init Container
A multi-container pod is restarted any of the pods fail or run to completion.
To avoid this when running one time commands we can use initContainers
```yaml
# pod-definition.yml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
    - name: myapp-container
      image: busybox:1.28
      command: ['sh', '-c', 'echo The app is running && sleep 3600']
  initContainers:
    - name: init-myservice
      image: busybox:1.28
      command: ['sh', '-c', 'git clone [repository] ;']
```

multiple initContainers will be run in sequential order, if any fail kubernetes will restart the pod until they all work
```yaml
# pod-definition.yml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
    - name: myapp-container
      image: busybox:1.28
      command: ['sh', '-c', 'echo The app is running && sleep 3600']
  initContainers:
   - name: init-myservice
     image: busybox:1.28
     command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
   - name: init-mydb
     image: busybox:1.28
     command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```


### Readiness  and Liveness Probes
##### Readiness Probe
property of container in pod definition

http
```yaml
readinessProbe:
  httpGet:
    path: /api/ready
    port: 8080
```

tcp
```yaml
readinessProbe:
  tcpSocket:
    port 3306
```

command
```yaml
readinessProbe:
  exec:
    command:
      - cat
      - /app/is_ready
```

Example
```yaml
# pod-definition.yml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
    - name: simple-webapp
      image: simple-webap
      ports:
        - containerPort: 8080
      readinessProbe:
        httpGet:
          path: /api/ready
          port: 8080
```

##### Liveness Probe
periodically tests if the container is healthy
same syntax as readiness probe, restarts pod if given threshold is broken

http
```yaml
livenessProbe:
  httpGet:
    path: /api/healthy
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
  failureThreshold: 8
```

tcp
```yaml
livenessProbe:
  tcpSocket:
    port: 3306
```

command
```yaml
livenessProbe:
  exec:
    command:
      - cat
      - /ap/is_healthy
```

### Container Logging
steam logs of a specific container live
```bash
k logs -f [podname]
```

if there are multiple containers within a pod you have to specify shich container you want to see the logs from
```bash
k logs -f [podname] [containername]
```

### Monitoring
Metrics Server
cAdvisor is resposible for collecting metrics from kubelet and making them available to for example metrics server
to view the metrics
```bash
k top node # view node metrics
k top pod # view pod metrics
```

### Rolling Updates and Rollbacks
see the status of of a rollout
```bash
k rollout status [deployment-name]
```

see the revision history of rollouts
```bash
k rollout history [deployment-name]
```

##### Rollout Stratagy 1 Recreate
All pods are destroyed and then the new version is created. This causes downtime of the application.

##### Rollout Strategy 2 Rolling Update
Pods are destroyed and replaced one by one. This is the default update strategy of kubernetes. No downtime of the app

Rollouts are triggered by any changes to the deployment, be it file or through command

Rollback
```bash
k rollback undo [deplyoment-name]
```



### Restart Policy
by default the restart policy of a pod is set to "Always"
change the policy in the pod defintition
```yaml
# pod-definition.yml
apiVersion: v1
kind: Pod
metadata:
  name: math-pod
spec:
  containers:
    - name: math-add
      image: ubuntu
      command: ['expr', '3', '+', '2']
  restartPolicy: Never # another possibility is OnFailure
```

### Jobs
When we need to run a workload that is completed, and is not automatically restarted. These jobs are created with a job definition
the jobs are run sequentially until the specified number of completions is reached
```yaml
# job-definition.yml
apiVersion: batch/v1
kind: Job
metadata:
  name: math-add-job
spec:
  completions: 3 # this defines how many times to do the job
  template:
    spec:
      containers:
        - name: math-add
          image: ubuntu
          command: ['expr', '3', '+', '2']
      restartPolicy: Never
```

running jobs parallel
```yaml
# job-definition.yml
apiVersion: batch/v1
kind: Job
metadata:
  name: math-add-job
spec:
  completions: 3 # this defines how many times to do the job
  parallelism: 3 # run 3 conatiners for the job simultainiously
  template:
    spec:
      containers:
        - name: math-add
          image: ubuntu
          command: ['expr', '3', '+', '2']
      restartPolicy: Never
```

### CronJob
To create a Job on a repeating schedule use a CronJob definition
```yaml
apiVersion batch/v1
kind: CronJob
metadata:
  name: resporting-cron-job
spec:
  schedule: "*/1 * * *"
  jobTemplate: 
    spec:
      completions: 3
      parallelism: 3    
      template:
        spec:
          containers:
            - name: reporting-tool
              image: reporting-tool
          restartPolicy: Never
```

![Pasted image 20230706104204](IMG/Pasted-image-20230706104204.png)

### Services
NodePort
```yaml
# service-nodeport-definition.yml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  ports:
    - targetPort: 80 # containerPort
      port: 80 # port on the Service
      nodePort: 30010 # port on the Host
```

ClusterIP
```yaml
# service-clusterip-definition.yml
apiVersion: v1
kind: Service
metadata:
  name: back-end
spec:
  type: ClusterIP
  ports:
    - targetPort: 80
      port: 80
  selector:
    app: myapp
    type: back-end
```

### Network Policies
to limit ingress or egress traffic to a pod create a network policy definition
this example will limit traffic to a database to only a specific port
```yaml
# network-policy-definition.yml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
    - Ingress # All egress traffic isnt isolated as it isnt listed
  ingress:
    - from:
        - podSelector:
            matchLabels:
              name: api-pod
        ports:
          - protocol: TCP
            port: 3306
```

!! Flannel doesn't support Network Policies !!

assuming the api pod is not in the default namespace but in test
```yaml
# network-policy-definition.yml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
    - Ingress # All egress traffic isnt isolated as it isnt listed
  ingress:
    - from:
        - podSelector:
            matchLabels:
              name: api-pod
          namespaceSelector: # allow ingress traffic from ns test
            matchLabels:
              name: test
        ports:
          - protocol: TCP
            port: 3306
```

assuming the api pod isn't in the cluster but will be accessed over an IP
```yaml
# network-policy-definition.yml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
    - Ingress # All egress traffic isnt isolated as it isnt listed
  ingress:
    - from:
        - ipBlock:
            cidr: 192.168.0.113/24 # ingress is allowed from here
        ports:
          - protocol: TCP
            port: 3306
```

combining both will result in a network policy with following rule
allow ingress traffic if it comes from a pod with label api-pod and is in the ns test OR allow ingress traffic from the ip address 192.168.0.113/24 
```yaml
# network-policy-definition.yml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
    - Ingress # All egress traffic isnt isolated as it isnt listed
  ingress:
    - from:
        - podSelector:
            matchLabels:
              name: api-pod
          namespaceSelector: # allow ingress traffic from ns test
            matchLabels:
              name: test
        - ipBlock:
            cidr: 192.168.0.113/24 
        ports:
          - protocol: TCP
            port: 3306
```

Egress
```yaml
# network-policy-definition.yml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
    - Ingress # All egress traffic isnt isolated as it isnt listed
    - Egress
  ingress:
    - from:
        - ipBlock:
            cidr: 192.168.0.113/24 # ingress is allowed from here
        ports:
          - protocol: TCP
            port: 3306
  egress:
    - to:
        - ipBlock:
            cidr: 192.168.0.114/24 # egress is allowed to here
        ports:
          - protocol: TCP
            port: 80
        
```

### Storage Class
to automatically provision volumes (virtualdisks)
```yaml
# storage-class-definition.yml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: google.storage
provisioner: kubernetes.io/gce-pd
```

create a persistent volume claim that uses the storage class
```yaml
# pvc-definition.yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: google-storage
  resources:
    requests:
      storage: 500Mi
```

add the claim in the pod definition
```yaml
# pod-definition.yml
apiVersion: v1
kind: Pod
metadata:
  name:
spec:
  container:
    - image: alpine
      name: alpine
      command: ['/bin/sh', '-c']
      args: ['shuf -i 0-100 -n 1 >> /opt']
      volumeMounts:
        - mountPath: /opt
          name: data-volume
      volumes:
        - name: data-volume
          persistentVolumeClaim:
            claimName: myclaim
```

Parameters can be added to the storage class but these are extremely provisioner dependent

### Stateful Sets
Pods are created in a sequential order, the next pod is created if the prior one is successful. The pods are numbered reliably, the pods have a  sticky identity -> no more random names
This makes it possible to create for example a master database server and 2 slave database servers that communicate with mysql-0 (master) -> mysql-1 and mysql-2 are the slaves
```yaml
# statefulset-definition.yml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
  labels:
    apps: mysql
spec:
  template:
    metadata:
      labels:
        app: mysql
      spec:
        containers:
          - name: mysql
            image: mysql
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  serviceName: mysql-h # headless service
```

##### Storage in Statefulsets
to seperate the volumes so each pod has it's own storage in a stafulset add volumeClaimTemplate to the statefulset definition
```yaml
# statefulset-definition.yml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
  labels:
    apps: mysql
spec:
  template:
    metadata:
      labels:
        app: mysql
      spec:
        containers:
          - name: mysql
            image: mysql
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  serviceName: mysql-h # headless service
  volumeClaimTemplates:
    - metadata:
        name: data-volume
      spec:
        accessModes:
          - ReadWriteOnce
        storageClassName: google-storage
        resources:
          requests:
            storage: 500Mi
```

### Headless Services
this service does not have an ip of it's own and doesnt function as a loadbalancer. All it does is create dns entries for the pods in this format
```
podname.headless-service.namespace.svc.cluster-domain.example
```

thus in our example the master would be no matter how many times the pod is restarted
```
mysql-0.mysql.h.default.svc.cluster.local
```

headless service definition
```yaml
# headless-service-definition.yml
apiVersion: v1
kind: Service
metadata:
  name: mysql-h
spec:
  ports:
    - port: 3306
  selector:
    app: mysql
  clusterIP: none
```

the pod definition to utilize the headless service
```yaml
# pod-definition.yml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: mysql
spec:
  containers:
    - name: mysql
      image: mysql
  subdomain: mysql-h
  hostname: mysql-pod
```

!! if you use this in a deployment the pods will all have the same dns entry causing errors !!

In a statefulset add the headless service under spec.serviceName, subdomain and hostname are not necessary



### KubeConfig
this is the config of the kubectl commandline tool
```bash
$HOME/.kube.config
```

```yaml
apiVersion: v1
kind: Config
clusters:
  - name: test-cluster
    cluster:
      certificate-authority: ca.crt
      server: [server-address]
      
contexts:
  - name: test-cluster-admin@test-cluster # tells kctl which user 
    context:                              # for which cluster
      cluster: test-cluster
      user: test-cluster-admin

users:
  - name: test-cluster-admin
    user:
      client-certificate: admin.crt
      client-key: admin.key
```

to change your current context use
```bash
k config use-context  [users.name]@[clusters.name]
```


Checking the PodCIDR on each node (the PodCIDR is the range the Pods will get an IP address from)
```bash
kubectl get node -o jsonpath="{range .items[*]}{.metadata.name} {.spec.podCIDR}{'\n'}{end}" | column -t
# Example Output
kind-control-plane 10.244.0.0/24
kind-worker 10.244.1.0/24
kind-worker2 10.244.2.0/24
```

### Step 1 Configuration

The first step is to select a new CIDR for pods. It must be distinct from all other CIDRs in use.

For Kind clusters, the default is 10.244.0.0/16. So, for this example, we will use 10.245.0.0/16.

The second step is to select a different encapsulation protocol (Geneve instead of VXLAN for example) or a distinct encapsulation port.

For this example, we will use VXLAN with a non-default port of 8473 (the default is 8472).

We have pre-created a Cilium Helm configuration file values-migration.yaml based on the details above:

You can review it in theEditor tab or with the following command:

##### Create a Cilium Helm Config based on these Infos
```yaml
---
operator:
  unmanagedPodWatcher:
    restart: false
tunnel: vxlan
tunnelPort: 8473
cni:
  customConf: true
  uninstall: false
ipam:
  mode: "cluster-pool"
  operator:
    clusterPoolIPv4PodCIDRList: ["10.245.0.0/16"]
policyEnforcementMode: "never"
bpf:
  hostLegacyRouting: true
```

-  operator.unmanagedPodWatcher.restart: false
	 - this ensures that pods that are not being managed by cilium are not restarted automatically
- tunnelPort: 8473
	- this is optional in the case we use an encapsulation like in this Example but with a different port
- cni:
     customConf: true
     uninstall: false
	- the first setting skips writing the CNI configuraton (customConf: true) This is to prevent Cilium from taking over immediatly. The second will prevent the CNI configuration file and binary to be removed automatically (uninstall: false)
 - ipam:
      mode: "cluster-pool"
      operator:
        clusterPoolIPv4PodCIDRList: ["10.245.0.0/16"]
	- Defining the use of cluster-pool IPAM mode and a distinct PodCIDR during the migration
- policyEnforcementMode: "never"
	- Disables the enforcement of network policy until the migration is completed.
bpf:
  hostLegacyRouting: true
		- Route traffic via host stack to provide connectivity during migration


### Step 2 Installing Cilium
1. Create a new Helm values file called values-initial.yaml
2. Pull the non-default values from values-migration.yaml
3. Fill in the missing values through the use of helm-auto-gen-values

```bash
cilium install --helm-values values-migration.yaml --helm-auto-gen-values values-initial.yaml
# Example Output
🔮 Auto-detected Kubernetes kind: kind
✨ Running "kind" validation checks
✅ Detected kind version "0.14.0"
ℹ️  Using Cilium version 1.12.0
🔮 Auto-detected cluster name: kind-kind
🔮 Auto-detected datapath mode: tunnel
ℹ️  helm template --namespace kube-system cilium cilium/cilium --version 1.12.0 --set bpf.hostLegacyRouting=true,cluster.id=0,cluster.name=kind-kind,cni.customConf=true,cni.uninstall=false,encryption.nodeEncryption=false,ipam.mode=cluster-pool,ipam.operator.clusterPoolIPv4PodCIDRList[0]=10.245.0.0/16,kubeProxyReplacement=disabled,operator.replicas=1,operator.unmanagedPodWatcher.restart=false,policyEnforcementMode=never,serviceAccounts.cilium.name=cilium,serviceAccounts.operator.name=cilium-operator,tunnel=vxlan,tunnelPort=8473
ℹ️  Storing helm values file in kube-system/cilium-cli-helm-values Secret
ℹ️  Generated helm values file "values-initial.yaml" successfully written
```

review the created file
```yaml
# output of -> cat values-initial.yaml 
bpf:
  hostLegacyRouting: true
cluster:
  id: 0
  name: kind-kind
cni:
  customConf: true
  uninstall: false
encryption:
  nodeEncryption: false
ipam:
  mode: cluster-pool
  operator:
    clusterPoolIPv4PodCIDRList:
    - 10.245.0.0/16
kubeProxyReplacement: disabled
operator:
  replicas: 1
  unmanagedPodWatcher:
    restart: false
policyEnforcementMode: never
serviceAccounts:
  cilium:
    name: cilium
  operator:
    name: cilium-operator
tunnel: vxlan
tunnelPort: 8473
```

install cilium with helm
```bash
helm repo add cilium https://helm.cilium.io/
helm install cilium cilium/cilium --namespace kube-system --values values-initial.yaml
```

now cilium is installed and an overlay established, but no pods are managed by cilium itself. You can verify this with the cilium command
```bash
cilium status --wait
# Example Output
    /¯¯\
 /¯¯\__/¯¯\    Cilium:         OK
 \__/¯¯\__/    Operator:       OK
 /¯¯\__/¯¯\    Hubble:         disabled
 \__/¯¯\__/    ClusterMesh:    disabled
    \__/

Deployment        cilium-operator    Desired: 1, Ready: 1/1, Available: 1/1
DaemonSet         cilium             Desired: 3, Ready: 3/3, Available: 3/3
Containers:       cilium             Running: 3
                  cilium-operator    Running: 1
Cluster Pods:     0/13 managed by Cilium
Image versions    cilium             quay.io/cilium/cilium:v1.13.4@sha256:bde8800d61aaad8b8451b10e247ac7bdeb7af187bb698f83d40ad75a38c1ee6b: 3
                  cilium-operator    quay.io/cilium/operator-generic:v1.13.4@sha256:09ab77d324ef4d31f7d341f97ec5a2a4860910076046d57a2d61494d426c6301: 1
```

### Step 3 CliliumNodeConfig
The Cilium agent process supports setting configuration on a per-node basis instead of a constant configuration across the cluster. This allows overriding the global Cilium config for a node or set of nodes. It is managed by CiliumNodeConfig objects. Note the Cilium `CiliumNodeConfig` CRD was added in Cilium 1.13.

A `CiliumNodeConfig` object consists of a set of fields and a label selector. The label selector defines to which nodes the configuration applies.

Create a per-node config that will instruct cilium to tak over CNI networking on the node

```bash
cat <<EOF | kubectl apply --server-side -f -
apiVersion: cilium.io/v2alpha1
kind: CiliumNodeConfig
metadata:
  namespace: kube-system
  name: cilium-default
spec:
  nodeSelector:
    matchLabels:
      io.cilium.migration/cilium-default: "true"
  defaults:
    write-cni-conf-when-ready: /host/etc/cni/net.d/05-cilium.conflist
    custom-cni-conf: "false"
    cni-chaining-mode: "none"
    cni-exclusive: "true"
EOF
```

verify that it is applied
```bash
kubectl -n kube-system get ciliumnodeconfigs.cilium.io cilium-default -o yaml
# Example Output
apiVersion: cilium.io/v2alpha1
kind: CiliumNodeConfig
metadata:
  creationTimestamp: "2023-07-21T05:59:49Z"
  generation: 1
  name: cilium-default
  namespace: kube-system
  resourceVersion: "4199"
  uid: 8d272699-6dcc-47f5-8416-adf7bfe9f1e2
spec:
  defaults:
    cni-chaining-mode: none
    cni-exclusive: "true"
    custom-cni-conf: "false"
    write-cni-conf-when-ready: /host/etc/cni/net.d/05-cilium.conflist
  nodeSelector:
    matchLabels:
      io.cilium.migration/cilium-default: "true"
```

Initially this will not apply to any nodes as the CiliumNodeConfig only applies to Nodes with the `io.cilium.migration/cilium-default:"true"` label

Once the node is reloaded the custom Cilium configuration will be applied, the CNI configuration will be written and the CNI functionality will be enabled

### Step 4 Migration

cordon and drain the node so that end-users are not impacted by any potential issues

Cordon a node will prevent new pods from being scheduled on the node
Drain a node will gracefully evict all the running pods from the node. This ensures that the pods are not abruptly terminated and that their workload is gracefully handled by other available nodes

Set the node to start with
```bash
NODE="worker1"
```

Cordon the node
```bash
kubectl cordon $NODE
# Example Output
node/worker1 cordoned
```

Drain the node with the flag --ignore-daemonsets as several DaemonSets are still required to run
```bash
kubectl drain $NODE --ignore-daemonsets
```

verify that no pods are running on the node
```bash
kubectl get pods -o wide
```

label the node: this causes `CiliumNodeConfig` to apply to the node
```bash
kubectl label node $NODE --overwrite "io.cilium.migration/cilium-default=true"
```

restart Cilium on the node to create the CNI config file
```bash
kubectl -n kube-system delete pod --field-selector spec.nodeName=$NODE -l k8s-app=cilium
kubectl -n kube-system rollout status ds/cilium -w
# Example Output
kubectl -n kube-system rollout status ds/cilium -w
pod "cilium-qchdw" deleted
Waiting for daemon set "cilium" rollout to finish: 2 of 3 updated pods are available...
daemon set "cilium" successfully rolled out
```

restart the node

### Step 5 Verification
Check the cilium status
```bash
cilium status --wait
# Example Output
    /¯¯\
 /¯¯\__/¯¯\    Cilium:         OK
 \__/¯¯\__/    Operator:       OK
 /¯¯\__/¯¯\    Hubble:         disabled
 \__/¯¯\__/    ClusterMesh:    disabled
    \__/

Deployment        cilium-operator    Desired: 1, Ready: 1/1, Available: 1/1
DaemonSet         cilium             Desired: 3, Ready: 3/3, Available: 3/3
Containers:       cilium             Running: 3
                  cilium-operator    Running: 1
Cluster Pods:     0/15 managed by Cilium
Image versions    cilium             quay.io/cilium/cilium:v1.13.4@sha256:bde8800d61aaad8b8451b10e247ac7bdeb7af187bb698f83d40ad75a38c1ee6b: 3
                  cilium-operator    quay.io/cilium/operator-generic:v1.13.4@sha256:09ab77d324ef4d31f7d341f97ec5a2a4860910076046d57a2d61494d426c6301: 1
```

We are going to deploy a pod shortly. We will verify that Cilium attributes the IP to the pod.

Remember that we rolled out Cilium in cluster-scope IPAM mode where Cilium assigns per-node PodCIDRs to each node and allocates IPs using a host-scope allocator on each node. The Cilium operator will manage the per-node PodCIDRs via the CiliumNode resource.

The following command will check the CiliumNode resource and will show us the Pod CIDRs used to allocate IP addresses to the pods:
```bash
kubectl get cn kind-worker -o jsonpath='{.spec.ipam.podCIDRs[0]}'
```

Verify that, when we deploy a pod on the migrated node, that the pod picks an IP from the Cilium CIDR. The command below deploys a temporary pod on the node and outputs the pod's IP details (filtering on the Cilium Pod CIDR 10.245). Note we use the toleration to override the cordon.
```bash
kubectl run --attach --rm --restart=Never verify  --overrides='{"spec": {"nodeName": "'$NODE'", "tolerations": [{"operator": "Exists"}]}}'   --image alpine -- /bin/sh -c 'ip addr' | grep 10.245 -B 2
```

spin up a pod on the node to test
```bash
kubectl run --attach --rm --restart=Never verify  --overrides='{"spec": {"nodeName": "'$NODE'", "tolerations": [{"operator": "Exists"}]}}'   --image alpine/curl --env NGINX=$NGINX -- /bin/sh -c 'curl -s $NGINX '
```

If pods have connectivity we can uncordon the node
```bash
kubectl uncordon $NODE
```

check if the pods are managed by cilium
```bash
cilium status --wait
```

> Repeat step 4 & 5 for each Node

after all nodes are migrated this should be the output of cilium status
```bash
    /¯¯\
 /¯¯\__/¯¯\    Cilium:         OK
 \__/¯¯\__/    Operator:       OK
 /¯¯\__/¯¯\    Hubble:         disabled
 \__/¯¯\__/    ClusterMesh:    disabled
    \__/

Deployment        cilium-operator    Desired: 1, Ready: 1/1, Available: 1/1
DaemonSet         cilium             Desired: 3, Ready: 3/3, Available: 3/3
Containers:       cilium             Running: 3
                  cilium-operator    Running: 1
Cluster Pods:     15/15 managed by Cilium
Image versions    cilium             quay.io/cilium/cilium:v1.13.4@sha256:bde8800d61aaad8b8451b10e247ac7bdeb7af187bb698f83d40ad75a38c1ee6b: 3
                  cilium-operator    quay.io/cilium/operator-generic:v1.13.4@sha256:09ab77d324ef4d31f7d341f97ec5a2a4860910076046d57a2d61494d426c6301: 1
```

### Step 6 Clean-up

Now that Cilium is healthy, let's update the Cilium configuration. First, let's create the right configuration file:
```bash
cilium install --helm-values values-initial.yaml --helm-set operator.unmanagedPodWatcher.restart=true --helm-set cni.customConf=false --helm-set policyEnforcementMode=default --helm-auto-gen-values values-final.yaml
```

Again, we are using the `cilium-cli` to generate an updated Helm config file. As you can see from checking the differences between the two files, we are changing three parameters.

We are:
1. Enabling Cilium to write the CNI configuration file.
2. Enabling Cilium to restart unmanaged pods.
3. Enabling Network Policy Enforcement.

Apply it
```bash
helm upgrade --namespace kube-system cilium cilium/cilium --values values-final.yaml
kubectl -n kube-system rollout restart daemonset cilium
cilium status --wait
```

Removing Flannel as it is no longer needed
```bash
kubectl delete -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```


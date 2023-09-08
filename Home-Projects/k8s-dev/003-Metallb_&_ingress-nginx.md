### Add the helm repository and install metallb
```bash
helm repo add metallb https://metallb.github.io/metallb
helm install metallb metallb/metallb --namespace metallb-system --create-namespace
# Output:
Now you can configure it via its CRs. Please refer to the metallb official docs
on how to use the CRs.
```

### Install ingress-nginx with helm
```bash
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

### Configure ipaddresspool and l2advertisement
```bash
ApiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default
  namespace: metallb-system
spec:
  addresses:
  - 10.130.5.200-10.130.5.210
  autoAssign: true
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default
  namespace: metallb-system
spec:
  ipAddressPools:
    - default
```

check if LoadBalancer Service got a LAN IP-address
```bash
kubectl get pods,services --all-namespaces
```


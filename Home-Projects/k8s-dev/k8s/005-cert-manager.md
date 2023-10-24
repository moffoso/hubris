https://cert-manager.io/docs/installation/helm/

add repo
```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update
```

install cert-manager
```bash
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.12.0 \
  --set installCRDs=true
```


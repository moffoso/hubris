installation
```bash
microk8s enable cert-manager
```

after deinstallation couldnt delete namespace "testpage" this was due to a custom resource not having been removed 
https://github.com/cert-manager/cert-manager/issues/1582

remove the finalizer 
```bash
kubectl patch crd challenges.acme.cert-manager.io -p '{"metadata":{"finalizers": []}}' --type=merge
```
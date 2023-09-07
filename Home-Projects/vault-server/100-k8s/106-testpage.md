> cert manager v.1.12.0, ingress-nginx v.1.8.0

create cluster-issuer
```bash
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging-nginx
spec:
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: daniel.i.hendricken@gmail.com
    privateKeySecretRef:
      name: letsencrypt-issuer-nginx-key
    solvers:
    - http01:
        ingress:
          ingressClassName: nginx
```

nginx-server deployment
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: testpage 
  name: testpage
  labels:
    app: testpage
spec:
  replicas: 1
  selector:
    matchLabels:
      app: testpage
  template:
    metadata:
      labels:
        app: testpage
    spec:
      containers:
       - name: nginx-testpage
         image: nginx
         ports:
          - containerPort: 80
            name: http
         volumeMounts:
          - mountPath: usr/share/nginx/html
            name: testpage-volume
      volumes:
       - name: testpage-volume
         persistentVolumeClaim:
           claimName: testpage-claim 
```

storage volume and claim
```
piVersion: v1
kind: PersistentVolume
metadata:
  name: testpage-pv
  namespace: testpage
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: "1Gi"
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/media/vault127/kubestor01/kdata_testpage" #where to get index.html
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: testpage-claim
  namespace: testpage
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

service
```bash
apiVersion: v1
kind: Service
metadata:
  name: testpage-svc
  namespace: testpage

spec:
  selector:
    app: testpage
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

ingress
```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-staging-nginx
  name: testpage-ing
  namespace: testpage
spec:
  tls:
  - hosts:
    - test.moff.ch
    secretName: nginx-secret
  ingressClassName: nginx
  rules:
  - host: test.moff.ch
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: testpage-svc
            port:
              number: 80
```

debugging
```bash
# get logs for a pod
kubectl logs -f -n ingress-nginx ingress-nginx-controller-746675cb5f-k2dv9 

# curl through host and specific ip
curl -H "Host: test.moff.ch" https://10.130.5.200:443 -k
# -k ignores certificate 
```
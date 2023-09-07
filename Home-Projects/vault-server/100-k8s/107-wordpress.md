### WordPress deployment on k8s
created the namespace
```bash
k create ns wp
```

created a secret for the db
```bash
echo -n 'k23wpD0615!new' | base64
# output
azIzd3BEMDYxNSFuZXc=
# wp-secret.yml
apiVersion: v1
kind: Secret
metadata:
  name: wp-db-secrets
  namespace: wp
type: Opaque
data:
  MYSQL_ROOT_PASSWORD: azIzd3BEMDYxNSFuZXc=
  # k23wpD0615!new
```
apply the yaml
```bash
k create -f wp-secret.yml
```

create storage and claims for wp and db
```bash
# wp-storage.yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
  namespace: wp
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: "1Gi"
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/media/vault127/kubestor01/kdata-wp/mysql"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-claim
  namespace: wp
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: wordpress-pv
  namespace: wp
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: "10Gi"
  accessModes:
    - ReadWriteOnce
  hostPath: 
    path: "/media/vault127/kubestor01/kdata-wp/wp"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wordpress-claim
  namespace: wp
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```
apply the yml
```bash
k create -f wp-storage.yml
```

create the database deployment and db service
```bash
# wp-mysql-dep.yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: mysql
  namespace: wp
  labels:
    app: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: database
        image: mysql:5.7
        args:
          # mount volume
          - "--ignore-db-dir=lost+found"
        # add root password
        envFrom:
          - secretRef:
              name: wp-db-secrets
        ports:
          - containerPort: 3306
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-data
        persistentVolumeClaim:
          claimName: mysql-claim
---
apiVersion: v1
kind: Service
metadata:
  name: mysql-service
  namespace: wp
spec:
  ports:
  - port: 3306
    protocol: TCP
  selector:
    app: mysql
```
apply the yml
```bash
k create -f wp-mysql-dep.yml
```
enter the db pod and create a wordpress db
```bash
k describe pod -n wp
k exec -it [podname] -- bash
# in the pod
mysql -u root -p
CREATE DATABASE wordpress;
SHOW DATABASES;
EXIT
exit
```

create the deployment  and service for wordpress
```bash
# wp-wp-dep.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  namespace: wp
  labels:
    app: wordpress
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wordpress
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      containers:
        - name: wordpress
          image: wordpress:6.2.2-apache
          ports:
          - containerPort: 80
            name: wordpress
          volumeMounts:
            - name: wordpress-data
              mountPath: /var/www
          env:
            - name: WORDPRESS_DB_HOST
              value: mysql-service
            - name: WORDPRESS_DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: wp-db-secrets
                  key: MYSQL_ROOT_PASSWORD
            - name: WORDPRESS_DB_USER
              value: root
            - name: WORDPRESS_DB_NAME
              value: wordpress
      volumes:
        - name: wordpress-data
          persistentVolumeClaim:
            claimName: wordpress-claim
---
apiVersion: v1
kind: Service
metadata:
  name: wordpress-service
  namespace: wp
spec:
  selector:
    app: wordpress
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```
apply the yml
```bash
k create -f wp-wp-dep.yml
```

for the initial installation and config of wp we need a nodeport service
```bash
# wp-nodeport.yml
apiVersion: v1
kind: Service
metadata:
  name: wp-nodeport
  namespace: wp
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30010
  selector:
    app: wordpress
```
apply the nodeport service
```bash
k create -f wp-nodeport.yml
```

now wordpress is accessible in the browser with following url
```
[nodeip]:30010
```
install wordpress normally

To make wordpress accessible over the domain, have an A record pointing towards the public IP of the ingress-controller then create an ingress
```bash
# wp-ing.yml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod-ci
  name: wp-ing
  namespace: wp
spec:
  tls:
  - hosts:
    - www.moff.ch
    secretName: wp-secret
  ingressClassName: nginx
  rules:
  - host: www.moff.ch
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: wordpress-service
            port:
              number: 80
```
apply the ingress
```bash
k create -f wp-ing.yml
```

access your wordpress dashboard and under settings change 
WordPress Address -> http://www.example.com
Site Address -> http://www.example.com

### Troubleshooting non-accessible website in lan
curl from home network
```bash
curl -H "Host: www.moff.ch" https://10.130.5.200:443 -kv                                                                                
* Uses proxy env variable https_proxy == 'http://localhost:8888'
*   Trying 127.0.0.1:8888...
* Connected to localhost (127.0.0.1) port 8888 (#0)
* allocate connect buffer
* Establish HTTP proxy tunnel to 10.130.5.200:443
> CONNECT 10.130.5.200:443 HTTP/1.1
> Host: 10.130.5.200:443
> User-Agent: curl/8.0.1
> Proxy-Connection: Keep-Alive
> 
< HTTP/1.1 200 Connection established
< 
* CONNECT phase completed
* CONNECT tunnel established, response 200
* ALPN: offers h2,http/1.1
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
* ALPN: server accepted h2
* Server certificate:
*  subject: O=Acme Co; CN=Kubernetes Ingress Controller Fake Certificate
*  start date: Jun 14 10:57:22 2023 GMT
*  expire date: Jun 13 10:57:22 2024 GMT
*  issuer: O=Acme Co; CN=Kubernetes Ingress Controller Fake Certificate
*  SSL certificate verify result: self-signed certificate (18), continuing anyway.
* using HTTP/2
* h2h3 [:method: GET]
* h2h3 [:path: /]
* h2h3 [:scheme: https]
* h2h3 [:authority: www.moff.ch]
* h2h3 [user-agent: curl/8.0.1]
* h2h3 [accept: */*]
* Using Stream ID: 1 (easy handle 0x557b02403ea0)
> GET / HTTP/2
> Host: www.moff.ch
> user-agent: curl/8.0.1
> accept: */*
> 
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* old SSL session ID is stale, removing
< HTTP/2 200 
< date: Fri, 16 Jun 2023 09:12:46 GMT
< content-type: text/html; charset=UTF-8
< x-powered-by: PHP/8.0.29
< link: <https://www.moff.ch/wp-json/>; rel="https://api.w.org/"
< vary: Accept-Encoding
< strict-transport-security: max-age=15724800; includeSubDomains
```

curl from home network
```bash
 curl -v https://www.moff.ch                                                             
* Uses proxy env variable https_proxy == 'http://localhost:8888'
*   Trying 127.0.0.1:8888...
* Connected to localhost (127.0.0.1) port 8888 (#0)
* allocate connect buffer
* Establish HTTP proxy tunnel to www.moff.ch:443
> CONNECT www.moff.ch:443 HTTP/1.1
> Host: www.moff.ch:443
> User-Agent: curl/8.0.1
> Proxy-Connection: Keep-Alive
> 
< HTTP/1.1 503 Connect failed
< Content-Length: 6558
< Content-Type: text/html
< Cache-Control: no-cache
< Date: Fri, 16 Jun 2023 09:11:29 GMT
< Last-Modified: Wed, 08 Jun 1955 12:00:00 GMT
< Expires: Sat, 17 Jun 2000 12:00:00 GMT
< Pragma: no-cache
< Connection: close
< 
* CONNECT tunnel failed, response 503
* Closing connection 0
curl: (56) CONNECT tunnel failed, response 503

```

curl from external network works
```bash
 curl -v https://www.moff.ch                                          
* Uses proxy env variable https_proxy == 'http://localhost:8888'
*   Trying 127.0.0.1:8888...
* Connected to localhost (127.0.0.1) port 8888 (#0)
* allocate connect buffer
* Establish HTTP proxy tunnel to www.moff.ch:443
> CONNECT www.moff.ch:443 HTTP/1.1
> Host: www.moff.ch:443
> User-Agent: curl/8.0.1
> Proxy-Connection: Keep-Alive
> 
< HTTP/1.1 200 Connection established
< 
* CONNECT phase completed
* CONNECT tunnel established, response 200
* ALPN: offers h2,http/1.1
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
*  CAfile: /etc/ssl/certs/ca-certificates.crt
*  CApath: none
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
* ALPN: server accepted h2
* Server certificate:
*  subject: CN=www.moff.ch
*  start date: Jun 15 18:40:44 2023 GMT
*  expire date: Sep 13 18:40:43 2023 GMT
*  subjectAltName: host "www.moff.ch" matched cert's "www.moff.ch"
*  issuer: C=US; O=Let's Encrypt; CN=R3
*  SSL certificate verify ok.
* using HTTP/2
* h2h3 [:method: GET]
* h2h3 [:path: /]
* h2h3 [:scheme: https]
* h2h3 [:authority: www.moff.ch]
* h2h3 [user-agent: curl/8.0.1]
* h2h3 [accept: */*]
* Using Stream ID: 1 (easy handle 0x5651f846eea0)
> GET / HTTP/2
> Host: www.moff.ch
> user-agent: curl/8.0.1
> accept: */*
> 
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* old SSL session ID is stale, removing
< HTTP/2 200 
< date: Fri, 16 Jun 2023 09:23:17 GMT
< content-type: text/html; charset=UTF-8
< x-powered-by: PHP/8.0.29
< link: <https://www.moff.ch/wp-json/>; rel="https://api.w.org/"
< vary: Accept-Encoding
< strict-transport-security: max-age=15724800; includeSubDomains
```

within the local network www.moff.ch cannot be resolved, the solution for this is to set the domain to point to the local ip address it needs to connect to.

```bash
sudo nano /etc/hosts
# add following line to the device that wants to reach the site in the local network
10.130.5.200 moff.ch www.moff.ch
# 10.130.5.200 is the local ip of the loadbalancer
```

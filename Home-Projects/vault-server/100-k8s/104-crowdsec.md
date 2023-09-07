```bash
helm install crowdsec crowdsec/crowdsec -f crowdsec-values.yml -n crowdsec

NAME: crowdsec
LAST DEPLOYED: Fri Jun 16 23:07:05 2023
NAMESPACE: crowdsec
STATUS: deployed
REVISION: 1
NOTES:
Thank you for installing crowdsec.

## Local API URL
http://crowdsec-service:8080


## Dashboard information

You can access to the dashboard using :
http://crowdsec-service

The default credentials are : 
login : crowdsec@crowdsec.net
password : !!Cr0wdS3c_M3t4b4s3??
```

upgrade ingress-controller with lua plugin
```bash
helm upgrade --install ingress-nginx -f crowdsec-ingress-bouncer.yml ingress-nginx  --repo https://kubernetes.github.io/ingress-nginx --namespace ingress-nginx
```

```
cscli console enroll clizpuosb0000mk08e76z55zt
```
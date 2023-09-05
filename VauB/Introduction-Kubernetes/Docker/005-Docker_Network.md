#### Default networks
```bash
# Bridge
docker run ubuntu
# none
docker run ubuntu --network=none
# host
docker run ubuntu --network=host
```

Bridge connects all Containers within the bridge over a virtual network within this range per default: 172.17.0.2 - 17.17.0.255

Host is NAT, a webserver on the host network that is linked with port 5000 is reachable per hosts ip at port 5000

None means no network

#### Create a network
Creating a network in the docker host looks like this
```
docker network create \
	--driver bridge \
	--subnet 182.18.0.0/16
	custom-isolated-network
```

to list all existing networks on the docker host
```bash
docker network ls
```


#### Embedded DNS
All containers can resolve eachother by their names.


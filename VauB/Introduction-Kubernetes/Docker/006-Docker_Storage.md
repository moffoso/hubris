#### Filesystem
Per default docker stores all date under /var/lib/docker in here you'll find
- aufs
- containers
- image
- volumes

#### Layered architecture
Layer | first application | size | second application | size
---------------- | --------------|-------------|------------|--------------
1 | Base Ubuntu | 120 MB | already exists | 0
2 | Changes in apt packages | 306 MB | already exists | 0
3 | Changes in pip packages | 6.3 MB | already exists | 0
4 | Source code | 229 B | new Source code | 242 B
5 | Update Entrypoint | 1 B | new Update Entrypoint | 1 B

All of the Layers that are created by the docker build command (which are part of an image) are read only. Docker creates a new Layer called the Container Layer which is read/write. This Layer is destroyed and all data in it is lost if the container is destroyed. All containers made from the same image share the same Container Layer.

#### COPY-ON-WRITE
If you modify a part of the read only layer of a container the container makes a copy of that part in the read/write layer. The image needs to be rebuilt to make the change part of it

#### Volumes
Persistant datastorages can be created by appending following command (volume mount)
```bash
# to create a persistant sql database we first create a volume we'll store it in
sudo docker volume create data_volume
# this creates the directory data_volume under /var/lib/volumes
# then we mount data_volume to the default mysql storage directory in a container
sudo docker run -v data_volume:/var/lib/mysql mysql
```

To mount a directory outside of docker host provide the full path (bind mount)
```bash
# mounting the existing db in data
sudo docker run -v /data/mysql:/var/lib/mysql mysql
```

Again in a verbose method
```bash
sudo docker run \
--mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql
```


#### View how a image was built
To view how a docker image was build use following command
```bash
sudo docker history [container id]
```

#### Basics
run a container
```bash
sudo docker run [application]
sudo docker run redis:4.0 # tag that tells it to use the version 4.0 of the application redis
```
per default containers run in a non interactive mode
```bash
sudo docker run -i [application] # this puts the container into interactive mode
sudo docker run -it [application] # attach terminal and enter interactive mode
```

list containers
```bash
sudo docker ps # lists all running containers
sudo docker ps -a # lists all containers
```

inspect a container
```bash
sudo docker inspect [name] # this returns all details in a json format
```

logs
```bash
sudo docker logs [name]
```

stop a container
```bash
sudo docker stop [names] # optionally the ID can be used instead of the name
```

remove a container
```bash
sudo docker rm [names]
```

show list of downloaded images
```bash
sudo docker images
```

remove images
```bash
sudo docker rmi [repository] # make sure no containers are running off of this image
```

download an image
```bash
sudo docker pull [repository] # this pulls the image without running the container
```

Append a command
```bash
sudo docker run ubuntu sleep 5 # after the sleep is over the container is exited
```
containers are exited when no services are running in them

Run - attach and detach
```bash
sudo docker run kodekcloud/simple-webapp 
# this will run the container attached to your current prompt
# to run the container detached add the parameter -d
sudo docker -d run kodekcloud/simple-webapp
# to attach the container to the current prompt use the 4 first digits of the id
# in this example "e76e"
sudo docker attach e76e
```

#### Appended Commands
To open the bash command prompt in a docker OS
```bash
docker run -it centos bash # -it parameter logs you into the CentOS
```

execute a command in a running container
```bash
sudo docker exec [container id] cat /etc/*release* 
# cat /etc/*release* shows the current os version in this case it will show the containers os
```

add custom name to container
```bash
sudo docker run -d --name webserver nginx
# the container running nginx will have the name "webserver"
```

stop all running containers
```bash
sudo docker stop $(docker ps -a -q)
```

PORT mapping
```bash
docker run -p 80:5000 [webserver]
# this sends all traffic from port 80 on the docker host to port 5000 on the container
```

Volume mapping
```bash
docker run -v /opt/datadir:/var/lib/mysql mysql
# /opt/datadir is the external persistant storage that you link to the dockers storage
# which in this case is /var/lib/mysql
```
containers have their own isolated storage
#### Environment Variables

If you dont want to "hardcode" something like the backgroundcolor you need to set an envronment variable
```python
color = os.environ.get('APP_COLOR') # an example to set an external variable
```

i the dockerfile you'd call on it like this
```bash
export APP_COLOR=blue; python app.py
```

Append the variable defintion to the run command
```bash
sudo docker run -e APP_COLOR=blue my-custom-app
```

If you want to find the env variables of an existing container you can use the docker inspect command
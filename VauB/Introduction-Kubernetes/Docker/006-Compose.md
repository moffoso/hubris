#### Example of a docker compose yaml file
```bash
# assuming you want to set up following environment:
sudo docker run dhe/webapp
sudo docker run mongodb
sudo docker run redis:alpine
sudo docker run ansible
# a docker compose file would look like this:
services:
    web:
		image: "dhe/webapp"
	database:
		image: "mongodb"
	messaging:
		image: "redis:alpine"
	orchastration:
		image: "ansible"

# the entire docker stack can be brought up with this command:
sudo docker-compose up
```

#### Manually linking containers

> Manually linking is Deprecated!

Containers can be linked manually. As an example lets take voting-app thats to be linked to a database in which it stores the votes. 
```bash
# the database container used here is postgres
sudo docker run -d --name=db postgres
# -p [hostport]:[containerport]

sudo docker run -d --name=voting-app -p 5000:80 --link db:[db hostname] voting-app # --link [containername]:[hostname] of the container to be linked
```


#### Links in Compose
Assuming you would want to link the same componants together it would look like as follows in compose:
```bash
db:
	image: postgres:9.4
vote:
	image: voting-app
	ports:
	  - 5000:80
	links:
	  - db
```


#### Build in Compose
To build in compose all you have to do is replace "image:"  and add an "./" to the directory you want to build the image from
```
db:
	image: postgres:9.4
vote:
	build: ./vote
	ports:
	  - 5000:80
	links:
	  - db
```


#### Compose Version
In docker Compose Version 2 and up you have to specify it as follows
```bash
version: 2
services:
	db:
		image: postgres:9.4
	vote:
		build: ./vote
		ports:
		  - 5000:80
```
in version 2 and up compose creates a network for the applications ina stack allowing them to communicate among themselves easily (per container name) thus no links are needed


#### Networks
Lets make a Frontend and Backend Network. For this following lines are added to the yaml
```bash
version: 2
services:
	db:
		image: postgres:9.4
		netorks:
		  - back-end
	vote:
		build: ./vote
		ports:
		  - 5000:80
		networks:
		  - front-end
networks:
	front-end:
	back-end:
```


#### Example Error
```bash
Error: Database is uninitialized and superuser password is not specified.

# Following changes to the docker are necessary:

version: 2
services:
	db:
		image: postgres:9.4
		environment:
			POSTGRES_USER: postgres
			POSTGRES_PASSWORD: postgres
		networks:
		  - back-end
	vote:
		build: ./vote
		ports:
		  - 5000:80
		networks:
		  - front-end
networks:
	front-end:
	back-end:

# now the docker-compose up command can be used again
sudo docker-compose up
```

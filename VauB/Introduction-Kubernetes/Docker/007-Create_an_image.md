#### Example Webapp with Flask
```bash
FROM Ubuntu

RUN apt-get update && apt-get -y install python
RUN pip install flask flask-mysql # install python dependencies using pip
COPY . /opt/source-code # copy source code to /opt folder
ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run # run webserver with "flask" command

sudo docker build . -f Dockerfile -t rdio/my-custom-app
sudo docker push rdio/my-custom-app # sends it to the docker repository
```

#### Building the Webapp
Start an os image and make it interactive with a terminal
```bash
sudo docker run -it ubuntu bash
```

install dependencies
```bash 
# in the bash which will have this format in the terminal: root@[container id] #
apt-get update
apt-get install -y python

apt-get isntall python-pip

pip install flask
```

once that is done, place the source code where it should be etc. and configure
Ex: 
- copy the application code /opt/app.py
- execute following command in /opt: FLASK_APP=app.py flask run --host=0.0.0.0

#### Containerize the webapp
Using the command "history" collect all that was necessary to create the webapp

Create the Dockerfile
```bash
sudo mkdir my-custom-app
cd my-custom-app
cat > Dockerfile
FROM ubuntu

RUN apt-get update
RUN apt-get install -y python python-pip
RUN pip install flask

COPY app.py /opt/app.py 
# app.py has to be in the local directory of the dockerfile

ENTRYPOINT FLASK_APP=/opt/app.py flask run --host=0.0.0.0

# CTRL + C

sudo docker build . -t my-custom-app
```

#### CMD vs Entrypoint
As an example lets look at a dockerfile with following entries
```bash
FROM ubuntu

CMD sleep 5
```
The container created from this file always starts sleeps for 5 seconds and exits
CMD (command) uses this format: `CMD command param1`

We dont want the time for sleep to be hardcoded to 5, as such we use an Entrypoint
```bash
FROM ubuntu

ENTRYPOINT["sleep"]
```
this way when we use docker run we can just append a number that specifies how many seconds the container shoudl sleep for. Ex: 10s  `docker run sleeper 10`
The issue now is that if we do not append a number it will result in an error due to missing param for sleep. This is resolved by combining Entrypoint and CMD as follows:
```bash
FROM ubuntu

ENTRYPOINT["sleep"]

CMD["5"]
```
appending a number to the docker run command overwrites the CMD value
to overwrite the entrypoint value and CMD the command is as follows:
```bash
docker run --entrypoint sleep2.0 sleeper 10
```
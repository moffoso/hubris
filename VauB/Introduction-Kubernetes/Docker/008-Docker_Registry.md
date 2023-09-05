To access a private registry you have to login
```bash
sudo docker login [private registry address]
```

Create your own private registry
```bash
sudo docker run -d -p 5000:5000 --name registry registry:2
```

Push image to private registry
```bash
sudo docker image tag my-image localhost:5000/my-image
sudo docker push localhost:5000/my-image
```

Pull image from registry
```bash
# if on the same host as registry
sudo docker pull localhost:5000/my-image 
# name or ip of host if on another host
sudo docker pull 192.168.56.100:5000/my-image 
```
#### Docker Containers
- Share the same Kernel
- Individual Environments with their own Processes, Services, etc.
- Docker does not virtualise -> doesn't run different Kernels on the same Hardware
- Docker have less isolation as more resources are shared vs VM's as they rely on the same OS

#### Container vs Image
- Docker Image -> runs on every host the same

#### Setup and Installation
check for older version of docker and remove
```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

get repository and install
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

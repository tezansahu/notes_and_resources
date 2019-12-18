# Docker

## Install Docker:

```bash
$ sudo apt-get remove docker docker-engine docker.io containerd runc
$ sudo apt-get update
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo apt-key fingerprint 0EBFCD88
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
$ sudo docker run hello-world # Verify the installation

# Run docker as non-root user
$ sudo groupadd docker
$ sudo usermod -aG docker $USER

# If you get the following error: 
# docker: Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Post http://%2Fvar%2Frun%2Fdocker.sock/v1.35/containers/create: dial unix /var/run/docker.sock: connect: permission denied.See 'docker run --help'
# Use the following:
$ sudo chmod 666 /var/run/docker.sock
```

## Docker Usage:

```bash
# Pull an image from Dockerhub
$ docker pull <image-name>

# To view all locally available images
$ docker images 

# Start an image in a container (`--rm` flag is to remove the container automatically on exit)
$ docker run --rm --name <container-name> -p <host-port>:<container-port> <image-name> 
# Use `-P` to publish all exposed container ports to random docker ports

# To view all running containers
$ docker ps 

# To view all containers (running, stopped, etc.)
$ docker ps -a 

# Stop a container
$ docker stop <container-id/name>

# Remove a container
$ docker rm <container-id/name>

# Remove an image
$ docker rmi <image-name>

# Create a Dockerfile with necessary information

# To create a docker image
$ docker build -t <image-tag> <path-to-folder-containing-Dockerfile>

# To push an image to Dockerhub
$ docker login
$ docker push <image-name>:<optional-tag>
```


## Install Docker Compose:

```bash
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
$ docker-compose --version # Verify installation
```
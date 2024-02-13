# Install docker

# What is "Docker"

Docker is a set of platform as a service products that use OS-level virtualization to deliver software in packages called containers.[^1}]

A container is a standard unit of software that packages up code and all its dependencies so the application runs quickly and reliably from one computing environment to another. A Docker container image is a lightweight, standalone, executable package of software that includes everything needed to run an application: code, runtime, system tools, system libraries and settings. Container images become containers at runtime and in the case of Docker containers – images become containers when they run on Docker Engine. Available for both Linux and Windows-based applications, containerized software will always run the same, regardless of the infrastructure. Containers isolate software from its environment and ensure that it works uniformly despite differences for instance between development and staging.[^2]


# Version used

> There can be slight differences on installation processes if you are on different versions or hardware

| Application | Version |
|----|----|
| Ubuntu | 20.04 |
| Synology DSM | 7.1.1-42962 Update 1 |
| Docker Compose | 1.29.2 |
| Portainer | 2.1.61 |


# How to install Docker

## On Linux server

> I assume that you know the most basic of unix commands and server setup

### Prepare the server

1. Make sure that you have updated `apt` package index

   ```bash
   sudo apt-get update && sudo apt-get upgrade -y
   ```

2. Allow `apt` to use repo over https

   ```bash
   sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
   ```

3. Add docker official `GPG` key

   ```bash
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   ```

4. Setup the repo

   ```bash
   sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
   ```


### Install Docker CE

1. Install `Docker CE` and `containerd` runtime

   ```bash
   sudo apt-get install docker-ce docker-ce-cli containerd.io
   ```

### Create Docker user

1. Create a new user i.e. `vangeaux`

   ```bash
   adduser vangeaux
   ```

2. Add that user to docker group

   ```bash
   usermod -aG docker vangeaux
   ```

3. Restart docker service

   ```bash
   systemctl restart docker
   ```


### Test Docker

1. Test if docker is running fine

   ```bash
   sudo docker run --name hello-world-container hello-world
   ```

2. You should get this:

   ```none
   Hello from Docker!
   
   
   This message shows that your installation appears to be working correctly.
   ```

3. Remove the container if it is running fine

   ```bash
   sudo docker rm hello-world-container
   ```


### Configure autostart

1. Enable docker to run on system boot

   ```bash
   systemctl enable docker
   ```


## On Synology NAS

1. Log in to `DSM` and open the `Package Center`
2. Type `docker` in the search bar, hit `ENTER`, then click `Install`
3. Wait a couple of minutes and `Docker` will show in the `Installed` tab in `Package Center`. It should also show in the `Main Menu`.


[^1]: https://en.wikipedia.org/wiki/Docker_(software)
[^2]: https://www.docker.com/resources/what-container/
# What is Portainer

Portainer Community Edition is a lightweight service delivery platform for containerized applications that can be used to manage Docker, Swarm, Kubernetes and ACI environments. It is designed to be as simple to deploy as it is to use.[^1]


# Version used

> There can be slight differences on installation processes if you are on different versions or hardware

| Application | Version |
|----|----|
| Debian | 11 |
| Synology DSM | 7.1.1-42962 Update 1 |
| Linux Docker | 20.10.21 |
| Synology Docker | 20.10.3-1308 |
| Docker Compose | 1.29.2 |
| Portainer | 2.1.61 |


# Prerequisites
* [Docker](install-docker.md)

# Environments in the examples

> Check these if you are not sure which environments you can safely change

| Name | Value |
|----|----|
| **Config directory mount** | /volume1/docker/portainer |
| **Container name** | portainer |
| **Port 9443 map** | 10001 |
| **Port 9000 map** | 10000 |
| **Container image** | portainer/portainer-ce |


# How to install Portainer

> This tutorial assume that you know the most basic of unix commands and server setup

## On Linux server(easy)

1. Prepare config directory for `Portainer`

   ```bash
   sudo mkdir -p /volume1/docker/10000-portainer
   ```

2. Go to the directory

   ```bash
   cd /volume1/docker/10000-portainer
   ```

3. Copy my repo

   ```bash
   sudo git clone https://github.com/van-geaux/docker/compose
   ```



## On Linux server(harder)

### Prepare the directory and docker compose file


1. Prepare config directory for `Portainer`

   ```bash
   sudo mkdir -p /volume1/docker/10000-portainer
   ```


2. Create `docker-compose.yml` file

   ```bash
   sudo nano /volume1/docker/portainer/docker-compose.yml
   ```


3. Fill the file with these:

   
:::warning
   Change every `${}` to your values

   :::

   ```yaml
   version: "3.3"
   services:
     portainer-ce:
       ports:
         - ${SSH_PORT_FOR_AGENT}:8000
         - ${HTTP_PORT}:9000
         - ${HTTPS_PORT}:9443
       container_name: 10000-portainer # You can change that
       restart: always
       volumes:
         - /var/run/docker.sock:/var/run/docker.sock
         - ${PORTAINER_DATA_PATH}:/data
       image: portainer/portainer-ce
   ```


4. Save the file, after that go to where the file is

   ```bash
   cd /volume1/docker/10000-portainer
   ```


5. Install `Portainer` using `Docker Compose`

   ```bash
   sudo docker-compose up
   ```


6. If you have no reverse proxy setup for `Portainer`, you need to open up the port to access `Portainer`

   ```javascript
   sudo ufw allow 10000
   ```


## On Synology NAS

### Using Synology Docker Wizard


:::tip
In my opinion, installing a container using Synology Docker Wizard is **less recommended**. You may want to try installing your first container using CLI instead as it is more versatile, and it will teach you to configure your containers better. Check how to do it [below](#h-using-synology-cli)

:::


#### Prepare the directory


1. Create `/volume1/docker/10000-portainer` directory.


:::info
The `docker` part is a shared directory, go to the `Control Panel` to create it then create `10000-portainer` directory inside `docker` shared directory

:::


#### Download Portainer image


1. Open up `Docker` application, go to the `Registry` tab, then search for `portainer`
2. Double click on the image named `portainer/portainer-ce` make sure the tag is `latest` then hit `Select`
3. Go to the `Image` tab and wait a couple minutes for the image to download, when it is done the burger icon should be all blue


#### Deploy Portainer


1. Double click the `portainer/portainer-ce:latest`
2. Make sure the selected network under `Use the selected networks` is `bridge`
3. Fill the `Container Name` i.e. `10000-portainer`
4. Make sure `Enable auto-restart` is checked then click `Next`


:::info
I did not check `Enable web portal via Web Station` because I'm not using it. You can check it to open `Portainer` using `Synology Web Station`

:::


5. Put your bind port into each `Local Port`, i.e. `10001` on `Container Port` 8000 and `10000` on `Container Port` 9000, then click `Next`
6. Click `Add Folder` then pick the directory you created before i.e. `/docker/10000-portainer`
7. Fill `Mount Path` with `/data` then click `Next`
8. Make sure `Run this container after the wizard is finished` is checked, then click `Done`


### Using Synology CLI


:::tip
In my opinion, installing `Portainer` using `CLI` is the best way to install it on Synology

:::


1. Open `Control Panel` then open the `Task Scheduler`
2. Click `Create` then pick `Scheduled Task` then `User-defined script`
3. Pick a task name i.e. `Install Portainer`, choose `root` as `user`, then make sure that `Enabled` checkbox is unticked
4. Go to the `Schedule` tab, click on `Run on the following date` and pick `Do not repeat` in the lower dropdown list
5. Go to the `Task Settings` tab then paste the following  codes to the `User-defined script` box

   ```bash
   docker run -d \
     --name=10000-portainer \ #<<< you can change that
     -p 10001:8000 \ #<<< you can change that
     -p 10000:9000 \ #<<< you can change that
     -v /var/run/docker.sock:/var/run/docker.sock \
     -v /volume1/docker/10000-portainer:/data \ #<<< you can change that
     --restart=unless-stopped \
     portainer/portainer-ce
   ```


6. Save everything then right click the task and clik `Run`
7. Wait a couple minutes for the container to install. If you do not receive an error email then `10000-portainer` will show in Synology Docker's `Container` tab


## How to configure Portainer


1. Enter your VPS/Synology IP with `Portainer` port in your browser to open its web app i.e. http://192.168.1.1:10000
2. Under `New Portainer Installation`, fill up your credentials then click `Create user`
3. Click `Get Started` then just click `local`
4. `Portainer` is ready to use


## How to update Portainer

### On Linux Server


1. Stop the `Portainer`

   ```bash
   sudo docker stop 10000-portainer
   ```


2. Delete the old container

   ```bash
   sudo docker rm 10000-portainer
   ```


3. Go to where you put `docker-compose.yml`

   ```bash
   cd /docker/10000-portainer
   ```


4. Run `Docker Compose` again

   ```bash
   sudo docker-compose up
   ```


### On Synology NAS

#### If you used Synology Docker Wizard


1. Open `Docker` application
2. Go to the `Container` tab
3. `Right click` on `10000-Portainer`, pick `Action` then `Stop`
4. `Right click` on `10000-Portainer` again, pick `Action` then `Delete`
5. Do all the steps [here](#h-deploy-portainer) again

#### If you used Synology CLI


1. Open `Docker` application
2. Go to the `Container` tab
3. `Right click` on `10000-Portainer`, pick `Action` then `Stop`
4. `Right click` on `10000-Portainer` again, pick `Action` then `Delete`
5. Do all the steps [here](#h-using-synology-cli) again


[^1]: https://github.com/portainer/portainer
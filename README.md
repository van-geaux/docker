# Docker

Self-hosting is the practice of running and maintaining a website or service using a private web server, instead of using a service outside of your own control.

These are the how-to-install notes of self-hosted apps I personally using or have used.

> You can use these on any system you can install `Docker` on, be it on Unix-based or Windows machine

> The majority of the containers are installed using `YML` format inside `Portainer`
> You can also use 

> PIRACY DISCLAIMER
> 
> It's crucial to understand that while my notes *can* theoretically be used with pirated media, I do **not** endorse or support piracy.


# Prerequisites

1. [Install Docker](docs/prerequisites/docker.md)
2. [Install Portainer](docs/prerequisites/portainer.md)


# Self-hosted compose

**Linux TL;DR w/o Portainer**

1. Copy the compose and .env you want to your working directory, just change the `container_name` part
   ```bash
   wget https://raw.githubusercontent.com/van-geaux/docker/main/compose/container_name/docker-compose.yml
   ```
   ```bash
   wget https://raw.githubusercontent.com/van-geaux/docker/main/compose/container_name/.env
   ```
2. Edit the files as necessary
   ```bash
   sudo nano docker-compose.yml
   ```
   ```bash
   sudo nano .env
   ```
3. Deploy the container
   ```bash
   sudo docker compose up -d
   ```



1. [System](/docs/system/system.md)
2. [Administration](/docs/administration/administration.md)
3. [Productivity](/docs/productivity/productivity.md)
4. [Entertainment](/docs/entertainment/entertainment.md)
5. [Utility](/docs/utility/utility.md)
6. [Other](/docs/other/other.md)


# Useful Docker configuration


# Inter-app integration


# Application chain

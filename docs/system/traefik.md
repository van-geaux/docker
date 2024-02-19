# Traefik

# What is Traefik

Traefik is a leading modern reverse proxy and load balancer that makes deploying microservices easy. Traefik integrates with your existing infrastructure components and configures itself automatically and dynamically. [^1]

> Traefik is a bit harder to setup the first time but much easier to maintain going forward compared to other reverse proxy

# Version used in this documentation

> There can be slight differences on installation processes if you are on different versions or hardware


| Application | Version |
|----|----|
| Debian | 11 |
| Synology DSM | 7.2-64570 Update 1 |
| Linux Docker | 20.10.22 |
| Synology Docker | 20.10.23-1413 |
| Portainer | 2.19.1 |
| Traefik | 2.10.4 |


# Prerequisites

* [Docker](../prerequisites/docker.md)
* [Portainer](../prerequisites/portainer.md)

> This documentation used `Portainer` to install `Traefik` on `Linux` and `Synology`

# Environments in the examples

> Check these if you are not sure which environments you can safely change

| Environment name | Example value |
|----|----|
| **Container name** | traefik |
| **Container image** | traefik:v2.10 |
| **${TRAEFIK_DYNCONFIG_PATH}** | /volume1/docker/traefik/config |
| **${TRAEFIK_CONFIG_FILE}** | /volume1/docker/traefik/traefik.yml |
| **${TRAEFIK_CERTIFICATE_PATH}** | /volume1/docker/traefik/letsencrypt |
| **${TRAEFIK_DOMAIN}** | traefik.domain.tld |

# How to install Traefik

## Linux TL;DR w/o Portainer

> Please [click here](#prepare-the-directories) to skip Linux TL;DR and read the detailed guide instead

1. **In your traefik working directory**, copy the compose, .env, and config file
   ```bash
   wget https://raw.githubusercontent.com/van-geaux/docker/main/compose/traefik/docker-compose.yml && wget https://raw.githubusercontent.com/van-geaux/docker/main/compose/traefik/.env && wget https://raw.githubusercontent.com/van-geaux/docker/main/compose/traefik/traefik.yml
   ```
2. **In your traefik working directory**, create the `config` and `letsencrypt` directory
   ```bash
   sudo mkdir -p /{config,letsencrypt}
   ```

3. Edit the files as necessary
   ```bash
   sudo nano docker-compose.yml
   ```
   ```bash
   sudo nano .env
   ```
   ```bash
   sudo nano traefik.yml
   ```
4. Deploy the container
   ```bash
   sudo docker compose up -d
   ```

## Prepare the directories

1. If you are on Linux, use this:

   ```bash
   sudo mkdir -p /volume1/docker/traefik/{config,letsencrypt}
   ```
2. If you are on Synology then just create `traefik` directory inside `/volume1/docker` then create `config` and `letsencrypt` directories inside `/volume1/docker/traefik`

## Create the static configuration

1. If you are on Linux, use this:

   ```bash
   sudo nano /volume1/docker/traefik/traefik.yml
   ```
2. If you are on Synology then just create a file named `traefik.yml` inside `/volume1/docker/traefik`
3. Fill the file with these:

   ```yaml
   global:
     checkNewVersion: true
     sendAnonymousUsage: false
   
   entryPoints:
     web:
       address: ":80"
       http:
         redirections:
           entryPoint:
             to: websecure
             scheme: https
     websecure:
       address: ":443"
   
   api:
     insecure: false
     dashboard: true
   
   providers:
     file:
       directory: /config
       watch: true
     docker:
       endpoint: "unix:///var/run/docker.sock"
       exposedByDefault: false
   
   certificatesResolvers:
     myresolver:
       acme:
         email: yourmail@domain.tld # you need to change that to your email
         storage: /letsencrypt/acme.json
         httpChallenge:
           entryPoint: web
   ```

> This configuration forward every `http` requests to `https`

## Create the dynamic configuration

> Dynamic configuration can be used for services not in the same network with Traefik. It is not necessary and you can skip this part

1. If you are on Linux, use this:

   ```bash
   sudo touch /volume1/docker/traefik/config/config.yml
   ```
2. If you are on Synology then just create a file named `config.yml` inside `/volume1/docker/traefik/config.yml`
3. You can use dynamic configuration for services not in the same network with your traefik. There is a lot to read but here is an example to start:

> This is just an example of a case where you want to use traefik to connect to plex in a different server. You need another Traefik in that server too, I will expand the guide a bit sometime

```yaml
http:
  routers:
    plex:
     rule: Host(`plex.domain.tld`)
     service: plex
     tls:
       certResolver: myresolver

  services:
    plex:
      loadBalancer:
        servers:
        - url: "http://100.100.1.1:32400"
        passHostHeader: true
```

## Create a bridge network

1. Log in to `Portainer` web GUI on your browser then click on the environment you are using, in my case it is `local`
2. Go to the `Networks` tab on the left then click on `+ Add network`
3. Fill the name i.e `proxy.bridge`
4. Make sure the `Driver` is `bridge`
5. You can leave everything else as is and click `Create the network`

## Install Traefik

1. Log in to `Portainer` web GUI on your browser then click on the environment you are using, in my case it is `local`
2. Go to the `Stacks` tab on the left then click on `+ Add stack`
3. Copy the example configuration below to the `Web editor`, do not forget to fill the stack's name i.e `traefik`

   > Change every `${}` to your values, you can check the examples [here](#environments-in-the-examples)

   ```yaml
   version: '3'
   services:
     traefik:
       container_name: traefik
       image: traefik:v2.10
       ports:
         - "80:80"
         - "443:443"
         - "8080:8080"
       volumes:
         - /var/run/docker.sock:/var/run/docker.sock:ro
         - ${TRAEFIK_DYNCONFIG_PATH}:/config
         - ${TRAEFIK_CONFIG_FILE}:/traefik.yml
         - ${TRAEFIK_CERTIFICATE_PATH}:/letsencrypt
       labels:
         - traefik.enable=true
         - traefik.http.routers.traefik.rule=Host(`${TRAEFIK_DOMAIN}`)
         - traefik.http.services.traefik.loadbalancer.server.port=8080
         - traefik.http.routers.traefik.service=api@internal
         - traefik.http.routers.traefik.tls=true
         - traefik.http.routers.traefik.tls.certresolver=myresolver

   # edit them if you used different setting but the container you are planning to use Traefik with needs to be in the same network
   networks:
     default:
       name: proxy.bridge
       external: true
   ```

   > Traefik needs to be in the same network of whatever service you want to reverse proxy to. You can put multiple bridge networks in the compose to connect to your services

4. Click `Update the stack` then wait a bit until the container is deployed

   > You can open Traefik dashboard through `traefik.domain.tld`

# How to use Traefik

## Prepare your domain

Add an A name to your domain nameserver i.e. `npm.domain.tld`, I myself use wildcard domain `*.domain.tld` so I don't need to create one for each subdomain on `domain.tld`

## How to reverse proxy

1. To use Traefik to reverse proxy to your services, you just have to make sure that Traefik can access your web service and you have to put the right labels in your service’s compose:

   ```yaml
       labels:
         - traefik.enable=true
         - traefik.http.routers.yourservice.rule=Host(`yourservice.domain.tld`)
         - traefik.http.routers.yourservice.service=yourservice
         - traefik.http.services.yourservice.loadbalancer.server.port=internalport
         - traefik.http.routers.yourservice.tls=true
         - traefik.http.routers.yourservice.tls.certresolver=myresolver
   ```
2. In this example, I’m deploying Dozzle in the `proxy.bridge` network:

   ```yaml
   version: '3.3'
   services:
     dozzle:
       container_name: dozzle
       ports:
         - '8080:8080'
       volumes:
         - '/var/run/docker.sock:/var/run/docker.sock'
       restart: unless-stopped
       image: 'amir20/dozzle:latest'
       labels:
         - traefik.enable=true
         - traefik.http.routers.dozzle.rule=Host(`dozzle.domain.tld`)
         - traefik.http.routers.dozzle.service=dozzle
         - traefik.http.services.dozzle.loadbalancer.server.port=8080
         - traefik.http.routers.dozzle.tls=true
         - traefik.http.routers.dozzle.tls.certresolver=myresolver
   networks:
     default:
       name: proxy.bridge
       driver: bridge
   ```

   > You can change the service name in line 13 to 17, in the example they are `dozzle`
   
   > You need change the internal port in line 15 to match your service’s
  
   > Now you can access Dozzle through `dozzle.domain.tld`

   > You only need to repeat step 1 for your other web services

## How to use redirection

1. You can use regex for redirection

   ```yaml
       labels:
         - traefik.http.routers.sourcename.rule=Host(`sourcename.domain.tld`)
         - traefik.http.routers.sourcename.middlewares=sourcename
         - traefik.http.routers.sourcename.service=log
         - traefik.http.services.sourcename.loadbalancer.server.port=internalport
         - traefik.http.routers.sourcename.tls=true
         - traefik.http.routers.sourcename.tls.certresolver=myresolver
         - traefik.http.middlewares.redirectname.redirectregex.regex=^https?://sourcename.domain.tld/(.*)
         - traefik.http.middlewares.redirectname.redirectregex.replacement=https://targetname.domain.tld/$1
         - traefik.http.middlewares.redirectname.redirectregex.permanent=false
   ```
2. For example, if you want to redirect `log.domain.tld` to `dozzle.domain.tld`, you can do so by adding these labels:

   ```yaml
       labels:
         - traefik.http.routers.log.rule=Host(`log.domain.tld`)
         - traefik.http.routers.log.middlewares=log-redirect
         - traefik.http.routers.log.service=log
         - traefik.http.services.log.loadbalancer.server.port=8080
         - traefik.http.routers.log.tls=true
         - traefik.http.routers.log.tls.certresolver=myresolver
         - traefik.http.middlewares.dozzle.redirectregex.regex=^https?://log.domain.tld/(.*)
         - traefik.http.middlewares.dozzle.redirectregex.replacement=https://dozzle.domain.tld/$1
         - traefik.http.middlewares.dozzle.redirectregex.permanent=false
   ```
   
   > Don’t forget to change the service name and the domain

3. This is how it looks in the above Dozzle’s compose:

   ```yaml
   version: '3.3'
   services:
     dozzle:
       container_name: 20030-dozzle
       ports:
         - '20030:8080'
       volumes:
         - '/var/run/docker.sock:/var/run/docker.sock'
       restart: unless-stopped
       image: 'amir20/dozzle:latest'
       labels:
         - traefik.enable=true
         - traefik.http.routers.dozzle.rule=Host(`dozzle.domain.tld`)
         - traefik.http.routers.dozzle.service=dozzle
         - traefik.http.services.dozzle.loadbalancer.server.port=8080
         - traefik.http.routers.dozzle.tls=true
         - traefik.http.routers.dozzle.tls.certresolver=myresolver
         - traefik.http.routers.log.rule=Host(`log.domain.tld`)
         - traefik.http.routers.log.middlewares=log-redirect
         - traefik.http.routers.log.service=log
         - traefik.http.services.log.loadbalancer.server.port=8080
         - traefik.http.routers.log.tls=true
         - traefik.http.routers.log.tls.certresolver=myresolver
         - traefik.http.middlewares.log-redirect.redirectregex.regex=^https?://log.domain.tld/(.*)
         - traefik.http.middlewares.log-redirect.redirectregex.replacement=https://dozzle.domain.tld/$1
         - traefik.http.middlewares.log-redirect.redirectregex.permanent=false
   networks:
     default:
       name: system.bridge
       driver: bridge
   ```

   > Now if you try to access `log.domain.tld` you will be redirected to `dozzle.domain.tld`
   
   > You only need to repeat step 1 for your other services


[^1]: https://traefik.io/traefik/
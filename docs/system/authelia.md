# Authelia

# What is Authelia

Authelia is an open-source authentication and authorization server providing two-factor authentication and single sign-on (SSO) for your applications via a web portal. It acts as a companion for reverse proxies by allowing, denying, or redirecting requests. [^1]

# Version used for the tutorial

> There can be slight differences on installation processes if you are on different versions or hardware

| Application | Version |
|----|----|
| Ubuntu | 20.04 |
| Synology DSM | 7.1.1-42962 Update 1 |
| Linux Docker | 20.10.21 |
| Synology Docker | 20.10.3-1308 |
| Portainer | 2.16.2 |
| Authelia | 4.37.4 |
| Redis | 7.0.7 |

# Prerequisites

* [Docker](../prerequisites/docker.md)
* [Portainer](../prerequisites/portainer.md)
* [NGINX Proxy Manager](npm.md)
* [Traefik](traefik.md)

> This documentation used `Portainer` to install `Traefik` on `Linux` and `Synology`

> This documentation used `NGINX Proxy Manager` or `Traefik` to be used along `Authelia`

# Environments in the examples

> Check these if you are not sure which environments you can safely change.

| Environment name | Example value |
|----|----|
| **Authelia container name** | authelia |
| **Redis container name** | redis |
| **Authelia container image** | authelia/authelia |
| **Redis container image** | redis:alpine |
| **Authelia config directory mount** | /volume1/docker/authelia/config |
| **Redis data directory mount** | /volume1/docker/authelia/redis |
| **Authelia port** | 16000 |
| **User/group** | docker/docker |
| **uid/gid** | 1000/1000 |
| **Timezone** | Asia/Singapore |

# How to install Authelia

## Linux TL;DR w/o Portainer

> Please [click here](#prepare-the-directories) to skip Linux TL;DR and read the detailed guide instead

1. **In your authelia working directory**, copy the compose, .env, and config file
   ```bash
   wget https://raw.githubusercontent.com/van-geaux/docker/main/compose/authelia/docker-compose.yml && wget https://raw.githubusercontent.com/van-geaux/docker/main/compose/authelia/.env && wget https://raw.githubusercontent.com/van-geaux/docker/main/compose/authelia/configuration.yml && wget https://raw.githubusercontent.com/van-geaux/docker/main/compose/authelia/users_database.yml
   ```
2. **In your traefik working directory**, create the `config` and `redis` directory
   ```bash
   sudo mkdir -p /{config,redis}
   ```

3. Edit the files as necessary
   ```bash
   sudo nano docker-compose.yml
   ```
   ```bash
   sudo nano .env
   ```
   ```bash
   sudo nano configuration.yml
   ```
   ```bash
   sudo nano users_database.yml
   ```

4. Move the `configuration.yml` and `users_database.yml` to the `config` directory
   ```bash
   sudo mv configuration.yml /config/configuration yml && sudo mv users_database.yml /config/users_database.yml
   ```

5. Deploy the container
   ```bash
   sudo docker compose up -d
   ```

## Prepare the directories

1. If you are on Linux, use this:

   ```bash
   sudo mkdir -p /volume1/docker/authelia/{config,redis}
   ```

2. If you are on Synology then just create `authelia` directory inside `/volume1/docker` then create `config` and `redis` directory inside `/volume1/docker/authelia`

## Get your timezone

1. Go to [wikipedia's tz database](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)
2. Look up your timezone under `TZ database name` in the table

## Prepare configuration file

1. Create a `configuration.yml` inside `/volume1/docker/authelia/config`

   ```bash
   sudo nano /volume1/docker/authelia/config/configuration.yml
   ```

2. If you are on Synology then just copy any file to `/volume1/docker/authelia/config` then rename it to `configuration.yml`
3. Copy the example configuration below to the file (open the file first with text editor if you are on Synology)

> Change every `${}` to your values

   ```yaml
   ---
   ###############################################################
   #                   Authelia configuration                    #
   ###############################################################
   
   jwt_secret: 17X91RxaXM4vWkqYr5tItxtss #<<< you need to change that
   default_redirection_url: https://auth.domain.tld #<<< you need to change that
   
   server:
     host: 0.0.0.0
     port: ${AUTHELIA_PORT}
   
   log:
     level: debug
   
   totp:
     issuer: ${TOP_DOMAIN}
   
   # duo_api:
   #  hostname: api-123456789.domain.tld
   #  integration_key: ABCDEF
   #  # This secret can also be set using the env variables AUTHELIA_DUO_API_SECRET_KEY_FILE
   #  secret_key: 1234567890abcdefghifjkl
   
   authentication_backend:
     file:
       path: /config/users_database.yml #<<< you can change that
   
   access_control:
     default_policy: deny
     rules:
       # Rules applied to everyone
       - domain: auth.domain.tld #<<< you need to change that
         policy: bypass
       - domain:
         - app1.domain.tld #<<< you can change that
         - app2.domain.tld #<<< you can change that
         policy: one_factor
       - domain: 
         - app3.domain.tld #<<< you can change that
         - app4.domain.tld #<<< you can change that
         policy: two_factor
   
   session:
     name: authelia_session
     secret: unsecure_session_secret
     expiration: 3600  #<<< you can change that
     inactivity: 3600  #<<< you can change that
     domain: domain.tld  #<<< you can change that
   
     redis:
       host: redis #<<< you can change that
       port: 6379
   
   regulation:
     max_retries: 3 #<<< you can change that
     find_time: 120 #<<< you can change that
     ban_time: 300 #<<< you can change that
   
   storage:
     encryption_key: ZMsm2yBGuRLPr311rvg9E5zXV #<<< you need to change that
     local:
       path: /config/db.sqlite3
   
   notifier:
     filesystem:
       filename: /config/notification.txt
     #smtp:
     #  username: test
       # This secret can also be set using the env variables AUTHELIA_NOTIFIER_SMTP_PASSWORD_FILE
     #  password: password
     #  host: mail.domain.tld
     #  port: 25
     #  sender: admin@domain.tld
   ...
   ```

> You can also use wildcard domain \*.domain.tld in access control

4. Save the file
5. Here's a list of what you **HAVE** to change:

| Line number | Environment | Example Value | Notes |
|----|----|----|----|
| 6 | jwt_secret | 17X91RxaXM4vWkqYr5tItxtss | At least random 20 digits alphanumeric will be fine, you can also use secret |
| 7 | default_redirection_url | https://auth.domain.tld | Change it to your Authelia authentication sub-domain of choice |
| 17 | issuer | domain.tld | Change it to your top-level-domain |
| 33 | domain | auth.domain.tld | Make sure it's the same with line 7 |
| 49 | domain | domain.tld | Change it to your top-level-domain |
| 61 | encryption_key | ZMsm2yBGuRLPr311rvg9E5zXV | Recommended at least random 64 digits alphanumeric, you can also use secret |

6. Here's a list of what you **CAN** change:

| Line number | Environment | Example Value | Notes |
|----|----|----|----|
| 11 | port | 16000 | If you change it, make sure it will be the same port you expose on the container later |
| 27 | path | /config/users_database.yml | If you change it, make sure it's where you put the user database later |
| 36-37 | domain | app1.domain.tld | Put the sub-domain you want protected with Authelia SSO, you can also add more domains |
| 40-41 | domain | app3.domain.tld | Put the sub-domain you want protected with Authelia SSO and 2FA, you can also add more domains |
| 47 | expiration | 3600 | In seconds, session default expiration time |
| 48 | inactivity | 300 | In seconds, session expiration time on user inactivity |
| 52 | host | redis | If you change it, make sure it will be the same redis container name later |
| 56 | max_retries | 3 | Maximum failed retries before ban |
| 57 | find_time | 120 | In seconds, period of time for max_retries |
| 58 | ban_time | 300 | In seconds, ban duration |
| 66-74 | SMTP notification |    | If you want to use email notification for authentication and 2FA, configure this to your SMTP server and comment out line 66-67 |

## Prepare user database file

1. SSH to your machine (yes you too, Synology user) then hash your chosen password i.e `supersecretpassword` with this command:

   ```bash
   sudo docker run authelia/authelia:latest authelia hash-password supersecretpassword
   ```

2. It should output a hashed password starting with `$argon2...`, take a note of that value because you need it for later
3. Create a `configuration.yml` inside `/volume1/docker/authelia/config`

   ```bash
   sudo nano /volume1/docker/authelia/config/users_database.yml
   ```

4. If you are on Synology then just copy any file to `/volume1/docker/authelia/config` then rename it to `users_database.yml`
5. Copy the example configuration below to the file (open the file first with text editor if you are on Synology)

   ```yaml
   ###############################################################
   #                         Users Database                      #
   ###############################################################
    
   # This file can be used if you do not have an LDAP set up.
    
   # List of users
   users:
     vangeaux:
       displayname: "admin"
       password: "$argon2..."   # put the hashed password there, make sure it is inside quotation marks
       email: admin@domain.tld #<<< you need to change that
       groups:
         - admins
         - dev
   ```

6. Save the file

## Install Authelia

1. Log in to `Portainer` web GUI on your browser then click on the environment you are using, in my case it is `local`
2. Go to the `Stacks` tab on the left then click on `+ Add stack`
3. Copy the example configuration below to the `Web editor`, do not forget to fill the stack's name i.e `authelia`

> Change every `${}` to your values

   ```yaml
   version: '3.3'
   services:
     authelia:
       image: authelia/authelia
       container_name: authelia # You can change that
       volumes:
         - ${CONFIG_PATH}:/config
       ports:
         - ${AUTHELIA_PORT}:${AUTHELIA_PORT} # Make sure it's the same with the one in the configuration.yml
       restart: unless-stopped
       healthcheck:
         disable: true
       environment:
         - TZ=Asia/Singapore # You can change that
       depends_on:
         - redis
     redis:
       image: redis:alpine
       container_name: redis # You can change that
       volumes:
         - ${YOUR_REDIS_DATA_PATH}:/data
       expose:
         - 6379
       restart: unless-stopped
       environment:
         - TZ=Asia/Singapore # You can change that
   ```

4. After you are done, click `Deploy the stack`
5. Wait a couple minutes for `Authelia` to deploy

# Reverse proxy integration

## With NGINX Proxy Manager

1. If you are on Linux, open up port `16000`

   ```bash
   sudo ufw allow 16000
   ```

2. Open up the GUI on your Linux/Synology IP with `16000` port i.e `http://192.168.1.1:16000`
3. Create a proxy host to `Authelia` i.e `auth.authelia.com` forwarded to port `16000`
4. Make sure it also configured with SSL, then inside `auth.authelia.com` NPM configuration, go to `Advanced` tab and fill these inside `Custom NGINX Configuration`:

   ```yaml
       location / {
           set $upstream_authelia http://localhost:16000;
           proxy_pass $upstream_authelia;
           client_body_buffer_size 128k;
    
           # Timeout if the real server is dead
           proxy_next_upstream error timeout invalid_header http_500 http_502 http_503;
    
           # Advanced Proxy Config
           send_timeout 5m;
           proxy_read_timeout 360;
           proxy_send_timeout 360;
           proxy_connect_timeout 360;
    
           # Basic Proxy Config
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
           proxy_set_header X-Forwarded-Host $http_host;
           proxy_set_header X-Forwarded-Uri $request_uri;
           proxy_set_header X-Forwarded-Ssl on;
           proxy_redirect  http://  $scheme://;
           proxy_http_version 1.1;
           proxy_set_header Connection "";
           proxy_cache_bypass $cookie_session;
           proxy_no_cache $cookie_session;
           proxy_buffers 64 256k;
    
           # If behind reverse proxy, forwards the correct IP
           set_real_ip_from 10.0.0.0/8;
           set_real_ip_from 172.0.0.0/8;
           set_real_ip_from 192.168.0.0/16;
           set_real_ip_from fc00::/7;
           real_ip_header X-Forwarded-For;
           real_ip_recursive on;
       }
   ```

5. Make sure the port in line 2 is the port you configured when you install `Authelia`
6. IN NGINX Proxy Manager (NPM), create a proxy host for your application like you usually do. In this example I'm using `app3.example.<area>com` forwarded to port `20000`
7. Make sure it also configured with SSL then inside its configuration go to `Advanced` tab and fill these inside `Custom NGINX Configuration`:

   ```yaml
   location /authelia {
       internal;
       set $upstream_authelia http://localhost:16000/api/verify;
       proxy_pass_request_body off;
       proxy_pass $upstream_authelia;    
       proxy_set_header Content-Length "";
    
       # Timeout if the real server is dead
       proxy_next_upstream error timeout invalid_header http_500 http_502 http_503;
       client_body_buffer_size 128k;
       proxy_set_header Host $host;
       proxy_set_header X-Original-URL $scheme://$http_host$request_uri;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $remote_addr; 
       proxy_set_header X-Forwarded-Proto $scheme;
       proxy_set_header X-Forwarded-Host $http_host;
       proxy_set_header X-Forwarded-Uri $request_uri;
       proxy_set_header X-Forwarded-Ssl on;
       proxy_redirect  http://  $scheme://;
       proxy_http_version 1.1;
       proxy_set_header Connection "";
       proxy_cache_bypass $cookie_session;
       proxy_no_cache $cookie_session;
       proxy_buffers 4 32k;
    
       send_timeout 5m;
       proxy_read_timeout 240;
       proxy_send_timeout 240;
       proxy_connect_timeout 240;
   }
    
       location / {
           set $upstream_example http://localhost:20000; 
           proxy_pass $upstream_example; 
    
           auth_request /authelia;
           auth_request_set $target_url $scheme://$http_host$request_uri;
           auth_request_set $user $upstream_http_remote_user;
           auth_request_set $groups $upstream_http_remote_groups;
           proxy_set_header Remote-User $user;
           proxy_set_header Remote-Groups $groups;
           error_page 401 =302 https://auth.domain.tld/?rd=$target_url;
    
           client_body_buffer_size 128k;
    
           proxy_next_upstream error timeout invalid_header http_500 http_502 http_503;
    
           send_timeout 5m;
           proxy_read_timeout 360;
           proxy_send_timeout 360;
           proxy_connect_timeout 360;
    
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
           proxy_set_header X-Forwarded-Host $http_host;
           proxy_set_header X-Forwarded-Uri $request_uri;
           proxy_set_header X-Forwarded-Ssl on;
           proxy_redirect  http://  $scheme://;
           proxy_http_version 1.1;
           proxy_set_header Connection "";
           proxy_cache_bypass $cookie_session;
           proxy_no_cache $cookie_session;
           proxy_buffers 64 256k;
    
           set_real_ip_from 192.168.1.0/16;
           real_ip_header X-Forwarded-For;
           real_ip_recursive on;
    
       }
   ```

 8. Make sure the port in line 8 is the port you configured when you install `Authelia`
 9. Make sure the url in line 42 point to your `Authelia` sub-domain i.e `auth.domain.tld`
10. Make sure the url in line 33 point to your app i.e. `localhost:20000` or using IP i.e. `192.168.1.1:20000`
11. Make sure you also change `$upstream_example` in line 33 and 34 to your app name i.e. $upstream_sonarr
12. Save the configuration
13. Now if you go to `app3.domain.tld` you should be redirected to `Authelia` login page
14. Login using your credentials that you configured when you created the `users_database.yml`
15. Since we configured `app3.domain.tld` to use 2FA in the `configuration.yml`, we now need to configure said 2FA, pick code authentication
16. Now SHH to your machine again (you only need to do this once, and yes you too Synology users), then open up the `notification.txt`

    ```bash
    sudo cat /volume1/docker/authelia/config/notification.txt
    ```

17. Go to the link provided there and there should be a QR code on your screen
18. Scan the QR code with an 2FA app of your choice i.e `Google Authenticator` and now you can use the code inside the app to login to `Authelia`

> Congratulations now your webapp are protected by `Authelia`

> You only need to repeat step 6 to 12 for your other webapps

## With Traefik

1. Make sure Traefik can access Authelia, then add these labels to Authelia compose:

   ```yaml
       labels:
         - traefik.enable=true
         - traefik.http.routers.authelia.rule=Host(`authelia.domain.tld`)
         - traefik.http.services.authelia.loadbalancer.server.port=16000
         - traefik.http.routers.authelia.entryPoints=websecure
         - traefik.http.routers.authelia.tls=true
         - traefik.http.routers.authelia.tls.certresolver=myresolver
         - traefik.http.middlewares.authelia.forwardAuth.address=http://authelia:16000/api/verify?rd=https%3A%2F%2Fauthelia.domain.tld%2F
         - traefik.http.middlewares.authelia.forwardAuth.trustForwardHeader=true
         - traefik.http.middlewares.authelia.forwardAuth.authResponseHeaders=Remote-User,Remote-Groups,Remote-Name,Remote-Email
         - traefik.http.middlewares.authelia-basic.forwardAuth.address=http://authelia:16000/api/verify?auth=basic
         - traefik.http.middlewares.authelia-basic.forwardAuth.trustForwardHeader=true
         - traefik.http.middlewares.authelia-basic.forwardAuth.authResponseHeaders=Remote-User,Remote-Groups,Remote-Name,Remote-Email
   ```

> Don't forget to change the domain in line 3 and 8

2. Put this label in the service's compose that you want to put behind Authelia:

   ```yaml
       labels:
         - traefik.http.routers.servicename.middlewares=authelia@docker
   ```

> Don't forget to change the `servicename`

3. For example, this is how it looks in the above Dozzle's compose:

   ```yaml
   version: '3.3'
   services:
     dozzle:
       container_name: dozzle
       ports:
         - 8080:8080
       volumes:
         - /var/run/docker.sock:/var/run/docker.sock
       restart: unless-stopped
       image: amir20/dozzle:latest
       labels:
         - traefik.enable=true
         - traefik.http.routers.dozzle.rule=Host(`dozzle.domain.tld`)
         - traefik.http.routers.dozzle.service=dozzle
         - traefik.http.services.dozzle.loadbalancer.server.port=8080
         - traefik.http.routers.dozzle.tls=true
         - traefik.http.routers.dozzle.tls.certresolver=myresolver
         - traefik.http.routers.dozzle.middlewares=authelia@docker
   networks:
     default:
       name: system.bridge
       driver: bridge
   ```

> Congratulations now your webapp are protected by `Authelia`

> You only need to repeat step 2 for your other webapps


[^1]: https://www.authelia.com/
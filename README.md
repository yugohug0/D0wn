# d0wn
**d0wn - Docker OWN Server**

>A personnal docker managed server behind a traefik rverse proxy w/SSL | Let's encrypt

SSL enabled & disabled version

This project allow you to deploy several services on a server with docker containers, you just have to pick the one you want and stick them together !

>Thanks to SmartHomeBeginner for his well detailed tutorial ! https://www.smarthomebeginner.com/docker-home-media-server-2018-basic

## How does it works ?

The goal here is to make home server creation easy, secure & accessible for all.

EASY -> By using docker each service is constitued of one or two container maximum, the deployement of a container is so easy it can be done with one command-line
SECURE -> With traefik reverse-proxy and docker, we can use SSL to encrypt every communication with our server from the internet !
ACCESSIBLE -> The goal of this project is to automate this long process of creation with a script, one line to rule them all !

There is a lot more of benefits here but I want to make the long story short.

```
######################################### My configuration #########################################

OS : Ubuntu 18.04.3 LTS
CPU : Intel i5 XXX w/ Stock cooler
Motherboard : Mortar Arctic XXX
RAM : 8Gb Corsair Value Select
Powerbrick : XXX
Case : XXX
Disks :
- 120G Kingston SSD (OS)
- 2x4Tb Red Barracuda Drives (Medias)
- 2x1Tb Green Barracuda Drives (Important Stuff/Configs/RAID 1 MIRROR)

####################################################################################################
```

/!\ Important /!\

Prerequisites :
If you plan to make a SSL enabled server, you need to aleready have a domain name or get one and preferably move your DNS to cloudflare because it's more convenient (API Support, ...)
you also need to redirect ports 80, 443 & 8000 from the router to your server. And of course, create a DNS record for your domain name to your router public IP

## 1- Create a bootable USB Stick for your server with your favorite OS
```
NOTE : I'M USING UBUNTU SERVER
```
Find your best 8G USB Stick

Download the latest ISO image of Ubuntu on their official website, you have to choose between two versions.

* Ubuntu Desktop : Ubuntu w/ Graphical interface - Use this one if you plan to use a screen on your server

* Ubuntu Server : Command-line interface ONLY (You can download a graphical environnement afterwards)

Official download page : <https://ubuntu.com/#download>

Now you have the image, you need a tool to make your USB stick bootable

I personnaly love Balena Etcher : <https://www.balena.io/etcher/> (It's open source :)

Open your flashing program, select your USB, select your OS image and flash it !

Now your USB is ready we can install the OS on the server


## 2- Install Ubuntu & making directory tree

Now you can connect your USB to your fresh server, power it up and spam your boot option button on the keyboard, usually F11, F12, ...

Now you can see the Ubuntu installer menu select the normal install and follow the instructions, it's pretty straightforward ;)

After the installation of Ubuntu you need to create the global organisation of your folders and services, you can create your folder tree following this plan :
```
~/Docker_______
    |         |
Services    Storage_________ Data______________________________________
    |           |                           |                 |        \
SERVICENAME  Medias_____________        Nextcloud           Other      ...
    |           |       |       |
CONFIG.FILES  Movies   Shows  Music
```

```
 ############################################################################################
/!\       You can skip step 3 & 4 by executing the **Auto-folder-build.bash** script that I made, it will build the basic repository structure and ask for your environnement variables       /!\    
 ###########################################################################################
```


## 3- Update your packages and Install Docker

Let's update the list of the available packets
```
  sudo apt update
  sudo apt upgrade
```
Snap should be installed by default

Now let's install docker
```
  sudo snap install docker
  sudo apt-get install docker-compose
```
If snap is not installed
```
  sudo apt update
  sudo apt install snapd
```
Good you now have a fresh up-to-date Ubuntu server with a docker daemon running !

Try docker to check if he was installed correctly by running the hello-world image
  ``docker run hello-world``

## 4- Define your environnement

What's gonna make you server YOUR server is your environnement : YOUR own IP, YOUR passwords, ...

All of these informations are stored on your server in your environnement file, that's what gonna make the deployement easy because you only need to enter these informations once after that, a generic name is given to these environnement variables

EXAMPLE :
If I put the following line in my environnement file "mypassword123=PASSWORD", my password is called everytime I type $PASSWORD

Now we need to edit your environnement file !

Edit the /etc/environnement file and complete it with your informations (You don't need to do that if you used my script)
```
PUID=                    | The result of the "id" command
PGID=                    | The result of the "id" command
TZ=                      | Timezone, you can check your in the IANA TZ Database or following this link https://en.wikipedia.org/wiki/Tz_database
WORKDIR=                 | The directory where your config files should be (~/Docker/Services/ In this case)
MYSQL_ROOT_PASSWORD=     | SQL root password
CLOUDFLARE_EMAIL=        | your cloudlare email
CLOUDFLARE_API_KEY=      | your cloudflare API Key
DOMAINNAME=              | your domain name (If you have one)
HTTP_USERNAME=           | your generic usernanme
HTTP_PASSWORD=           | your generic password
```

## 5- Select the services you want to run !

The docker-compose file with every services included can be downloaded on the GitHub project repository

List of the official supported services :
```
######### FRONTENDS ##########
   #Portainer - A WebUI for Containers
   #Organizer - Unified HTPC/Home Server Web Interface
   #Nextcloud - Your own cloud storage
   #Phpmyadmin - A WebUI for your MariaDB database
   #Plex - WebUI to stream media
   #Tautulli (aka PlexPy) – Monitoring Plex Usage

######### BACKENDS ##########
   #Traefik - A reverse-proxy/load balancer for your services
   #Watchtower - Automatic Updater for your Containers/Apps
   #MariaDB – Database Server for your Apps
```
## 6- Global docker-compose rules

This file may scare you a first but it fact it's really easy ! It just automate de deployement of containers and explain the way docker have deploy them.

So, to select wich services you want, you just have to delete the one you don't want between two "#----------------------" statement.

READ THE COMMENTS IN THE DOCKER-COMPOSE !

docker-compose.yml

```yaml
#Reference: https://www.smarthomebeginner.com/docker-home-media-server-2018-basic
#Requirement: Set environmental variables: WORKDIR, PUID, PGID, MYSQL_ROOT_PASSWORD, and TZ as explained in the reference.
#~/Docker/Services = ${WORKDIR}
#     |
#/services_directories


version: "3.3"
services:

########## ROUTING /!\ Mandatory /!\ ###########
# Version 1.7 is important, traefik keeps restarting on newest versions
  traefik:
      hostname: traefik
      image: traefik:v1.7.16
      container_name: traefik
      restart: always
      domainname: ${DOMAINNAME}
      networks:
        - default
        - reverse-proxy
      ports:
        - "80:80"
        - "443:443"
        - "8000:8080"
      environment:
        - CF_API_EMAIL=${CLOUDFLARE_EMAIL}
        - CF_API_KEY=${CLOUDFLARE_API_KEY}
      labels:
        - "traefik.enable=true"
        - "traefik.backend=traefik"
        - "traefik.frontend.rule=Host:traefik.${DOMAINNAME}"
        - "traefik.port=8080"
        - "traefik.docker.network=traefik_proxy"
        - "traefik.frontend.headers.SSLRedirect=true"
        - "traefik.frontend.headers.STSSeconds=315360000"
        - "traefik.frontend.headers.browserXSSFilter=true"
        - "traefik.frontend.headers.contentTypeNosniff=true"
        - "traefik.frontend.headers.forceSTSHeader=true"
        - "traefik.frontend.headers.SSLHost=example.com"
        - "traefik.frontend.headers.STSIncludeSubdomains=true"
        - "traefik.frontend.headers.STSPreload=true"
        - "traefik.frontend.headers.frameDeny=false"
        - "traefik.frontend.auth.basic.users=${HTTP_USERNAME}:${HTTP_PASSWORD}"
      volumes:
        - /var/run/docker.sock:/var/run/docker.sock:ro
        - ${WORKDIR}/traefik:/etc/traefik
#-------------------------------------------------------------------------------
######### FRONTENDS ##########
#-------------------------------------------------------------------------------
#Portainer - A WebUI for Containers
  portainer:
    image: portainer/portainer:latest
    hostname: portainer
    container_name: portainer
    restart: unless-stopped
    command: -H unix:///var/run/docker.sock
    ports:
      - "8001:9000"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ${WORKDIR}/Portainer/:/data
    environment:
      - TZ=${TZ}
    labels:
      - "traefik.enable=true"
      - "traefik.backend=portainer"
      - "traefik.frontend.rule=Host:portainer.${DOMAINNAME}"
      - "traefik.port=9000"
      - "traefik.docker.network=reverse-proxy"
      - "traefik.frontend.headers.SSLRedirect=true"
      - "traefik.frontend.headers.STSSeconds=315360000"
      - "traefik.frontend.headers.browserXSSFilter=true"
      - "traefik.frontend.headers.contentTypeNosniff=true"
      - "traefik.frontend.headers.forceSTSHeader=true"
      - "traefik.frontend.headers.SSLHost=example.com"
      - "traefik.frontend.headers.STSIncludeSubdomains=true"
      - "traefik.frontend.headers.STSPreload=true"
      - "traefik.frontend.headers.frameDeny=false"
      - "com.centurylinklabs.watchtower.enable=true"
#-------------------------------------------------------------------------------
#Organizer - Unified HTPC/Home Server Web Interface
  organizr:
    container_name: organizr
    hostname: organizr
    restart: unless-stopped
    image: lsiocommunity/organizr:latest
    volumes:
      - ${WORKDIR}/Organizr:/config
    ports:
      - "8002:80"
    environment:
      - PUID=${PUID}
      - PGID=${PGID}
      - TZ=${TZ}
    labels:
      - "traefik.enable=true"
      - "traefik.backend=organizr"
      - "traefik.frontend.rule=Host:organizr.${DOMAINNAME}"
      - "traefik.port=80"
      - "traefik.docker.network=reverse-proxy"
      - "traefik.frontend.headers.SSLRedirect=true"
      - "traefik.frontend.headers.STSSeconds=315360000"
      - "traefik.frontend.headers.browserXSSFilter=true"
      - "traefik.frontend.headers.contentTypeNosniff=true"
      - "traefik.frontend.headers.forceSTSHeader=true"
      - "traefik.frontend.headers.SSLHost=example.com"
      - "traefik.frontend.headers.STSIncludeSubdomains=true"
      - "traefik.frontend.headers.STSPreload=true"
      - "traefik.frontend.headers.frameDeny=false"
      - "com.centurylinklabs.watchtower.enable=true"
#-------------------------------------------------------------------------------
#Nextcloud - Your own cloud storage (Needs Mariadb backend)
  nextcloud:
    hostname: nextcloud
    container_name: nextcloud
    image: nextcloud
    restart: unless-stopped
    ports:
      - "8003:80"
    links:
      - mariadb:db
    volumes:
      - ${WORKDIR}/Nextcloud:/var/www/html
    labels:
      - "traefik.enable=true"
      - "traefik.backend=nextcloud"
      - "traefik.frontend.rule=Host:nextcloud.${DOMAINNAME}"
      - "traefik.port=80"
      - "traefik.docker.network=reverse-proxy"
      - "traefik.frontend.headers.SSLRedirect=true"
      - "traefik.frontend.headers.STSSeconds=315360000"
      - "traefik.frontend.headers.browserXSSFilter=true"
      - "traefik.frontend.headers.contentTypeNosniff=true"
      - "traefik.frontend.headers.forceSTSHeader=true"
      - "traefik.frontend.headers.SSLHost=example.com"
      - "traefik.frontend.headers.STSIncludeSubdomains=true"
      - "traefik.frontend.headers.STSPreload=true"
      - "traefik.frontend.headers.frameDeny=false"
      - "com.centurylinklabs.watchtower.enable=true"
#-------------------------------------------------------------------------------
#Phpmyadmin - A WebUI for your MariaDB database (Optionnal)
  phpmyadmin:
    hostname: phpmyadmin
    container_name: phpmyadmin
    image: phpmyadmin/phpmyadmin:latest
    restart: unless-stopped
    links:
      - mariadb:db
    ports:
      - "8004:80"
    environment:
      - PMA_HOST=mariadb
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
    labels:
      - "traefik.enable=true"
      - "traefik.backend=pma"
      - "traefik.frontend.rule=Host:pma.${DOMAINNAME}"
      - "traefik.port=80"
      - "traefik.docker.network=traefik_proxy"
      - "traefik.frontend.headers.SSLRedirect=true"
      - "traefik.frontend.headers.STSSeconds=315360000"
      - "traefik.frontend.headers.browserXSSFilter=true"
      - "traefik.frontend.headers.contentTypeNosniff=true"
      - "traefik.frontend.headers.forceSTSHeader=true"
      - "traefik.frontend.headers.SSLHost=example.com"
      - "traefik.frontend.headers.STSIncludeSubdomains=true"
      - "traefik.frontend.headers.STSPreload=true"
      - "traefik.frontend.headers.frameDeny=false"
      - "com.centurylinklabs.watchtower.enable=true"
#-------------------------------------------------------------------------------
#PlexMediaServer - A WebUI for streaming all of your media locally or remotly
      plexms:
          container_name: plexms
          restart: unless-stopped
          image: plexinc/pms-docker
          volumes:
            - ${WORKDIR}/plexms:/config
            - ${WORKDIR}/Downloads/plex_tmp:/transcode
            - ~/Docker/Storage/Medias/:/media
          ports:
            - "32400:32400/tcp"
            - "3005:3005/tcp"
            - "8324:8324/tcp"
            - "32469:32469/tcp"
            - "1900:1900/udp"
            - "32410:32410/udp"
            - "32412:32412/udp"
            - "32413:32413/udp"
            - "32414:32414/udp"
          environment:
            - TZ=${TZ}
            - HOSTNAME="Docker Plex"
            - PLEX_CLAIM="claim-YYYYYYYYY"
            - PLEX_UID=${PUID}
            - PLEX_GID=${PGID}
            - ADVERTISE_IP="http://SERVER-IP:32400/"
          networks:
            - traefik_proxy
          labels:
            - "traefik.enable=true"
            - "traefik.backend=plexms"
            - "traefik.frontend.rule=Host:plex.${DOMAINNAME}"
            - "traefik.port=32400"
            - "traefik.protocol=http"
            - "traefik.docker.network=traefik_proxy"
            - "traefik.frontend.headers.SSLRedirect=true"
            - "traefik.frontend.headers.STSSeconds=315360000"
            - "traefik.frontend.headers.browserXSSFilter=true"
            - "traefik.frontend.headers.contentTypeNosniff=true"
            - "traefik.frontend.headers.forceSTSHeader=true"
            - "traefik.frontend.headers.SSLHost=example.com"
            - "traefik.frontend.headers.STSIncludeSubdomains=true"
            - "traefik.frontend.headers.STSPreload=true"
            - "traefik.frontend.headers.frameDeny=true"
#-------------------------------------------------------------------------------
#Tautulli (aka PlexPy) – Monitoring Plex Usage
  tautulli:
    container_name: tautulli
    hostname: tautulli
    restart: unless-stopped
    image: tautulli/tautulli
    volumes:
      - ${WORKDIR}/Tautulli:/config
      - ${WORKDIR}/Plex/config/Library/Application Support/Plex Media Server/Logs:/logs:ro
    ports:
      - "8005:8181"
    environment:
      - PUID=${PUID}
      - PGID=${PGID}
      - TZ=${TZ}
    labels:
      - "traefik.enable=true"
      - "traefik.backend=tautulli"
      - "traefik.frontend.rule=Host:tautulli.${DOMAINNAME}"
      - "traefik.port=8181"
      - "traefik.docker.network=reverse-proxy"
      - "traefik.frontend.headers.SSLRedirect=true"
      - "traefik.frontend.headers.STSSeconds=315360000"
      - "traefik.frontend.headers.browserXSSFilter=true"
      - "traefik.frontend.headers.contentTypeNosniff=true"
      - "traefik.frontend.headers.forceSTSHeader=true"
      - "traefik.frontend.headers.SSLHost=example.com"
      - "traefik.frontend.headers.STSIncludeSubdomains=true"
      - "traefik.frontend.headers.STSPreload=true"
      - "traefik.frontend.headers.frameDeny=false"
      - "com.centurylinklabs.watchtower.enable=true"
#-------------------------------------------------------------------------------
######### BACKENDS ##########
#-------------------------------------------------------------------------------
# MariaDB – Database Server for your Apps (Used by Nextcloud)
  mariadb:
    image: linuxserver/mariadb:latest
    container_name: mariadb
    hostname: mariadb
    volumes:
        - ${WORKDIR}/Mariadb:/config
    ports:
      - target: 3306
        published: 3306
        protocol: tcp
        mode: host
    restart: unless-stopped
    environment:
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - PUID=${PUID}
      - PGID=${PGID}
      - TZ=${TZ}
    labels:
      - "com.centurylinklabs.watchtower.enable=true"
#-------------------------------------------------------------------------------
# Watchtower - Automatic Update of Containers/Apps
  watchtower:
    container_name: watchtower
    hostname: watchtower
    restart: always
    image: v2tec/watchtower:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    command: --schedule "0 0 4 * * *" --cleanup
#-------------------------------------------------------------------------------
 Watchtower - Automatic Update of Containers/Apps
  watchtower:
    container_name: watchtower
    hostname: watchtower
    restart: always
    image: v2tec/watchtower:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    command: --schedule "0 0 4 * * *" --cleanup
#-------------------------------------------------------------------------------
#Declaration for the revrse proxy network
    networks:
      reverse-proxy:
        external:
          name: reverse-proxy
      default:
        driver: bridge
```



TROUBLESHOOTING - Nextcloud reverse-proxy troubleshooting

If you go to the global overview in your Nextcloud settings you may encounter the following message :

" Il y a quelques avertissements concernant votre configuration.

  PROBLEME REVERSE PROXY

    La configuration du serveur web ne permet pas d'atteindre "/.well-known/caldav". Vous trouverez plus d'informations dans la documentation.
    La configuration du serveur web ne permet pas d'atteindre "/.well-known/carddav". Vous trouverez plus d'informations dans la documentation. "

That's because your Nextcloud instance is located behind a reverse-proxy (traefik), you can get more information about this warning in the Nextcloud documentation : https://docs.nextcloud.com/server/16/go.php?to=admin-setup-well-known-URL

To get over this warning you need to add your traefik ip instance in the list of trusted revers-proxys.

In order to do that you can edit your Nextcloud configuration file in your Nextcloud instance : /config/config.php

And add the following lines at the end of the configuration file :

```
'trusted_proxies' =>
 array (
   0 => 'YOUR.DOMAINNAME',
   1 => 'YOURSERVERPRIVATEIP',
   2 => 'localhost',
  ),
```
You can now restart your Nextcloud instance, the problem should be solved.

---
title: "Server Rebuild"
author: "Opiskull"
tags: ["Docker", "Server"]
date: 2018-04-22T20:54:21+02:00
draft: true
---

Some weeks ago I upgraded my Server with some new hardware. So I had to setup the server. With getting a new server I also thought about using some new technology with it. So I tried to setup the server only with Docker containers.

# Kubernetes with Docker
At first I tried to setup Kubernetes since many large cloud platforms (Azure, Amazon...) are embracing Kubernetes. I got it work after my 3rd try. Kubernetes is rather complex :( although there exists well written documentation it is still a little challenge to set it up correctly.
So after I got it to work I tried to setup Teamspeak as my first container. 

I searched for an Kubernetes image of Teamspeak, but found none.
Next I tried to write my own Kubernetes file/image... I tried to build it from scratch but really failed badly. 

After this I finally understood that I tried to do too much at once. I don't need the complexity of Kubernetes since I only got one server. Kubernetes doesn't really make sense on one server. 

Maybe I should have thought about this sooner when I had to enable the master for the deployment of pods with the following command:

```bash
kubectl taint nodes --all node-role.kubernetes.io/master-
```

# Lets try only Docker

Now that I failed with Kubernetes lets go with Docker only. That should be easier and I already had some experience with Docker. I only did the tutorial at their site https://docs.docker.com/get-started/ but it counts as experience right? :P

Now I setup a server with docker only. My host system is Ubuntu so followed the instructions on https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-docker-ce.

```bash
apt-get update && apt-get install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

After the above lines I could simply update and install the `docker-ce` version.

```bash
apt-get update && apt-get install docker-ce
```

Now that Docker is running lets create our containers and Images that we need!

## Teamspeak

On the old server was a Teamspeak server that I needed to move to the new Server. I had already created a backup before from the old server. On the old Server I had Teamspeak running without problems but I never really liked it. It was a pain to upgrade... and I didn't really have any idea what I was doing.

At first I tried to use an existing image but somehow didn't like how it was setup (it had an dependency on an external service for getting the latest version number).

What should I do now? Why don't I create an Image myself that parses the Teamspeak website and always downloads the latest server?

Yup lets do that. So I started to write an script to open the website and parse the website and get the latest version ;)

I used an small app with the name pup https://github.com/ericchiang/pup and a little curl magic to download the website of Teamspeak. The result was a small oneliner with the following content

```bash
curl -s -L https://www.teamspeak.com/en/downloads | pup "[href*=\"teamspeak3-server_linux_amd64\"] attr{href}"
```

After the execution the result is the download URL like we wanted `http://dl.4players.de/ts/releases/3.1.1/teamspeak3-server_linux_amd64-3.1.1.tar.bz2`

You need to escape the " in the script above in the pup call as this is a bug. I tried it at first with single quotes (') but these do not work!

And now download the server with curl again:

```bash
curl -O -L $(curl -s -L https://www.teamspeak.com/en/downloads | pup "[href*=\"teamspeak3-server_linux_amd64\"] attr{href}")
```

With this we can download our server each time an new image is created!

Lets add extraction to our target folder:

```bash
mkdir -p /home/teamspeak && curl -L $(curl -s -L https://www.teamspeak.com/en/downloads | pup "[href*=\"teamspeak3-server_linux_amd64\"] attr{href}") | tar xj -C /home/teamspeak
```

Now I also needed to add pup to the image. pup was only available from github with a zip file. So i added unzip when I created the image unzip pup to the `/bin` folder and make it executable with `cmod a+x /bin/pup`.

So... now I was at a point where i thought i got it to work finally... But somehow... I asked myself again why does no good image exist for Teamspeak. I searched the docker-hub again and found it. My holy grail!! a official docker image of teamspeak!!! 

I removed all the files in my docker image... and removed all my work i had done until now... Yeah... that wasn't such a good idea... I also removed my `Dockerfile`... So i cannot post it now...

I created a `docker-compose.yaml` file with the following content

```yml
version: "3"
services:
  teamspeak:
    image: teamspeak
    ports:
      - "9987:9987/udp"
      - "10011:10011"
      - "30033:30033"
    volumes:
      - "/home/teamspeak:/var/ts3server"
    restart: always
```

I am using the `/home/teamspeak` directory because I need to migrate my files from the old server to the new one with all the files and with a license.

relatively unspectacular right? :/

So... What have I learned from this. Sometimes it is a good idea to look for something 2 times before rebuilding something from scratch ;D


## Minecraft

What is a server without a game server? ;D

One of my next projects was to setup a Minecraft server. That was relatively easy :) there exists a really really good image from itzg https://hub.docker.com/r/itzg/minecraft-server/

So I created a simple `docker-compose.yaml` file for starting a Minecraft server :)

```yml
version: '3'

services:
  minecraft:
    image: itzg/minecraft-server
    container_name: minecraft
    ports:
      - "25565:25565"
    volumes:
      - /home/minecraft:/data
      - /etc/localtime:/etc/localtime:ro
    environment:
      EULA: "TRUE"
      CONSOLE: "false"
      ENABLE_RCON: "true"
      RCON_PORT: 28016
      UID: 1001
      GID: 1001
      MOTD: "Opiskulls minecraft server powered by Docker"
    restart: always
```

I removed some parts from the config as those are internal :)

## Mailserver

On my old server I setup a mailserver from hand and always hat problems with postfix and could never really send emails correctly... The setup was somehow complicated and didn't work as it should ;)

So with this server I really had to use a preconfigured server so that updates would be easier and error prune. So I decided to use a Docker image with containers.

There where some really interesting Docker mailserver out there. I used the mailserver from https://github.com/tomav/docker-mailserver. I really have to say it was really pleasant to work with and really good examples on how to use the services it provides.

### First steps

I followed the information as provided on the readme page on the docker-mailserver:

```bash
docker pull tvial/docker-mailserver:latest
```

```bash
curl -o setup.sh https://raw.githubusercontent.com/tomav/docker-mailserver/master/setup.sh; chmod a+x ./setup.sh
curl -o docker-compose.yml https://raw.githubusercontent.com/tomav/docker-mailserver/master/docker-compose.yml.dist
curl -o .env https://raw.githubusercontent.com/tomav/docker-mailserver/master/.env.dist
```

Configure the environment as I needed and created my email account with

```bash
./setup.sh email add <user@domain> [<password>]
```

So now I started up my server

```bash
docker-compose up
```

And it worked! IT REALLY WORKED :D. So that is why I really love this image ;D

### DKIM

Domain validation on Namecheap

### Autodiscover/Autoconfig

Another nice feature I found was that autodiscover was also supported... I didn't even know that something like that exists ;). That is really really a nice feature... as i always failed add configuring my email cliet correctly.

As always for this also exists an nice image. This time I used  `weboaks/autodiscover-email-settings:latest`

That part of my docker-compose looked like that:

```yml
  autodiscover:
    image: weboaks/autodiscover-email-settings:latest
    container_name: mail_autodiscover
    environment:
    - DOMAIN=peerzone.net
    - IMAP_HOST=imap.peerzone.net
    - IMAP_PORT=993
    - SMTP_HOST=smtp.peerzone.net
    - SMTP_PORT=587
    labels:
      - traefik.enable=true
      - traefik.port=8000
      - traefik.frontend.rule=Host:autoconfig.peerzone.net,autodiscover.peerzone.net
      - traefik.docker.network=traefik-net
    networks:
    - traefik-net
    restart: always
```

I used traefik as my reverse proxy of choice as it is easy to manage and use. We can simply use labels to set the correct parts of traefik to forward how it should.

### WebMail

My webmail interface of choice was rainloop. I already used it before and liked how easy it was to setup and configure. This time I used `hardware/rainloop`

```yml
  web:
    image: hardware/rainloop
    container_name: mail_web
    labels:
      - traefik.enable=true
      - traefik.frontend.rule=Host:mail.peerzone.net
      - traefik.port=8888
      - traefik.docker.network=traefik-net
    volumes:
      - mail_web_data:/rainloop/data
    networks:
    - traefik-net
    restart: always
```

That image really needed no configuration at all only my configuration for traefik.

## Blog

Hosted on github. 
Created with Hugo. 
Webhook golang.

## Reverse Proxy

A reverse proxy is required so that multiple services can be used. as in my case the mailserver and my blog itself.

### Nginx 

First i tried to setup the reverse proxy with nginx and docker-gen from jwilder. The image was nice and worked...

but somehow i wanted something else.

### Traefik
Traefik is a nice and modern reverse proxy with the possibillity to read from the docker itself. We can configure the port and names that should be used with labels.



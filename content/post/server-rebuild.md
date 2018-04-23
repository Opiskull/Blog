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

`
kubectl taint nodes --all node-role.kubernetes.io/master-
`

# Lets try only Docker

Now that I failed with Kubernetes lets go with Docker only. That should be easier and I already had some experience with Docker. I only did the tutorial at their site https://docs.docker.com/get-started/ but it counts as experience right? :P

Now I setup a server with docker only. My host system is Ubuntu so followed the instructions on https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-docker-ce.

`
apt-get update && apt-get install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
`

After the above lines i could simply update and install the docker-ce version.

`
apt-get update && apt-get install docker-ce
`

Now that Docker is running lets create our containers and Images that we need!

## Teamspeak

On the old server was a teamspeak server that i needed to move over to the new server and that worked really good.

With the teamspeak server i needed to invest some of my time. First i tried to build it from stretch... But that didn't work... after one day searching i finally found the official teamspeak server image and that was really easy to integrate. I Setup my docker-compose.yml script with the licence in it and it worked on the first try.

## Minecraft

official minecraft image

## Mailserver

setup the mailserver
additional add rainloop as an webmail client
used mailserver and added rainloop with nginx server as the webserver

## Blog

Hosted on github. Created with Hugo. Webhook golang.

## Reverse Proxy

A reverse proxy is required so that multiple services can be used. as in my case the mailserver and my blog itself.

### Nginx 

First i tried to setup the reverse proxy with nginx and docker-gen from jwilder. The image was nice and worked...

but somehow i wanted something else.

### Traefik
Traefik is a nice and modern reverse proxy with the possibillity to read from the docker itself. We can configure the port and names that should be used with labels.



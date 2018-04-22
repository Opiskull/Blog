---
title: "Server Rebuild"
author: "Opiskull"
tags: ["Docker", "Server"]
date: 2018-04-22T20:54:21+02:00
draft: true
---

Some weeks ago I upgraded my Server with some new hardware. So i had to setup the server. The whole thought of setting up my new server i thought about

As always when you get something new you want todo something with it. So i tried to setup the server only with docker components.

# First to 4th try
on my first try i tried to setup kubernetes as every big cloud hoster is now using kubernetes. I got it work after my 3rd try. The setup of kubernetes is really not easy :(.
So after i got it to work i tried to setup teamspeak as my first server... the bad thing was... that there is no kubernetes image available for teamspeak... so i tried to build one from scratch but really failed badly :D.

After this i finally understood that i don't need the complexity of kubernetes as i only have one server. So Kubernetes doesn't really make sense on one server.

# Lets go Docker only

Now i setup a server with docker only. 



I used the following services with existing images.

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



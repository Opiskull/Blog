---
title: "My First Post"
date: 2018-03-05T21:19:50+01:00
tags: ["Docker","Tech","Nginx","DevOps"]
---

# My First Post

Hi there! 

If you have found this post then I have finally started to write my own blog. In this blog I will write about my experience with Programming, IT and other things.

Today is the first day so I will keep this short.

So last Friday I worked the first time with Docker and I got to say it's really cool. When I think at how much time I have thrown away with creating VMs or how often I have killed one of my systems when I installed something new. But now with Docker I can simply start a container and throw it away afterwards :D

For the people out there which sill don't know what Docker is I will try to explain it. 

Docker is a management interface for containers. Containers are lightweight VMs. Lightweight means that they don't need there own OS like a VM. They lay a layer over the host system and use somethings from the host. There is no need emulate the CPU or memory. You can use it from the host. Another thing that needs to be noted is that each Container is a VM for itself.


## A small example

In this small example we will try to get an NGINX server to run in some simple steps!

`docker run --name static-nginx -v D:/dev/sample-app:/usr/share/nginx/html:ro -d nginx`

With this command we create a container with the name `static-nginx`.

With `-v D:/dev/sample-app:/usr/share/nginx/html:ro` we share the `D:/dev/sample-app` folder from the host with the container with the name `/usr/share/nginx/html` in read only mode as we only need to read from this folder

With the `-d` switch we let it run in the background

And with `nginx` we say what image name. Normally a image name starts with a name like `opiskull/getting-started` bug NGINX is used directly.



So this was my first post I hope that my English wasn't that bad and that some people find it interesting like me,

so see you till next time :)
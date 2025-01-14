---
layout: post
title: "Docker - Ultimate Installation Guide"
date: 2025-01-05
categories: [infrastructure]
tags: [docker]
---

In this tutorial, I want to present the most convenient way of docker installation on Ubuntu based systems. I always use docker-compose for simplified and robust configuration, especially with multiple container applications. Another characteristic quality of my setup is that I want to manage docker from non-root (or admin) user account for more convenient work.

First, install docker and docker-compose:

```
$ sudo apt update
$ sudo apt install docker.io docker-compose
```

Now, add current user (or spceify the user name) to docker group:

```
$ sudo gpasswd -a ${USER} docker
```

Enable systemd unit for autostart:

```
$ sudo systemctl start docker
$ sudo systemctl enable docker
```

Check if everythiong was installed correctly:

```
$ docker -v
$ sudo systemctl status docker
$ docker-compose version
```
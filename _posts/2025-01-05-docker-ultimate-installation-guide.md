---
layout: post
title: "Docker - Ultimate Installation Guide"
date: 2025-01-05
categories: [infrastructure]
tags: [docker]
---

In this tutorial, I want to present the most convenient way of docker installation on Ubuntu based systems. I always use docker with docker-compose for simplified and robust configuration, especially with multiple container applications. Another characteristic quality of my setup is that I want to manage docker from non-root (or admin) user account for more convenient and secure work. I will also cover a basic example of creating and running containers with Docker Compose.

## Installing Docker and Docker Compose

First, update your package index and install Docker and Docker Compose with the following commands:

```bash
$ sudo apt update
$ sudo apt install docker.io docker-compose
```

## Adding Your User to the Docker Group

To run Docker commands without using sudo, add your user to the docker group. Replace ${USER} with different username if necessary:

``` bash
$ sudo gpasswd -a ${USER} docker
```

## Enabling Docker to Start on Boot

Enable and start the Docker service to ensure it runs on system boot:

``` bash
$ sudo systemctl start docker
$ sudo systemctl enable docker
```

## Verifying the Installation

Check if Docker and Docker Compose are installed correctly:

```bash
$ docker -v
$ sudo systemctl status docker
$ docker-compose version
```

You should see version information for Docker and Docker Compose if everything is set up correctly.

## Running a Simple Docker Container

To ensure Docker works as expected, try running a lightweight Alpine Linux container:

```bash
docker run -it --rm alpine:latest
```

This command launches an interactive shell in an Alpine container. You can play around and the container will be destroyed once you close the console The `--rm` flag ensures the container is deleted when you exit.

## Using Docker Compose with an Example

Docker Compose simplifies managing multi-container applications. Here is an example on how to run a simple web service using Nginx. Create file called  `docker-compose.yml`:

```yml
version: '3.8'

services:
    web:
        image: nginx:latest
        ports:
            - "8080:80"
        volumes:
            - ./html:/usr/share/nginx/html
```

Create an `html` directory in the same location with an `index.html` file containing a simple message, for example:

```html
<!DOCTYPE html> 
<html> 
    <head> <title>My Dockerized Website</title> </head> 
    <body> <h1>Welcome to my Docker-powered site!</h1> </body> 
</html>
```

Run the container using command (in the location of `docker-compose.yml`):

```bash
docker-compose up -d
```

You can now access your website at `http://localhost:8080`. To stop and remove the containers, use:

```bash
docker-compose down
```

With Docker and Docker Compose set up, you can explore the fascinating technology of containerisation. Enjoy your Docker journey!
---
layout: post
title: "Ansible Lab in Docker"
date: 2025-01-20
categories: [infrastructure]
tags: [docker]
---

If you are not familiar with docker setup, please take a look at my previous post [Docker - Ultimate Installation Guide](https://skaczmarek-dev.github.io/posts/docker-ultimate-installation-guide/).

Docker containers offer a lightweight, efficient way to create a local testing environment for Ansible. Unlike full virtual machines, containers allow you to set up isolated systems on your machine without the need for a hypervisor. This guide will walk you through creating a lab consisting of an Ansible master node and two managed nodes (Ubuntu and Rocky Linux) using Docker Compose. The environment will be configured in a way allowing Ansible to connect to the managed hosts via `ssh` and perform basic commands or run playbooks.

## Project Structure

The project will include the following files:

```plaintext
ansible_lab/
├── ansible/            # Directory for Ansible playbooks and configuration
├── docker-compose.yml  # Docker Compose configuration
├── dockerfile.master   # Dockerfile for Ansible Master
├── dockerfile.ubuntu   # Dockerfile for managed Ubuntu node
└── dockerfile.rocky    # Dockerfile for managed Rocky Linux node
```

## Creating Dockerfiles for Containers

### Dockerfile for Ansible Master

Create a Dockerfile specific to Ansible Master. Create a file called: `dockerfile.master`:

```
FROM ubuntu:latest

RUN apt-get update && apt-get install -y openssh-server sudo software-properties-common && apt-get clean

RUN add-apt-repository --yes --update ppa:ansible/ansible && \
    apt-get update && \
    apt-get install -y ansible

RUN useradd -m -s /bin/bash ansible && \
    echo "ansible:ansible" | chpasswd && \
    usermod -aG sudo ansible

RUN mkdir /var/run/sshd && ssh-keygen -A \
    && mkdir /home/ansible/.ssh \
    && chown -R ansible:ansible /home/ansible/.ssh

RUN echo "PermitRootLogin no" >> /etc/ssh/sshd_config && \
    echo "PasswordAuthentication yes" >> /etc/ssh/sshd_config && \
    echo "AllowUsers ansible" >> /etc/ssh/sshd_config

RUN chmod 644 /etc/ssh/sshd_config

EXPOSE 22
WORKDIR /home/ansible/playbooks
RUN mkdir -p /home/ansible/playbooks && chown -R ansible:ansible /home/ansible/playbooks
CMD ["/usr/sbin/sshd", "-D"]
```

### Dockerfile for Ubuntu Managed Host

Create a Dockerfile for Ubuntu Managed Host. Create a file called: `dockerfile.ubuntu`:

```
FROM ubuntu:latest

# Install necessary packages
RUN apt update && apt install -y \
    openssh-server \
    sudo \
    python3 \
    python3-pip \
    ansible && \
    apt clean

# Create a technical user
RUN useradd -m -s /bin/bash ansible && \
    echo "ansible:ansible" | chpasswd && \
    usermod -aG sudo ansible

# Configure SSH
RUN mkdir /var/run/sshd && \
    mkdir /home/ansible/.ssh && \
    chown -R ansible:ansible /home/ansible/.ssh

RUN echo "PermitRootLogin no" >> /etc/ssh/sshd_config && \
    echo "PasswordAuthentication yes" >> /etc/ssh/sshd_config && \
    echo "AllowUsers ansible" >> /etc/ssh/sshd_config

EXPOSE 22
WORKDIR /home/ansible

CMD ["/usr/sbin/sshd", "-D"]
```

### Dockerfile for Rocky Managed Host

Create a Dockerfile specific to Rocky Linux. Create a file called: `dockerfile.rocky`:

```
FROM rockylinux:9

# Install necessary packages
RUN dnf update -y && dnf install -y openssh-server sudo python3 && dnf clean all

# Create a technical user
RUN useradd -m -s /bin/bash ansible && echo "ansible:ansible" | chpasswd && usermod -aG wheel ansible

# Configure SSH
RUN mkdir /var/run/sshd && ssh-keygen -A \
    && mkdir -p /home/ansible/.ssh \
    && chown -R ansible:ansible /home/ansible/.ssh

RUN echo "PermitRootLogin no" >> /etc/ssh/sshd_config && \
    echo "PasswordAuthentication yes" >> /etc/ssh/sshd_config && \
    echo "AllowUsers ansible" >> /etc/ssh/sshd_config && \
    chmod 644 /etc/ssh/sshd_config

EXPOSE 22
WORKDIR /home/ansible

CMD ["/usr/sbin/sshd", "-D"]
```

## Creating the Docker Compose File

The `docker-compose.yml` file will define the Ansible environment with the following setup:

- A dedicated bridge network with static IP addresses for each container.
- Separate services for the Ansible Master and managed hosts.

Create the `docker-compose.yml` file:

```yaml
version: '3.9'
services:
  ansible-master:
    build:
      context: .
      dockerfile: dockerfile.master
    container_name: lab-ansible-master
    hostname: ansible-master
    networks:
      - ansible_net
    tty: true

  ubuntu-managed:
    build:
      context: .
      dockerfile: dockerfile.ubuntu
    container_name: lab-ubuntu-managed
    hostname: ubuntu-managed
    networks:
      - ansible_net
    tty: true

  rocky-managed:
    build:
      context: .
      dockerfile: dockerfile.rocky
    container_name: lab-rocky-managed
    hostname: rocky-managed
    networks:
      - ansible_net
    tty: true

networks:
  ansible_net:
    driver: bridge
```

## Deploying the Environment

Build and run the containers:

```bash
$ docker-compose up --build -d
```

Verify if containers were created:

```bash
$ docker ps
```

# Configuring the Ansible Inventory

First of all, we need to make sure, what ip address of our containers are. Look for the conatiners names from the previous step and check ip address of each one of them:

```bash
$ docker inspect <container-id> | grep "IPAddress"
```

Access the Ansible Master container:

```bash
$ docker exec -it lab-ansible-master bash
```

Create an Ansible inventory file:

- paste ip addresses obtained in the previous step
- paste password for ansible user

```bash
ansible@ansible-master:~$ cat /home/ansible/inventory 
[all:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_user=ansible
ansible_password=ansible

[ubuntu_managed]
ubuntu-managed ansible_host=<ip> ansible_port=2223

[rocky_managed]
rocky-managed ansible_host=<ip> ansible_port=2224
```

## Testing the Environment

Confirm that Ansible is installed:

```bash
$ ansible --version
```

Test connectivity to the managed hosts using the ping module:

```bash
$ ansible -i /home/ansible/inventory all -m ping
```

Expected Output (ip addresses may vary, depending on your setup):

```plaintext
172.26.0.3 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}

172.26.0.2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

This tutorial demonstrates how to set up a lightweight Ansible testing lab using Docker containers. By defining custom Dockerfiles and using Docker Compose, you can replicate this environment on your local machine, experiment with playbooks, and practice configuration management across different Linux distributions.

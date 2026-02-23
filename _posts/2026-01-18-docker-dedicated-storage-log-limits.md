---
layout: post
title: "Keep Docker Off Your Root Filesystem: Dedicated Storage and Log Limits Explained"
date: 2026-02-23
categories: [infrastructure]
tags: [docker]
---

In my previous posts about [LVM](https://skaczmarek-dev.github.io/posts/lvm/) and [Docker - Ultimate Installation Guide](https://skaczmarek-dev.github.io/posts/docker-ultimate-installation-guide/), I covered the basics of setting up logical volumes and installing Docker. Today, I want to address a critical issue that can cause serious problems in production environments: **uncontrolled Docker log growth** and **containers size**.

## The Problem with Docker Logs

By default, Docker containers write logs without any size limits. Over time, these logs can consume all available disk space on your system partition, potentially causing:

- System crashes due to full disk
- Failed deployments
- Database corruption
- Service interruptions

This is especially problematic when Docker is installed on the system partition (`/var/lib/docker`), which is by default, and where space is typically limited.

## The Solution: Separate Storage for Docker

The best practice is to keep Docker on a separate disk or logical volume. This prevents container logs from filling up your system partition and makes resource management much easier. There are a couple of methods to do that:

- Create symbolic link within the docker default location to different partition
- Mount another partition in `/var/lib/docker`
- Change properties of docker's systemd unit

In this article I will cover the first method, because I find it the most practical.

### Step 1: Prepare Your Storage

First, create a dedicated LVM volume for Docker (you can skip LVM if you want to use whole partition or disk, but I like to keep my setup flexible with LVM). If you're not familiar with LVM setup, check out my [LVM article](https://skaczmarek-dev.github.io/posts/lvm/) for detailed instructions.

For this example, I'll assume you have a mount point ready at `/mnt/data_docker`.

### Step 2: Move Docker to the New Location

Before installing Docker, create a symbolic link from `/var/lib/docker` to your dedicated storage:

```bash
sudo ln -s /mnt/data_docker /var/lib/docker
```

**Important:** This must be done **before** installing Docker. If you already have Docker installed, you'll need to:

1. Stop Docker: `sudo systemctl stop docker`
2. Move existing data: `sudo mv /var/lib/docker /mnt/data_docker`
3. Create the symlink as shown above
4. Start Docker: `sudo systemctl start docker`

### Step 3: Install Docker

Now proceed with the standard Docker installation as described in my [Docker - Ultimate Installation Guide](https://skaczmarek-dev.github.io/posts/docker-ultimate-installation-guide/):

```bash
sudo apt update -y
sudo apt install docker.io docker-compose python3-setuptools -y
sudo gpasswd -a ethadmin docker
```

Enable and start Docker:

```bash
sudo systemctl start docker
sudo systemctl enable docker
sudo systemctl status docker
```

### Step 4: Configure Log Rotation

Even with separate storage, it's crucial to limit log sizes. Create or edit `/etc/docker/daemon.json`:

```bash
sudo vim /etc/docker/daemon.json
```

Add the following configuration:
```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

This configuration:
- Limits each log file to 10MB (`max-size`)
- Keeps only 3 rotated log files per container (`max-file`)
- Effectively caps each container's logs at ~30MB total

### Step 5: Restart Docker

Apply the new configuration:

```bash
sudo systemctl restart docker
sudo systemctl status docker
```

## Verifying Your Setup

Check that Docker is using the correct storage location:

```bash
docker info | grep "Docker Root Dir"
```

You should see your custom path (e.g., `/mnt/data_docker`).

To verify log rotation is working, check an existing container's log configuration:

```bash
docker inspect <container_name> | grep -A 10 LogConfig
```

## Benefits of This Approach

✅ **System partition protection** - Logs can't fill up your OS disk  
✅ **Better resource allocation** - Dedicated storage for Docker data  
✅ **Easier monitoring** - Separate mount point to track Docker storage usage  
✅ **Controlled log growth** - Automatic rotation prevents runaway log files  
✅ **Improved performance** - Potentially faster I/O on dedicated storage

## Additional Considerations

For existing containers, the new log limits only apply to newly created containers. To apply limits to running containers, you'll need to recreate them:

```bash
docker-compose down
docker-compose up -d
```

Remember to monitor your Docker storage regularly:

```bash
df -h /mnt/data_docker
docker system df
```

This setup has saved me from multiple disk-full incidents and is now a standard part of my Docker deployment process. Combined with proper LVM management and the installation practices from my previous guides, you'll have a robust and maintainable Docker environment.

Happy containerizing! 🐳

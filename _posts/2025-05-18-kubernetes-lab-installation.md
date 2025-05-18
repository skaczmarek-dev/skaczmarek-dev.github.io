---
layout: post
title: "Lightweight Kubernetes Lab with k3s on Virtual Machines"
date: 2025-05-18
categories: [infrastructure]
tags: [docker, kubernetes]
---

If you're new to Kubernetes, you might want to explore a lightweight and easy-to-install option like [K3s](https://k3s.io/). It's a great choice for home labs, edge computing, or testing environments, especially when combined with tools like Proxmox.

In this guide, we'll walk through setting up a minimal K3s cluster with one master node and one or more worker nodes. We'll also deploy a sample NGINX application to verify everything works as expected.

## Why K3s?

K3s is a simplified, production-ready Kubernetes distribution. Here are a few of its perks:

- Single binary install
- Low system requirements
- Built-in containerd
- Fully compatible with `kubectl`

## VM Requirements

A minimal setup on a virtual environment like Proxmox might look like this:

| Role         | vCPU | RAM   | System Disk | Data Disk | OS               |
|--------------|------|-------|-------------|-----------|------------------|
| Master Node  | 2    | 2–4GB | 16 GB       | 10 GB+    | Any Linux distro |
| Worker Node  | 2    | 2–4GB | 16 GB       | 10 GB+    | Any Linux distro |

**Note:** The data disk (`/dev/vda` in this example) will be mounted as `/var/lib/rancher`, where K3s stores its data.

## Installation Scripts

After creating the VMs, log into them and copy the scripts separately for master and worker node.

**Note:** The scripts will format additional disks, so make sure their names in the `DISK_DEVICE` variable are correct.

### Master Node Setup Script

`k3s-master-setup.sh`

```bash
#!/bin/bash

set -e

DISK_DEVICE="/dev/vda"
MOUNT_POINT="/mnt/k3s-data"
K3S_INSTALL_URL="https://get.k3s.io"
NEW_HOSTNAME="k3s-master"

echo "[+] Setting hostname to ${NEW_HOSTNAME}..."
hostnamectl set-hostname ${NEW_HOSTNAME}

echo "[+] Formatting ${DISK_DEVICE}..."
mkfs.ext4 -F ${DISK_DEVICE}

echo "[+] Mounting to ${MOUNT_POINT}..."
mkdir -p ${MOUNT_POINT}
mount ${DISK_DEVICE} ${MOUNT_POINT}
UUID=$(blkid -s UUID -o value ${DISK_DEVICE})
echo "UUID=${UUID} ${MOUNT_POINT} ext4 defaults 0 2" >> /etc/fstab

mkdir -p ${MOUNT_POINT}/{rancher,kubelet,containerd}
mv /var/lib/rancher /var/lib/rancher.bak 2>/dev/null
mv /var/lib/kubelet /var/lib/kubelet.bak 2>/dev/null
mv /var/lib/containerd /var/lib/containerd.bak 2>/dev/null

ln -s ${MOUNT_POINT}/rancher /var/lib/rancher
ln -s ${MOUNT_POINT}/kubelet /var/lib/kubelet
ln -s ${MOUNT_POINT}/containerd /var/lib/containerd

echo "[+] Installing K3s (Master)..."
curl -sfL ${K3S_INSTALL_URL} | sh -

echo "[✓] Installation complete. Node token:"
cat /var/lib/rancher/k3s/server/node-token
```

### Worker Node Setup Script

`k3s-worker-setup.sh` 

**Note:** Replace `PASTE_MASTER_IP_HERE` and `PASTE_YOUR_TOKEN_HERE` accordingly.

```bash
#!/bin/bash

set -e

DISK_DEVICE="/dev/vda"
MOUNT_POINT="/mnt/k3s-data"
K3S_INSTALL_URL="https://get.k3s.io"
MASTER_IP="PASTE_MASTER_IP_HERE"
K3S_TOKEN="PASTE_YOUR_TOKEN_HERE"
NEW_HOSTNAME="k3s-exp-worker1"

echo "[+] Setting hostname to ${NEW_HOSTNAME}..."
hostnamectl set-hostname ${NEW_HOSTNAME}

echo "[+] Formatting ${DISK_DEVICE}..."
mkfs.ext4 -F ${DISK_DEVICE}

echo "[+] Mounting to ${MOUNT_POINT}..."
mkdir -p ${MOUNT_POINT}
mount ${DISK_DEVICE} ${MOUNT_POINT}
UUID=$(blkid -s UUID -o value ${DISK_DEVICE})
echo "UUID=${UUID} ${MOUNT_POINT} ext4 defaults 0 2" >> /etc/fstab

mkdir -p ${MOUNT_POINT}/{rancher,kubelet,containerd}
mv /var/lib/rancher /var/lib/rancher.bak 2>/dev/null
mv /var/lib/kubelet /var/lib/kubelet.bak 2>/dev/null
mv /var/lib/containerd /var/lib/containerd.bak 2>/dev/null

ln -s ${MOUNT_POINT}/rancher /var/lib/rancher
ln -s ${MOUNT_POINT}/kubelet /var/lib/kubelet
ln -s ${MOUNT_POINT}/containerd /var/lib/containerd

echo "[+] Installing K3s (Worker)..."
curl -sfL ${K3S_INSTALL_URL} | K3S_URL="https://${MASTER_IP}:6443" K3S_TOKEN="${K3S_TOKEN}" sh -

echo "[✓] Worker joined the cluster"
```

Run both scripts as root:

```
# chmod +x k3s-master-setup.sh
# ./k3s-master-setup.sh
```

Same thing for the worker node:

```
# chmod +x k3s-worker-setup.sh
# ./k3s-worker-setup.sh
```

## Deploying a Test Application

Once the cluster is ready, let's deploy a simple NGINX app to test everything.

### Create Deployment

```bash
kubectl create deployment hello --image=nginx
```

### Customize Index Page

```bash
echo "Hello from Kubernetes Cluster!" > index.html
POD_NAME=$(kubectl get pod -l app=hello -o jsonpath="{.items[0].metadata.name}")
kubectl cp index.html $POD_NAME:/usr/share/nginx/html/index.html
rm index.html
```

### Expose the Service

```bash
kubectl expose deployment hello --port=80 --type=NodePort
```

### Get the NodePort

```bash
# kubectl get svc hello
NAME    TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
hello   NodePort   10.43.255.208   <none>        80:31884/TCP   115s
```

### Access in Browser

Navigate to:

```
http://<MASTER_NODE_IP>:<NodePort>
http://<WORKER_NODE_IP>:<NodePort>
```

In my case, the nodes have ip addresses 192.168.1.142 and 192.168.1.178, and the port checked in the previous command is 31884:

## Cleanup

To remove the test deployment and service:

```bash
kubectl delete svc hello
kubectl delete deployment hello
```

K3s is a fantastic way to get hands-on with Kubernetes without the overhead of full-scale setups. It’s perfect for learning, testing, and home lab experimentation.

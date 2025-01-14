---
layout: post
title: "Ansible Lab in Docker"
date: 2025-01-20
categories: [infrastructure]
tags: [docker]
---

If you are not familiar with docker setup, please have a look at my previous post ...

Docker containers provide a lightweight and efficient solution for testing Ansible. Unlike traditional virtual machines that require a hypervisor, containers allow you to set up a complete environment on your local machine quickly. This tutorial will guide you through creating a test Ansible environment consisting of three Docker containers:

- Ansible Master - The controller node with Ansible installed.
- Ubuntu Managed Host - A managed node running Ubuntu.
- Rocky Managed Host - A managed node running Rocky Linux.

You will configure the environment so that Ansible can connect to the managed hosts via SSH and perform basic commands or run playbooks.

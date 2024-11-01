---
layout: post
title: "Memory usage investigation"
date: 2024-11-02
categories: [admin]
tags: [linux]
---

When troubleshooting performance issues on a Linux system, it’s often helpful to examine how memory and swap are being utilized. High RAM or swap usage can signal which processes are consuming excessive resources, potentially causing slowdowns. This guide provides quick commands to identify memory-intensive and swap-heavy processes, making it a handy tool for diagnosing system issues in real-time.

## Checking RAM and Swap Availability

The `free` command provides a snapshot of memory and swap usage, showing both total and available resources. This quick overview is helpful for understanding how much RAM and swap space your system currently has in use and how much is free.

```
# free -h
```

The `-h` option displays the output in human-readable format (e.g., MB or GB).

The output of free -h is divided into rows and columns. Here’s what to look for:

- Mem: Shows total, used, and free RAM. The “available” column provides a good estimate of how much RAM is available for new applications.
- Swap: Displays the total, used, and free swap space. If swap usage is high, it can indicate that your system is running low on RAM and is offloading memory to swap, which is slower.

Example:

```
              total        used        free      shared  buff/cache   available
Mem:           15G         6.5G       1.2G        400M         7.3G          8G
Swap:          2G          500M       1.5G
```

In this example:

- 8G of RAM is available.
- 1.5G of swap is free.

The `free` command is a quick way to gauge memory and swap status at a glance, making it especially useful when diagnosing memory-related issues.

## List processes by memory usage

To see which processes are consuming the most RAM, use the following command:

```
# ps aux --sort=%mem | head
```

This command lists all running processes, sorted by memory usage in descending order. Only the top entries are shown, making it easy to identify memory-heavy processes.

## List processes by swap usage

If you need to investigate swap usage specifically, run this command:

```
for file in /proc/*/status; do awk '/VmSwap|Name/{printf $2 " " $3}END'
```

This script checks each process for its swap usage by reading data from the /proc directory. Sorting by swap usage, it displays only the largest swap-consuming processes, which is useful for identifying those processes that rely heavily on swap space.

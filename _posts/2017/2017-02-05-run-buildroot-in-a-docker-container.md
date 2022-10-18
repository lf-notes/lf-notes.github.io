---
layout: default
title: Run Buildroot in a Docker Container
tags: buildroot docker windows linux
comments: true
---
# Run Buildroot in a Docker Container

This post explores how you can run Buildroot in a Docker container. Follow the [getting started](https://docs.docker.com/docker-for-windows/) to install Docker. I prefer using Docker with the PowerShell prompt on Windows 10.

![docker-ubuntu-windows-10.PNG](/assets/img/docker-ubuntu-windows-10.png)

To download and run the ubuntu image in a new container

```bash
docker run -it ubuntu bash
```

From another command prompt, run the following to find container id

```bash
docker ps -l
```

Use the `-a` option to see all containers

```bash
docker ps -a
```

Type `exit` to exit bash shell and stop container.

To return to container created earlier

```bash
docker start -ai container_id
```

Update apt package cache so that you can search and install additional packages

```bash
sudo apt update
```

You should now be able to search

```bash
sudo apt search wget
```

And install your favorite tools

```bash
sudo apt install wget
```

Obtain Buildroot

```bash
wget https://buildroot.org/downloads/buildroot-2016.11.2.tar.gz
```

Untar Buildroot

```bash
tar xvzf buildroot-2016.11.2.tar.gz
```

Install dependencies required to run Buildroot

```bash
sudo apt install patch cpio python unzip rsync bc bzip2 ncurses-dev git make g++
```

Go ahead and build your [Linux system](_posts/2014/2014-07-15-embedded-linux-system-for-raspberry-pi-with-buildroot.md).

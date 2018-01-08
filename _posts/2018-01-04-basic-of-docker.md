---
title: about docker
description: Docker is an open platform for developers and sysadmins to build, ship, and run distributed applications, whether on laptops, data center VMs, or the cloud.
categories:
 - docker
tags: docker
---

## install
- [centos 7](https://stackoverflow.com/questions/43869867/installing-docker-17-version-on-centos-7)

```shell
wget -qO- https://get.docker.com/ | sh
```


## base cmd

```shell
sudo service docker start
docker ps
docker run
```

## How to get into a docker container?
```shell
docker exec -it container-id /bin/bash
# docker attach container-id
```

## How to install “ifconfig” command in ubuntu docker image?

```shell
apt-get install net-tools
```

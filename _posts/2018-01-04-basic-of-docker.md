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
#启动docker damon
sudo service docker start

#启动容器
docker run container-id

#删除本地镜像
docker image rm -f image-id

#查看正在运行的容器
docker ps

#查看曾经运行过的容器
docker ps -a

#停止容器
docker stop container-id
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

---
title: about docker
description: Docker is an open platform for developers and sysadmins to build, ship, and run distributed applications, whether on laptops, data center VMs, or the cloud.
categories:
 - docker
tags: docker
photos:
- /assets/images/posts/2018-01-04-basic-of-docker-1.png
---

![img](/assets/images/posts/2018-01-04-basic-of-docker-1.png)

## install

- [centos 7](https://stackoverflow.com/questions/43869867/installing-docker-17-version-on-centos-7)

```shell
$ wget -qO- https://get.docker.com/ | sh
```

- [Ubuntu](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/#install-using-the-repository)

```shell
$ curl -fsSL get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
```

## cmd

```shell
#启动docker damon
sudo service docker start

#新建容器并启动
docker run image-id

#启动/停止/重启容器
docker start/stop/restart container-id

#搜索镜像
docker search image-name

#查看正在运行的容器
docker ps

#查看曾经运行过的容器
docker ps -a

#删除本地镜像
docker image rm -f image-id

#删除容器
docker rm container-id

#查看容器cpu等占用情况
docker stats -a

#进入容器bash
docker exec -it container-id /bin/bash
# docker exec -it container-id sh
# docker attach container-id #not recommended
```
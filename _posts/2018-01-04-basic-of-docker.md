---
title: about docker
description: Docker is an open platform for developers and sysadmins to build, ship, and run distributed applications, whether on laptops, data center VMs, or the cloud.
categories:
 - docker
tags: docker
---


## base cmd
```shell
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

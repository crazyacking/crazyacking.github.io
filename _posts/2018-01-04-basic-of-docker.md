---
title: docker
description: docker基础
categories:
 - docker
tags: docker
---


## base cmd

```shell
docker ps
docker run
```

##How to get into a docker container?

```shell
docker exec -it container-id /bin/bash
# docker attach container-id
```
## How to install “ifconfig” command in ubuntu docker image?

```shell
apt-get install net-tools
```

---
title: about docker
description: Docker is an open platform for developers and sysadmins to build, ship, and run distributed applications, whether on laptops, data center VMs, or the cloud.
categories:
 - docker
tags: docker
photos:
- /assets/images/posts/2018-01-04-basic-of-docker-1.png
---

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
docker run {image-id}

#启动/停止/重启容器
docker start/stop/restart {container-id}

#搜索镜像
docker search {image-name}

#查看正在运行的容器
docker ps

#查看曾经运行过的容器
docker ps -a

#删除本地镜像
docker rmi {image-id}
docker image rm -f {image-id}

#删除容器
docker rm {container-id}

#重命名容器
docker rename old_name new_name

#查看容器cpu等占用情况
docker stats -a

#查看docker运行日志
docker logs -f {container-id}

#进入容器bash
docker exec -it {container-id} /bin/bash
# docker exec -it {container-id} sh
# docker attach {container-id} #not recommended

#新增映射端口
docker stop test01
docker commit test01 test02
docker run -p 8080:8080 -td test02

#保存镜像到本地
docker save {image-id} > {file-name}
sudo docker save busybox > /root/busybox.tar
#加载本地镜像
docker load -i {file-name}
docker load < {file-name}
docker load < /root/busybox.tar

#打tag
docker tag {IMAGE_ID} {REPOSITORY}:{TAG}

#磁盘清理
#https://stackoverflow.com/questions/32723111/how-to-remove-old-and-unused-docker-images
docker system prune -a
#You also have:
docker image prune
docker container prune
docker network prune
docker volume prune

#登录 （注意区分uid和邮箱）
docker login
```

## action

``` shell
# 创建数据卷(not recommended)
docker volume create --name {volume-name}

# 安装gitlab
# https://docs.gitlab.com/omnibus/docker/README.html
sudo docker run --detach \
    --hostname {hostname} \
    --publish 443:443 --publish 80:80 --publish 22:22 \
    --name gitlab \
    --restart always \
    --volume /srv/gitlab/config:/etc/gitlab \
    --volume /srv/gitlab/logs:/var/log/gitlab \
    --volume /srv/gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ce:9.3.11-ce.0

# 安装nexus
docker volume create --name nexus-data

docker run -d \
    -p 8081:8081 \
    --name nexus \
    -v nexus-data:/nexus-data \
    sonatype/nexus3

# 安装influxDB
# 生成默认配置文件：
docker run --rm influxdb influxd config > influxdb.conf

# 创建influxDB容器：
docker run --name influxdb -d \
      -p 8086:8086 \
      -p 8083:8083 \
      -p 2003:2003 \
      --restart always \
      -e INFLUXDB_ADMIN_ENABLED=true \
      -e INFLUXDB_GRAPHITE_ENABLED=true \
      -e INFLUXDB_HTTP_AUTH_ENABLED=true \
      -e INFLUXDB_ADMIN_USER=admin \
      -e INFLUXDB_ADMIN_PASSWORD='123456' \
      -e INFLUXDB_READ_USER=reader \
      -e INFLUXDB_READ_USER_PASSWORD='123456' \
      -v /mnt/influxdb:/var/lib/influxdb \
      -v /mnt/influxdb/influxdb.conf:/etc/influxdb/influxdb.conf:ro \
      influxdb -config /etc/influxdb/influxdb.conf
```

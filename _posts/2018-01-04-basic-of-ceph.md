---
title: about ceph
description: 
categories:
 - ceph
tags: ceph
photos:
- 
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
# [radosgw-admin] 
# http://docs.ceph.com/docs/giant/man/8/radosgw-admin/

# list bucket
radosgw-admin bucket list

# create user
radosgw-admin user create --display-name="Dylan" --uid=dylan

# remove user
radosgw-admin user rm --uid=dylan

# remove user and clear their bucket
radosgw-admin user rm --uid=dylan --purge-data

# remove a bucket
radosgw-admin bucket unlink --bucket=foo



```

## deploy

``` shell
ssh-copy-id cephadmin@node1

Host node1
   Hostname node1
   User cephadmin
Host node2
   Hostname node2
   User cephadmin
Host node3
   Hostname node3
   User cephadmin
Host node4
   Hostname node3
   User cephadmin

yum install *argparse* -y
rm -f /var/run/yum.pid
yum --enablerepo=Ceph clean metadata

yum remove -y ceph-common
ceph-deploy purgedata node1 node2 node3 node4
ceph-deploy forgetkeys
ceph-deploy purge node1 node2 node3 node4

ceph-deploy install node1 node2 node3 node4

ceph-deploy osd prepare node2:/var/local/osd0 node3:/var/local/osd1 node4:/var/local/osd2
chmod 777 /var/local/osd0
ceph-deploy osd activate node2:/var/local/osd0 node3:/var/local/osd1 node4:/var/local/osd2

ceph-deploy admin node1 node2 node3 node4

ceph-deploy mds create node2
ceph-deploy rgw create node3

ceph -w

192.168.0.96 node1
192.168.0.97 node2
192.168.0.98 node3
192.168.0.99 node4
192.
ceph-deploy --overwrite-conf mon add node3

# 读写测试
ceph osd pool create data 64 64
echo test-data > testfile.txt
rados put test-object-1 testfile.txt --pool=data

ceph-deploy add mon失败:https://www.zybuluo.com/dyj2017/note/920621

ceph-deploy osd prepare node2:/path/to/directory

sudo rpm -Uvh --replacepkgs http://download.ceph.com/rpm-hammer/el7/noarch/ceph-release-1-0.el7.noarch.rpm


sudo rpm -Uvh --replacepkgs --force https://download.ceph.com/rpm-jewel/el7/noarch/ceph-release-1-0.el7.noarch.rpm

node2][WARNIN] python-requests-2.6.0-1.el7_1.noarch 有缺少的需求 python-urllib3 >= ('0', '1.10.2', '1')

sudo rpm -Uvh --replacepkgs --force http://download.ceph.com/rpm-jewel/el7/x86_64/ceph-common-10.2.10-0.el7.x86_64.rpm
sudo rpm -Uvh --replacepkgs --force http://download.ceph.com/rpm-jewel/el7/x86_64/ceph-base-10.2.10-0.el7.x86_64.rpm
sudo rpm -Uvh --replacepkgs --force http://download.ceph.com/rpm-jewel/el7/x86_64/ceph-mds-10.2.10-0.el7.x86_64.rpm
sudo rpm -Uvh --replacepkgs --force http://download.ceph.com/rpm-jewel/el7/x86_64/ceph-mon-10.2.10-0.el7.x86_64.rpm
sudo rpm -Uvh --replacepkgs --force http://download.ceph.com/rpm-jewel/el7/x86_64/ceph-osd-10.2.10-0.el7.x86_64.rpm
sudo rpm -Uvh --replacepkgs --force http://download.ceph.com/rpm-jewel/el7/x86_64/ceph-radosgw-10.2.10-0.el7.x86_64.rpm
sudo rpm -Uvh --replacepkgs --force http://download.ceph.com/rpm-jewel/el7/x86_64/ceph-selinux-10.2.10-0.el7.x86_64.rpm

yum install * -y 

pip uninstall urllib3
yum install python-urllib3
没有可用软件包 ceph。
没有可用软件包 ceph-radosgw。

[node3][DEBUG ]   正在安装    : 1:ceph-base-10.2.10-0.el7.x86_64                            2/8
[node3][DEBUG ]   正在安装    : 1:ceph-selinux-10.2.10-0.el7.x86_64                         3/8
[node3][DEBUG ]   正在安装    : 1:ceph-mds-10.2.10-0.el7.x86_64                             4/8
[node3][DEBUG ]   正在安装    : 1:ceph-mon-10.2.10-0.el7.x86_64                             5/8
[node3][DEBUG ]   正在安装    : 1:ceph-osd-10.2.10-0.el7.x86_64                             6/8
[node3][DEBUG ]   正在安装    : 1:ceph-10.2.10-0.el7.x86_64                                 7/8
[node3][DEBUG ]   正在安装    : 1:ceph-radosgw-10.2.10-0.el7.x86_64

filestore xattr use omap



radosgw-admin user create --uid=mengyu --display-name="mengyu" --email=crazyacking@gmail.com
```

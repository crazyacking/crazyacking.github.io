---
title: about ceph
description: Ceph is a unified, distributed storage system designed for excellent performance, reliability and scalability.
categories:
 - ceph
tags: ceph
photos:
- 
---

<img src="/assets/images/posts/2018-03-18-basic-of-ceph-1.png" style="zoom:50%" />

## cmd

```bash
#[radosgw-admin命令集] 
#http://docs.ceph.com/docs/giant/man/8/radosgw-admin/

#list bucket
radosgw-admin bucket list
#create user
radosgw-admin user create --display-name="dylan" --uid=dylan
#user info
radosgw-admin user info --uid=mengyu
#remove user
radosgw-admin user rm --uid=dylan
#remove user and clear their bucket
radosgw-admin user rm --uid=dylan --purge-data
#remove a bucket
radosgw-admin bucket rm --purge-objects --bucket=foo
#radosgw-admin bucket unlink --bucket=foo
#show usage information for user
radosgw-admin usage show --uid=mengyu --start-date=2018-03-12 --end-date=2018-03-14
#show only summary of usage information for all users
radosgw-admin usage show --show-log-entries=false
#show rados pools
rados lspools
#show objects in pool
rados -p {pool-name} ls

#[ceph命令集]
#http://docs.ceph.com/docs/master/man/8/ceph/
#add a bucket
ceph osd crush add-bucket <name> <type>
ceph osd crush add-bucket demo rack
#view the CRUSH hierarchy for your cluster
ceph osd crush tree

#Link bucket to specified user
radosgw-admin bucket link --bucket=default --bucket_id="-1" --uid=mengyu

#读写测试
echo {Test-data} > testfile.txt
rados put {object-name} {file-path} --pool=demo
rados put test-object-1 testfile.txt --pool=demo

#[集群管理]
#查看集群状态
ceph -s
ceph status
ceph health
ceph health detail
ceph mon stat
#观察集群内正发生的事件
ceph -w
#检查集群的数据用量
ceph df
#检查osd状态
ceph osd ls
ceph osd stat
ceph osd tree
ceph osd dump
#Ceph CRUSH树状态
ceph osd tree
#检查监视器状态
ceph mon stat
ceph mon_status
ceph mon dump
#检查监视器的法定人数状态
ceph quorum_status
#检查MDS状态
ceph mds stat
ceph mds dump
#获取归置组信息
ceph pg dump
#找出故障归置组
ceph pg dump_stuck [unclean|inactive|stale|undersized|degraded]
#查看mon状态 仲裁结果
ceph daemon mon.node1 mon_status

#列出节点上的磁盘
ceph-deploy disk list node1

#日志地址
/var/log/ceph/
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

sudo yum update && sudo yum install ceph-deploy
mkdir my-cluster
cd my-cluster
ceph-deploy new node1

ceph-deploy install node1 node2 node3 node4

ceph-deploy mon create-initial

chmod 777 /var/local/osd0
ceph-deploy osd prepare node2:/var/local/osd0 node3:/var/local/osd1 node4:/var/local/osd2
ceph-deploy osd activate node2:/var/local/osd0 node3:/var/local/osd1 node4:/var/local/osd2
ceph-deploy admin node1 node2 node3 node4

ceph-deploy mds create node2
ceph-deploy rgw create node3
#然后看rgw的端口号

ceph-deploy --overwrite-conf mon add node3

#读写测试
ceph osd pool create data 64 64
echo test-data > testfile.txt
rados put test-object-1 testfile.txt --pool=data
rados -p data ls
ceph osd map data test-object-1
rados rm test-object-1 --pool=data

ceph-deploy add mon失败: https://www.zybuluo.com/dyj2017/note/920621

ceph-deploy osd prepare node2:/path/to/directory

sudo rpm -Uvh --replacepkgs http://download.ceph.com/rpm-hammer/el7/noarch/ceph-release-1-0.el7.noarch.rpm

sudo rpm -Uvh --replacepkgs --force https://download.ceph.com/rpm-jewel/el7/noarch/ceph-release-1-0.el7.noarch.rpm

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

pip install s3cmd
pip install awscli
```


查看Linux发行版

```bash
cat /etc/redhat-release
CentOS Linux release 7.4.1708 (Core)
```

在所有 Ceph 节点上安装 NTP 服务，以免因时钟漂移导致故障

```bash
sudo yum install ntp ntpdate ntp-doc
```

新建ceph安装工作空间

```bash
mkdir -p ~/ceph-workspace/rpm/deploy
```

复制ceph-deploy到当前目录

```bash
scp ceph-deploy-1.5.39-0.noarch.rpm root@128.128.98.59:/root/ceph-workspace/rpm/deploy
```

安装ceph-deploy

```bash
rpm -ivh ceph-deploy-1.5.39-0.noarch.rpm
```

由于ceph-deploy必须在普通用户下运行，故需创建ceph deploy用户

```bash
export username='cephadmin'
sudo useradd -d /home/${username} -m ${username}
sudo passwd ${username}
emas-ceph-pswd
su cephadmin && cd ~ && pwd
```

给ceph deploy用户添加免密执行sudo权限

```bash
echo "${username} ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/${username}
sudo chmod 0440 /etc/sudoers.d/${username}
```

开放所需端口

```bash
#若机器使用的是firewalld
sudo firewall-cmd --zone=public --add-port=6789/tcp --permanent
sudo firewall-cmd --zone=public --add-port=6789/udp --permanent
sudo firewall-cmd --permanent --zone=public --add-port=6800-7300/tcp
sudo firewall-cmd --permanent --zone=public --add-port=6800-7300/udp
sudo firewall-cmd --reload

#若机器使用的是iptables
sudo iptables -A INPUT -i {iface} -p tcp -s {ip-address}/{netmask} --dport 6789 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 6789:6789 -j ACCEPT
sudo iptables -A INPUT -p udp --dport 6789:6789 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 6800:7300 -j ACCEPT
sudo iptables -A INPUT -p udp --dport 6800:7300 -j ACCEPT

/sbin/service iptables save
```


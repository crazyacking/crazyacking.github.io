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


## 搭建ceph

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
yum -y install ceph-deploy
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

搭建本地源

```bash
本地从poc-ceph下载：
scp "root@101.132.192.133:/root/packages/*.rpm" .
rsync -chavzP --stats -a --ignore-existing root@101.132.192.133:/root/packages/*.rpm .

本地上传到跳板机：
scp *.rpm root@128.128.98.14:/root/ceph-rpm/dependences/2
rsync -chavzP --stats -a --ignore-existing *.rpm root@128.128.98.14:/root/ceph-rpm/dependences

跳板机上传到worker1：
scp *.rpm root@128.128.98.59:/root/ceph-workspace/rpm/dependences/2
rsync -chavzP --stats -a --ignore-existing *.rpm root@128.128.98.59:/root/ceph-workspace/rpm/ceph-yum-repo

createrepo /root/ceph-workspace/rpm/ceph-yum-repo ; yum clean all ; yum makecache

vim /etc/yum.repos.d/ceph-local.repo
[ceph-local]
name=ceph-local
baseurl=file:///root/ceph-workspace/rpm/ceph-yum-repo
gpgcheck=0
enabled=1
```

安装ceph

```
#先手动安装最坑的两个依赖
cd /root/ceph-workspace/rpm/ceph-yum-repo
rpm -ivh userspace-rcu-0.7.16-1.el7.x86_64.rpm
rpm -ivh lttng-ust-2.4.1-4.el7.x86_64.rpm

#安装ceph
yum -y install ceph ceph-release ceph-common ceph-radosgw
```

使用ceph-deploy创建集群

```bash
mkdir -p ~/my-cluster
cd ~/my-cluster
ceph-deploy new worker1
```

配置集群总配文件

```
[root@worker1 my-cluster]# pwd
/root/my-cluster
[root@worker1 my-cluster]# ls -al
drwxr-xr-x.  2 root root  4096 5月  10 10:20 .
dr-xr-x---. 26 root root  4096 5月  10 10:25 ..
-rw-r--r--.  1 root root   193 5月   9 22:06 bootstrap-monmap.bin
-rw-------.  1 root root   113 5月  10 10:20 ceph.bootstrap-mds.keyring
-rw-------.  1 root root    71 5月  10 10:20 ceph.bootstrap-mgr.keyring
-rw-------.  1 root root   113 5月  10 10:20 ceph.bootstrap-osd.keyring
-rw-------.  1 root root   113 5月  10 10:20 ceph.bootstrap-rgw.keyring
-rw-------.  1 root root   129 5月  10 10:20 ceph.client.admin.keyring
-rw-r--r--.  1 root root   324 5月  10 10:15 ceph.conf
-rw-r--r--.  1 root root 70384 5月  10 10:22 ceph-deploy-ceph.log
-rw-------.  1 root root    73 5月  10 10:13 ceph.mon.keyring
-rw-------.  1 root root   214 5月   9 22:04 cluster.bootstrap.keyring

[root@worker1 my-cluster]# cat ceph.conf
[global]
fsid = 90b13213-4dc4-4ac4-a99f-2bad9508088b
mon_initial_members = worker1
mon_host = 128.128.98.59

#貌似至少需要启动3个osd
osd pool default size = 3
osd max object name len = 256
osd max object namespace len = 64

public network =128.128.98.59/24

auth_cluster_required = cephx
auth_service_required = cephx
auth_client_required = cephx
```

初始化mon

```
ceph-deploy mon create-initial
```

创建OSD

```
rm -rf  /var/local/osd0
rm -rf  /var/local/osd1
rm -rf  /var/local/osd2

mkdir -p  /var/local/osd0
mkdir -p  /var/local/osd1
mkdir -p  /var/local/osd2

chmod 777 /var/local/osd0
chmod 777 /var/local/osd1
chmod 777 /var/local/osd2

#创建osd
ceph-deploy osd prepare worker1:/var/local/osd0
ceph-deploy osd prepare worker1:/var/local/osd1
ceph-deploy osd prepare worker1:/var/local/osd2

#激活osd
ceph-deploy osd activate worker1:/var/local/osd0
ceph-deploy osd activate worker1:/var/local/osd1
ceph-deploy osd activate worker1:/var/local/osd2
```

创建元数据服务器

```
ceph-deploy mds create worker1
```

添加 RGW 

```
ceph-deploy rgw create worker1
#RGW默认会监听7480端口
```

添加 MONITORS

```
ceph-deploy mon add worker1
```

## 文件地址

配置文件

```
/root/my-cluster/ceph.conf
#最终以这个为准
/etc/ceph/{cluster}.conf
#cephdeploy配置
/root/.cephdeploy.conf
```

日志文件

```
#日志地址
/var/log/ceph/
```

## 搭建命令

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

mkdir -p  /var/local/osd0
chmod 777 /var/local/osd0

ceph-deploy osd prepare node2:/var/local/osd0 node3:/var/local/osd1 node4:/var/local/osd2
ceph-deploy osd activate node2:/var/local/osd0 node3:/var/local/osd1 node4:/var/local/osd2
ceph-deploy admin node1 node2 node3 node4

ceph-deploy mds create node2
ceph-deploy rgw create node3
#然后看rgw的端口号

ceph-deploy --overwrite-conf mon add node3

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

## 运维命令

```bash
#[radosgw-admin命令集] 
#http://docs.ceph.com/docs/giant/man/8/radosgw-admin/

#list bucket
radosgw-admin bucket list
#create user
radosgw-admin user create --display-name="mengyu" --uid=mengyu
#user info
radosgw-admin user info --uid=mengyu
#create sub user
radosgw-admin subuser create --uid=mengyu --subuser=mengyu:mengyu02 --access=full
#remove sub user
radosgw-admin subuser rm --uid=mengyu:mengyu02 [--purge-keys]
#modify user info
radosgw-admin user modify --uid=mengyu --display-name="jiang"
radosgw-admin subuser modify --uid=mengyu:mengyu02 --access=full
#ENABLE/SUSPEND USER
radosgw-admin user suspend --uid=mengyu
radosgw-admin user enable --uid=mengyu
#remove user
radosgw-admin user rm --uid=mengyu
#remove user and clear their bucket
radosgw-admin user rm --uid=mengyu --purge-data
#remove a bucket
radosgw-admin bucket rm --purge-objects --bucket=foo
#radosgw-admin bucket unlink --bucket=foo
#show usage information for user
radosgw-admin usage show --uid=mengyu --start-date=2018-03-12 --end-date=2018-09-14
#show only summary of usage information for all users
radosgw-admin usage show --show-log-entries=false

#[ceph命令集]
#http://docs.ceph.com/docs/master/man/8/ceph/
#add a bucket
ceph osd crush add-bucket <name> <type>
ceph osd crush add-bucket emas-bucket rack
#view the CRUSH hierarchy for your cluster
ceph osd crush tree

#Link bucket to specified user
radosgw-admin bucket link --bucket=default --bucket_id="-1" --uid=mengyu

#pool管理
#创建pool
ceph osd pool create {pool-name} 64 64
ceph osd pool create emas 64 64
#show rados pools
rados lspools
#删除pool
ceph osd pool delete {pool-name} [{pool-name} --yes-i-really-really-mean-it]
ceph osd pool delete data data --yes-i-really-really-mean-it
#show objects in pool
rados -p {pool-name} ls

#读写测试
rados lspools
ceph osd pool create emas 64 64
echo test-data > testfile.txt
#写入数据
rados put {object-name} {file-path} --pool={pool-name}
rados put test-object testfile.txt --pool=emas
读取数据
rados get {object-name} {file-path} --pool={pool-name}
rados get test-object 'out.txt' --pool=emas
rados -p emas ls
ceph osd map emas test-object
#删除对象
rados rm test-object --pool=emas

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
ceph --cluster=ceph osd stat --format=json-pretty
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
ceph quorum_status --format json-pretty
#检查MDS状态
ceph mds stat
ceph mds dump
#获取归置组信息
ceph pg dump
#找出故障归置组
ceph pg dump_stuck [unclean|inactive|stale|undersized|degraded]
#查看mon状态 仲裁结果
ceph daemon mon.node1 mon_status

ceph --cluster=ceph --admin-daemon /var/run/ceph/ceph-mon.worker1.asok mon_status
ceph --cluster=ceph --admin-daemon /var/run/ceph/ceph-mon.worker1.asok mon_status

#列出节点上的磁盘
ceph-deploy disk list node1

#systemctl命令
systemctl enable ceph.target
systemctl enable ceph-mon@worker1
systemctl start ceph-mon@worker1

#更多集群管理操作命令
http://docs.ceph.com/docs/master/rados/operations/operating/
```

## 问题汇总

1. 安装基础依赖包时，要认准对应Linux发行版的包，常见的Linux发行版：el6, el7, arch, debian, fedora.

   - 切记：不要把`arch`的包装到`el6`/`el7`发型版上，否则会引入很多莫名其妙的依赖；
   - 切记：当发现依赖怎么解也解不完时，有可能依赖的上游rpm包是错误的；

2. 这两个包是环境相关的包，先把这两个包安装好：

```bash
rpm -ivh userspace-rcu-0.7.16-1.el7.x86_64.rpm
rpm -ivh lttng-ust-2.4.1-4.el7.x86_64.rpm
```

3. 如果有多个网卡，可以把 `public network` 写入 Ceph 配置文件的 `[global]` 段下。

```
public network = {ip-address}/{netmask}
#public network =128.128.98.59/24
```
4. Could not find keyring file: /etc/ceph/ceph.client.admin.keyring on host

   在执行`ceph-deploy new {hostname}`时，会在该执行目录`~/my-cluster`下生成一堆keyring文件，把这些文件拷贝到`/etc/ceph/`下面即可。

5. health HEALTH_WARN 64 pgs incomplete; 64 pgs stuck inactive; 64 pgs stuck unclean

   导致这个问题可能有两个原因：

   - `~/my-cluster/ceph.conf`中的`osd pool default size`和实际启动的osd数量不一致，比如这个值为3实际只启动了一个osd；
   - 启动osd时，osd目录`/var/local/osd0`下已经有脏数据。每次重新创建osd时，请确保该osd目录下是干净的。

## 参考文档

- Ceph中文文档：<http://docs.ceph.org.cn/start/>
- Ceph离线安装：<https://ivanzz1001.github.io/records/post/ceph/2017/07/14/ceph-install>
- docker安装：<https://github.com/ceph/ceph-container>
- Ceph运维经验：<http://blog.chenmiao.cf/2017/08/15/ceph%E7%BB%B4%E6%8A%A4%E7%BB%8F%E9%AA%8C%E6%80%BB%E7%BB%93?
---
title: Docker安装ceph luminous
author: starifly
date: 2021-10-06T16:16:07+08:00
lastmod: 2021-10-06T16:16:07+08:00
categories: [ceph]
tags: [ceph,docker,luminous]
draft: false
slug: docker-install-ceph-luminous
---

本文基于centos7以docker方式安装ceph 12.2.13版本


## 操作系统基础配置

1. 三节点创建文件夹：

```bash
mkdir -p  /etc/ceph/  /var/lib/ceph/ /var/log/ceph/
chown -R 167:167 /var/log/ceph/
```

2. 配置定时任务

```bash
systemctl start ntpd && systemctl enable ntpd
# 将时间每隔1小时自动校准同步
0 */1 * * * ntpdate ntp1.aliyun.com > /dev/null 2>&1; /sbin/hwclock -w
```

3. 内核优化

```bash
#调整内核参数
[root@CENTOS7-1 ~]# cat >> /etc/sysctl.conf << EOF
> kernel.pid_max=4194303
> vm.swappiness = 0
> EOF
[root@CENTOS7-1 ~]# sysctl -p
# read_ahead, 通过数据预读并且记载到随机访问内存方式提高磁盘读操作，8192是比较理想的值
[root@CENTOS7-1 ~]# echo "8192" > /sys/block/sda/queue/read_ahead_kb 
# I/O Scheduler优化，如果SSD要用noop，SATA/SAS设备采用deadline。
[root@CENTOS7-1 ~]#echo "deadline" > /sys/block/sda/queue/scheduler
[root@CENTOS7-1 ~]#echo "noop" > /sys/block/sda/queue/scheduler
```

4. 关闭selinux

```bash
# vi /etc/selinux/config文件， 将SELINUX设为disabled, 永久生效。
SELINUX=disabled

# 临时生效：
setenforce 0
```

5. 编辑别名

```bash
echo 'alias ceph="docker exec mon ceph"' >> /etc/profile
echo 'alias ceph-fuse="docker exec mon ceph-fuse"' >> /etc/profile
echo 'alias ceph-mon="docker exec mon ceph-mon"' >> /etc/profile
echo 'alias ceph-osd="docker exec mon ceph-osd"' >> /etc/profile
echo 'alias radosgw="docker exec mon radosgw"' >> /etc/profile
echo 'alias radosgw-admin="docker exec mon radosgw-admin"' >> /etc/profile
echo 'alias rados="docker exec mon rados"' >> /etc/profile
source /etc/profile
```

## 启动mon

主节点启动mon：

```bash
docker run -d --net=host     --name=mon1     --restart=always -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     -e MON_IP=192.168.5.203     -e CEPH_PUBLIC_NETWORK=192.168.5.0/24     ceph/daemon:latest-luminous  mon
```

复制配置到另外两个节点：

```bash
scp -r /etc/ceph/ root@192.168.5.204:/etc/
scp -r /etc/ceph/ root@192.168.5.205:/etc/
scp -r /var/lib/ceph/bootstrap-* root@192.168.5.204:/var/lib/ceph/
scp -r /var/lib/ceph/bootstrap-* root@192.168.5.205:/var/lib/ceph/
```

启动成功之后生成配置数据，在ceph主配置文件中，追加如下内容：

203节点

```bash
cat >>/etc/ceph/ceph.conf <<EOF
# 容忍更多的时钟误差
mon clock drift allowed = 2
mon clock drift warn backoff = 30
# 允许删除pool
mon_allow_pool_delete = true
 
[mgr]
# 开启WEB仪表盘
mgr modules = dashboard
[client.rgw.ceph1]
# 设置rgw网关的web访问端口
rgw_frontends = "civetweb port=20003"
EOF
```

204节点：

```bash
cat >>/etc/ceph/ceph.conf <<EOF
# 容忍更多的时钟误差
mon clock drift allowed = 2
mon clock drift warn backoff = 30
# 允许删除pool
mon_allow_pool_delete = true
 
[mgr]
# 开启WEB仪表盘
mgr modules = dashboard
[client.rgw.ceph2]
# 设置rgw网关的web访问端口
rgw_frontends = "civetweb port=20003"
EOF
```

205节点：

```bash
cat >>/etc/ceph/ceph.conf <<EOF
# 容忍更多的时钟误差
mon clock drift allowed = 2
mon clock drift warn backoff = 30
# 允许删除pool
mon_allow_pool_delete = true
 
[mgr]
# 开启WEB仪表盘
mgr modules = dashboard
[client.rgw.ceph3]
# 设置rgw网关的web访问端口
rgw_frontends = "civetweb port=20003"
EOF
```

204节点启动mon：

```bash
docker run -d --net=host     --name=mon2     --restart=always -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     -e MON_IP=192.168.5.204     -e CEPH_PUBLIC_NETWORK=192.168.5.0/24     ceph/daemon:latest-luminous  mon
```

205节点启动mon：

```bash
docker run -d --net=host     --name=mon3     --restart=always -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     -e MON_IP=192.168.5.205     -e CEPH_PUBLIC_NETWORK=192.168.5.0/24     ceph/daemon:latest-luminous  mon
```

## 启动OSD

osd建议以bluestore运行，因为性能高于filestore，这里因为仅做测试，所以使用的是filestore，需要新的硬盘作为OSD存储设备， 关闭虚拟机， 增加一块硬盘(具体增加可参考网上的教程)，然后挂载磁盘。

```bash
mkdir /cephdata
mkfs.xfs -f /dev/sdc1 #设备名字根据实际修改
mount /dev/sdc1  /cephdata
增加开机挂载vim /etc/fstab
/dev/sdc1 /cephdata/                       xfs     defaults        0 0
```

如果没有独立磁盘，怎么办？ 可以在Linux下面创建一个虚拟磁盘进行挂载。

```bash
mkdir -p /usr/local/ceph-disk
dd if=/dev/zero of=/usr/local/ceph-disk/ceph-disk-01 bs=1G count=2
将镜像文件虚拟成块设备：
losetup -f /usr/local/ceph-disk/ceph-disk-01
格式化：
#名称根据fdisk -l进行查询确认, 一般是/dev/loop0 
mkfs.xfs -f /dev/loop0 
挂载文件系统：
mkdir  -p /usr/local/ceph/data/osd/
mount /dev/loop0  /usr/local/ceph/data/osd/
```


三个节点分别执行如下命令启动osd：

```bash
docker run -d     --name=osd1     --net=host     --restart=always     --privileged=true     --pid=host     -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     -v /cephdata:/var/lib/ceph/osd -v /dev/:/dev/ -v /run/udev/:/run/udev/ -e OSD_TYPE=directory    ceph/daemon:latest-luminous  osd
docker run -d     --name=osd2     --net=host     --restart=always     --privileged=true     --pid=host     -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     -v /cephdata:/var/lib/ceph/osd -v /dev/:/dev/ -v /run/udev/:/run/udev/ -e OSD_TYPE=directory    ceph/daemon:latest-luminous  osd
docker run -d     --name=osd3     --net=host     --restart=always     --privileged=true     --pid=host     -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     -v /cephdata:/var/lib/ceph/osd -v /dev/:/dev/ -v /run/udev/:/run/udev/ -e OSD_TYPE=directory    ceph/daemon:latest-luminous  osd
如果直接指定磁盘分区，可以用如下命令启动osd：
docker run -d     --name=osd1     --net=host     --restart=always     --privileged=true     --pid=host     -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     -v /dev/:/dev/ -v /run/udev/:/run/udev/ -e OSD_DEVICE=/dev/sdc    ceph/daemon:latest-luminous  osd
```


## 启动mgr

三个节点执行如下命令启动mgr：

```bash
docker run -d --net=host    --name=mgr1   --restart=always -v /etc/localtime:/etc/localtime:ro   -v /etc/ceph:/etc/ceph   -v /var/lib/ceph:/var/lib/ceph   -v /var/log/ceph:/var/log/ceph   ceph/daemon:latest-luminous mgr
docker run -d --net=host    --name=mgr2   --restart=always -v /etc/localtime:/etc/localtime:ro   -v /etc/ceph:/etc/ceph   -v /var/lib/ceph:/var/lib/ceph   -v /var/log/ceph:/var/log/ceph   ceph/daemon:latest-luminous mgr
docker run -d --net=host    --name=mgr3   --restart=always -v /etc/localtime:/etc/localtime:ro   -v /etc/ceph:/etc/ceph   -v /var/lib/ceph:/var/lib/ceph   -v /var/log/ceph:/var/log/ceph   ceph/daemon:latest-luminous mgr
```

## 启动rgw

三个节点执行如下命令启动rgw：

```bash
docker run     -d --net=host     --name=rgw1   --restart=always  -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     ceph/daemon:latest-luminous rgw
docker run     -d --net=host     --name=rgw2   --restart=always  -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     ceph/daemon:latest-luminous rgw
docker run     -d --net=host     --name=rgw3   --restart=always  -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     ceph/daemon:latest-luminous rgw
```

## 启动mds

MDS 只是为 CephFS 的使用者提供。目前 MDS 多实例仍然处于试验阶段（不提供商业支持），而且文件存储的性能要低于对象存储或者块存储。实际部署参考如下命令。

三个节点执行如下命令启动mds，注意mds名称不能以数字开头：

```bash
docker run -d --net=host --name=mds1 --restart=always -v /etc/localtime:/etc/localtime:ro -v /etc/ceph:/etc/ceph -v /var/lib/ceph:/var/lib/ceph -v /var/log/ceph:/var/log/ceph -e CEPHFS_CREATE=1 -e MDS_NAME=m203 ceph/daemon:latest-luminous mds
docker run -d --net=host --name=mds2 --restart=always -v /etc/localtime:/etc/localtime:ro -v /etc/ceph:/etc/ceph -v /var/lib/ceph:/var/lib/ceph -v /var/log/ceph:/var/log/ceph -e CEPHFS_CREATE=1 -e MDS_NAME=m204 ceph/daemon:latest-luminous mds
docker run -d --net=host --name=mds3 --restart=always -v /etc/localtime:/etc/localtime:ro -v /etc/ceph:/etc/ceph -v /var/lib/ceph:/var/lib/ceph -v /var/log/ceph:/var/log/ceph -e CEPHFS_CREATE=1 -e MDS_NAME=m205 ceph/daemon:latest-luminous mds
```

## 安装Dashboard管理后台

在主节点执行

1. 开启dashboard功能

`docker exec mgr1 ceph mgr module enable dashboard`

2. 配置外部访问IP

`docker exec mgr1 ceph config-key put mgr/dashboard/server_addr 192.168.5.203`

3. 配置外部访问端口

`docker exec mgr1 ceph config-key put mgr/dashboard/server_port 18080`

4. 重启各个节点上的Mgr DashBoard服务，主节点最后重启

`docker restart mgr`

5. 查看Mgr DashBoard服务信息

```bash
[root@CENTOS7-1 admin]# docker exec mgr1 ceph mgr services
{
   "dashboard": "http://0.0.0.0:18080/"
}	
```

## Reference

- [用Docker搭建Ceph集群(luminous版本)](https://feige.blog.csdn.net/article/details/109159160)
- [Centos7系统Docker Ceph 集群的安装配置（中篇）](https://blog.csdn.net/hxx688/article/details/103440967)
- [Centos7系统下Ceph分布式存储集群的部署搭建配置（上篇）](https://blog.csdn.net/hxx688/article/details/103378485)
- [使用Docker快速部署Ceph集群](https://blog.51cto.com/john88wang/1947672)
- [ceph13.2使用docker部署](https://www.cnblogs.com/wukc/p/13501150.html)
- [关于ceph的一些问题及解决](https://blog.csdn.net/weixin_34345560/article/details/92348845)
- [Ceph 手把手教你部署ceph集群](https://blog.csdn.net/qq_34556414/article/details/116091036)
- [dockerhub](https://registry.hub.docker.com/r/ceph/daemon)
- [ceph搭建及使用详解](https://blog.csdn.net/weixin_40548182/article/details/108862031)
- [Ceph架构简介及使用](https://zhuanlan.zhihu.com/p/163537273)

---
title: Docker安装ceph nautilus
author: starifly
date: 2021-10-06T16:54:47+08:00
lastmod: 2021-10-06T16:54:47+08:00
categories: [ceph]
tags: [ceph,nautilus,bluestore]
draft: false
slug: docker-install-ceph-nautilus
---

## 操作系统基础配置

1. 三节点创建文件夹：

`mkdir -p /etc/ceph /var/lib/ceph /var/log/ceph`

2. 配置定时任务

```bash
systemctl start ntpd && systemctl enable ntpd
将时间每隔1小时自动校准同步
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

5. 修改主机名和hosts

```bash
hostnamectl set-hostname ceph001
hostnamectl set-hostname ceph002
hostnamectl set-hostname ceph003

# vim /etc/hosts
192.168.5.203 ceph001
192.168.5.203 ceph002
192.168.5.203 ceph003
```

6. 编辑别名

```bash
echo 'alias ceph="docker exec mon ceph"' >> /etc/profile
source /etc/profile
```

7. 准备磁盘

三节点各准备一块磁盘，不用分区。

以下安装配置基于centos7.8、ceph 14.2.22版本

## 启动mon

主节点：

```bash
docker run -d --net=host     --name=mon  --restart=always   -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     -e MON_IP=192.168.5.203     -e CEPH_PUBLIC_NETWORK=192.168.5.0/24     ceph/daemon:v4.0.22-stable-4.0-nautilus-centos-7-x86_64  mon
```

启动成功之后生成配置数据，在ceph主配置文件中，修改内容：

```bash
mon host = 192.168.5.203,192.168.5.204,192.168.5.205

cat >>/etc/ceph/ceph.conf <<EOF
# 容忍更多的时钟误差
mon clock drift allowed = 2
mon clock drift warn backoff = 30
# 允许删除pool
mon_allow_pool_delete = true
# rbd 配置
rbd_default_features = 3
# 关闭动态分片，在zone中配置
rgw dynamic resharding = false
 
[mgr]
# 开启WEB仪表盘
mgr modules = dashboard
[client.rgw.ceph001]
# 设置rgw网关的web访问端口
rgw_frontends = "civetweb port=20003 num_threads=2048"
EOF
```

复制配置到另外两个节点：

```bash
scp -r /etc/ceph/ root@192.168.5.204:/etc
scp -r /etc/ceph/ root@192.168.5.205:/etc
scp -r /var/lib/ceph/bootstrap-* root@192.168.5.204:/var/lib/ceph/
scp -r /var/lib/ceph/bootstrap-* root@192.168.5.205:/var/lib/ceph/
```

修改节点2和3配置文件：

```bash
client.rgw.ceph001 -> client.rgw.ceph002
client.rgw.ceph001 -> client.rgw.ceph003
```

204节点：

```bash
docker run -d --net=host     --name=mon  --restart=always  -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     -e MON_IP=192.168.5.204     -e CEPH_PUBLIC_NETWORK=192.168.5.0/24     ceph/daemon:v4.0.22-stable-4.0-nautilus-centos-7-x86_64  mon
```

205节点：

```bash
docker run -d --net=host     --name=mon  --restart=always  -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     -e MON_IP=192.168.5.205     -e CEPH_PUBLIC_NETWORK=192.168.5.0/24     ceph/daemon:v4.0.22-stable-4.0-nautilus-centos-7-x86_64  mon
```

## 启动OSD

首先需要在mon节点生成osd的密钥信息，不然直接启动会报错。命令如下：

```bash
docker exec -it mon ceph auth get client.bootstrap-osd -o /var/lib/ceph/bootstrap-osd/ceph.keyring

# 复制密钥到另外两个节点：
scp -r /var/lib/ceph/bootstrap-osd/ceph.keyring 192.168.5.204:/var/lib/ceph/bootstrap-osd/
scp -r /var/lib/ceph/bootstrap-osd/ceph.keyring 192.168.5.205:/var/lib/ceph/bootstrap-osd/
```

osd采用bluestore，所以磁盘事先不用分区和挂载文件系统。bluestore比filestore的效率更高，推荐采用。

三个节点执行如下命令启动osd：

如有必要先格式化磁盘：

```bash
docker run --privileged=true   --rm  --pid=host     -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     -v /dev/:/dev/ -v /run/udev/:/run/udev/ --entrypoint /bin/bash ceph/daemon:v4.0.22-stable-4.0-nautilus-centos-7-x86_64  -c 'ceph-volume lvm zap /dev/sdb'
```

ceph集群无法初始化osd问题,有时候会遇到这样的报错。

```bash
[node3-ceph][WARNIN] stderr: wipefs: error: /dev/sdb: probing initialization failed: Device or resource busy [node3-ceph][WARNIN] --> failed to wipefs device, will try again to workaround probable race condition
```

遇到这种报错时，只能上这台机器，手动进行dd命令清空磁盘并重启

```bash
dd if=/dev/zero of=/dev/sdb bs=512K count=1
reboot
```

重启完成后，就能够清理磁盘成功了。

部署bluestore并激活：

203节点：

```bash
docker run --privileged=true   --rm  --pid=host     -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     -v /dev/:/dev/ -v /run/udev/:/run/udev/ --entrypoint /bin/bash ceph/daemon:v4.0.22-stable-4.0-nautilus-centos-7-x86_64  -c 'ceph-volume lvm prepare --bluestore --data /dev/sdb'
docker run -d     --name=osd     --net=host     --restart=always     --privileged=true     --pid=host     -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     -v /dev/:/dev/ -v /run/udev/:/run/udev/ -e OSD_ID=0 ceph/daemon:v4.0.22-stable-4.0-nautilus-centos-7-x86_64  osd_ceph_volume_activate
```

204节点：

```bash
docker run --privileged=true   --rm  --pid=host     -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     -v /dev/:/dev/ -v /run/udev/:/run/udev/ --entrypoint /bin/bash ceph/daemon:v4.0.22-stable-4.0-nautilus-centos-7-x86_64  -c 'ceph-volume lvm prepare --bluestore --data /dev/sdb'
docker run -d     --name=osd     --net=host     --restart=always     --privileged=true     --pid=host     -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     -v /dev/:/dev/ -v /run/udev/:/run/udev/ -e OSD_ID=1 ceph/daemon:v4.0.22-stable-4.0-nautilus-centos-7-x86_64  osd_ceph_volume_activate
```

205节点：

```bash
docker run --privileged=true   --rm  --pid=host     -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     -v /dev/:/dev/ -v /run/udev/:/run/udev/ --entrypoint /bin/bash ceph/daemon:v4.0.22-stable-4.0-nautilus-centos-7-x86_64  -c 'ceph-volume lvm prepare --bluestore --data /dev/sdb'
docker run -d     --name=osd     --net=host     --restart=always     --privileged=true     --pid=host     -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     -v /dev/:/dev/ -v /run/udev/:/run/udev/ -e OSD_ID=2 ceph/daemon:v4.0.22-stable-4.0-nautilus-centos-7-x86_64  osd_ceph_volume_activate
```

## 启动mgr

三个节点执行如下命令启动mgr：

```bash
docker run -d --net=host    --name=mgr --restart=always  -v /etc/localtime:/etc/localtime:ro   -v /etc/ceph:/etc/ceph   -v /var/lib/ceph:/var/lib/ceph   -v /var/log/ceph:/var/log/ceph   ceph/daemon:v4.0.22-stable-4.0-nautilus-centos-7-x86_64 mgr
```

## 启动rgw

首先需要在mon节点生成rgw的密钥信息，不然直接启动会报错。命令如下：

```bash
docker exec -it mon ceph auth get client.bootstrap-rgw -o /var/lib/ceph/bootstrap-rgw/ceph.keyring

# 复制密钥到另外两个节点：
scp -r /var/lib/ceph/bootstrap-rgw/ceph.keyring 192.168.5.204:/var/lib/ceph/bootstrap-rgw/
scp -r /var/lib/ceph/bootstrap-rgw/ceph.keyring 192.168.5.205:/var/lib/ceph/bootstrap-rgw/
```

三个节点执行如下命令启动rgw：

```bash
docker run     -d --net=host     --name=rgw  --restart=always   -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     ceph/daemon:v4.0.22-stable-4.0-nautilus-centos-7-x86_64 rgw
```

## 启动mds

三个节点执行如下命令启动mds，注意mds名称不能以数字开头：

```bash
docker run -d --net=host --name=mds --restart=always -v /etc/localtime:/etc/localtime:ro -v /etc/ceph:/etc/ceph -v /var/lib/ceph:/var/lib/ceph -v /var/log/ceph:/var/log/ceph -e CEPHFS_CREATE=1 -e MDS_NAME=ceoh001 ceph/daemon:v4.0.22-stable-4.0-nautilus-centos-7-x86_64 mds

docker run -d --net=host --name=mds --restart=always -v /etc/localtime:/etc/localtime:ro -v /etc/ceph:/etc/ceph -v /var/lib/ceph:/var/lib/ceph -v /var/log/ceph:/var/log/ceph -e CEPHFS_CREATE=1 -e MDS_NAME=ceoh002 ceph/daemon:v4.0.22-stable-4.0-nautilus-centos-7-x86_64 mds

docker run -d --net=host --name=mds --restart=always -v /etc/localtime:/etc/localtime:ro -v /etc/ceph:/etc/ceph -v /var/lib/ceph:/var/lib/ceph -v /var/log/ceph:/var/log/ceph -e CEPHFS_CREATE=1 -e MDS_NAME=ceoh003 ceph/daemon:v4.0.22-stable-4.0-nautilus-centos-7-x86_64 mds
```

## 安装Dashboard管理后台

在主节点执行

1. 开启dashboard功能

`docker exec mgr ceph mgr module enable dashboard`

2. 创建证书

`docker exec mgr ceph dashboard create-self-signed-cert`

3. 创建登陆用户与密码：

`docker exec mgr ceph dashboard set-login-credentials admin -i /root/passwd.txt`

4. 配置外部访问IP

`docker exec mgr ceph config set mgr mgr/dashboard/server_addr 192.168.5.203`

5. 重启各个节点上的Mgr DashBoard服务，主节点最后重启

`docker restart mgr`

6. 查看Mgr DashBoard服务信息

```bash
[root@CENTOS7-1 admin]# docker exec mgr ceph mgr services
{
   "dashboard": "http://192.168.5.203:18080/"
}	
```

## Reference

- [用Docker搭建Ceph集群(octopus版本)](https://feige.blog.csdn.net/article/details/109159160)
- [Centos7系统Docker Ceph 集群的安装配置（中篇）](https://blog.csdn.net/hxx688/article/details/103440967)
- [Centos7系统下Ceph分布式存储集群的部署搭建配置（上篇）](https://blog.csdn.net/hxx688/article/details/103378485)
- [使用Docker快速部署Ceph集群](https://blog.51cto.com/john88wang/1947672)
- [ceph13.2使用docker部署](https://www.cnblogs.com/wukc/p/13501150.html)
- [关于ceph的一些问题及解决](https://blog.csdn.net/weixin_34345560/article/details/92348845)
- [Ceph 手把手教你部署ceph集群](https://blog.csdn.net/qq_34556414/article/details/116091036)
- [dockerhub](https://registry.hub.docker.com/r/ceph/daemon)
- [ceph搭建及使用详解](https://blog.csdn.net/weixin_40548182/article/details/108862031)
- [Ceph架构简介及使用](https://zhuanlan.zhihu.com/p/163537273)
- [ceph-volume部署filestore或bluestore](https://blog.csdn.net/wly_sky/article/details/80965605)
- [Update Documentation: OSD_TYPE=disk does not work in nautilus #1406](https://github.com/ceph/ceph-container/issues/1406)

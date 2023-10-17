---
title: ceph三节点故障恢复
author: starifly
date: 2021-10-06T19:11:07+08:00
lastmod: 2021-10-06T19:11:07+08:00
categories: [ceph]
tags: [ceph]
draft: false
slug: ceph-three-node-failure-recovery
---

ceph 集群有三个节点，每台节点都用 docker 容器部署了 mon、osd、mgr、rgw、mds 服务，现在假设在其它机器备份了ceph集群的配置和认证信息（/etc/ceph和/var/lib/ceph目录），而当前三个节点全部出现系统故障导致集群瘫痪的情况下，我们怎么恢复 ceph 集群。

## 恢复 mon 节点

当三个节点系统恢复正常后，首先我们要把备用配置拷贝至三节点对应目录，然后开始恢复三个 mon 节点。

主节点：

```bash
docker run -d --net=host     --name=mon  --restart=always   -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     -e MON_IP=192.168.5.203     -e CEPH_PUBLIC_NETWORK=192.168.5.0/24     ceph/daemon:v4.0.20-stable-4.0-nautilus-centos-7-x86_64  mon
```

204节点：

```bash
docker run -d --net=host     --name=mon  --restart=always  -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     -e MON_IP=192.168.5.204     -e CEPH_PUBLIC_NETWORK=192.168.5.0/24     ceph/daemon:v4.0.20-stable-4.0-nautilus-centos-7-x86_64  mon
```

205节点：

```bash
docker run -d --net=host     --name=mon  --restart=always  -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     -e MON_IP=192.168.5.205     -e CEPH_PUBLIC_NETWORK=192.168.5.0/24     ceph/daemon:v4.0.20-stable-4.0-nautilus-centos-7-x86_64  mon
```

## 恢复 osd 节点

当恢复了 mon 节点后，可以用命令 `ceph -s` 查询集群状态信息，发现 osd 还在集群中，所以首先要剔除 osd。

```bash
# ceph -s
……
  services:
    mon: 3 daemons, quorum ceph001,ceph002 (age 15s), out of quorum: ceph003
    mgr: ceph001(active, since 29h), standbys: ceph003
    mds: cephfs:1 {0=ceoh001=up:active} 2 up:standby
    osd: 3 osds: 3 up (since 136y), 3 in (since 34h)
    rgw: 3 daemons active (ceph001, ceph002, ceph003)
……
```

剔除 osd

```bash
ceph osd crush rm osd.0 && ceph osd down 0 && ceph osd rm 0
ceph osd crush rm osd.1 && ceph osd down 1 && ceph osd rm 1
ceph osd crush rm osd.2 && ceph osd down 2 && ceph osd rm 2
```

重新部署恢复 osd

```bash
# 只要激活即可
# 203节点
docker run -d     --name=osd     --net=host     --restart=always     --privileged=true     --pid=host     -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     -v /dev/:/dev/ -v /run/udev/:/run/udev/ -e OSD_ID=0 ceph/daemon:v4.0.20-stable-4.0-nautilus-centos-7-x86_64  osd_ceph_volume_activate

# 204节点
docker run -d     --name=osd     --net=host     --restart=always     --privileged=true     --pid=host     -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     -v /dev/:/dev/ -v /run/udev/:/run/udev/ -e OSD_ID=1 ceph/daemon:v4.0.20-stable-4.0-nautilus-centos-7-x86_64  osd_ceph_volume_activate

# 205节点
docker run -d     --name=osd     --net=host     --restart=always     --privileged=true     --pid=host     -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     -v /dev/:/dev/ -v /run/udev/:/run/udev/ -e OSD_ID=2 ceph/daemon:v4.0.20-stable-4.0-nautilus-centos-7-x86_64  osd_ceph_volume_activate
```

通过 `ceph -s` 命令观察集群状态，当 osd 和 pgs 都恢复正常后，即可进行下一步

```bash
[root@ceph001 ~]# ceph -s
  cluster:
    id:     d00c744a-17f6-4768-95de-1243202557f2
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum ceph001,ceph002,ceph003 (age 8m)
    mgr: ceph001(active, since 3m), standbys: ceph003, ceph002
    mds: cephfs:1 {0=ceoh001=up:active(laggy or crashed)} 2 up:standby
    osd: 3 osds: 3 up (since 4m), 3 in (since 34h)
 
  data:
    pools:   7 pools, 208 pgs
    objects: 262 objects, 14 MiB
    usage:   3.1 GiB used, 57 GiB / 60 GiB avail
    pgs:     208 active+clean
```

## 恢复 mgr 节点

三节点执行如下命令恢复：

```bash
docker run -d --net=host    --name=mgr --restart=always  -v /etc/localtime:/etc/localtime:ro   -v /etc/ceph:/etc/ceph   -v /var/lib/ceph:/var/lib/ceph   -v /var/log/ceph:/var/log/ceph   ceph/daemon:v4.0.20-stable-4.0-nautilus-centos-7-x86_64 mgr
```

## 恢复 rgw 节点

三节点执行如下命令恢复：

```bash
docker run     -d --net=host     --name=rgw  --restart=always   -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     ceph/daemon:v4.0.20-stable-4.0-nautilus-centos-7-x86_64 rgw
```

## 恢复 mds 节点

```bash
# 203节点
docker run -d --net=host --name=mds --restart=always -v /etc/localtime:/etc/localtime:ro -v /etc/ceph:/etc/ceph -v /var/lib/ceph:/var/lib/ceph -v /var/log/ceph:/var/log/ceph -e CEPHFS_CREATE=1 -e MDS_NAME=ceoh001 ceph/daemon:v4.0.20-stable-4.0-nautilus-centos-7-x86_64 mds

# 204节点
docker run -d --net=host --name=mds --restart=always -v /etc/localtime:/etc/localtime:ro -v /etc/ceph:/etc/ceph -v /var/lib/ceph:/var/lib/ceph -v /var/log/ceph:/var/log/ceph -e CEPHFS_CREATE=1 -e MDS_NAME=ceoh002 ceph/daemon:v4.0.20-stable-4.0-nautilus-centos-7-x86_64 mds

# 205节点
docker run -d --net=host --name=mds --restart=always -v /etc/localtime:/etc/localtime:ro -v /etc/ceph:/etc/ceph -v /var/lib/ceph:/var/lib/ceph -v /var/log/ceph:/var/log/ceph -e CEPHFS_CREATE=1 -e MDS_NAME=ceoh003 ceph/daemon:v4.0.20-stable-4.0-nautilus-centos-7-x86_64 mds
```

## Reference

- [CEPH集群MON全部挂掉后恢复方法](https://blog.csdn.net/xuensong520/article/details/78533209)

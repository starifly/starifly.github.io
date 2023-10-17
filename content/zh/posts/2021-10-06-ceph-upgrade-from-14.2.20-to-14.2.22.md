---
title: ceph从14.2.20升级到14.2.22
author: starifly
date: 2021-10-06T19:38:23+08:00
lastmod: 2021-10-06T19:38:23+08:00
categories: [ceph]
tags: [ceph]
draft: false
slug: ceph-upgrade-from-14.2.20-to-14.2
---

注意：此文档用于 Ceph Nautilus 版本（包括社区版 Ceph 14.2.x 和红帽版 Redhat Ceph Storage 4.x）内的小版本升级，不能用于 Ceph 大版本升级（例如从 Ceph Luminous 升级到 Ceph Nautilus）。

## Ceph 集群简介

Ceph Nautilus 集群（docker方式部署）包括的角色如下：

- mon：monitor 节点，用于集群选主。节点数量：3。
- mgr：manager 节点，用于集群管理，包括监控，告警等。节点数量：3。
- osd：存储节点，用于存放 ceph 集群所有数据。节点数量：3。
- mds：cephfs 元数据节点，用于管理 cephfs 文件系统元数据。节点数量：3。
- rgw：对象存储网关，用于对外提供 AWS S3 API 接口。节点数量：3。

## 升级步骤

根据官网升级文档，总结升级顺序如下：

升级 mon 节点 ——> 升级 mgr 节点 ——> 升级 osd 节点 ——> 升级 mds 节点 ——> 升级 rgw 节点

升级过程中每执行一步即时用 `ceph -s` 和 `ceph versions` 查看集群状态，确认状态到达期望值再进行下一步。

## 升级前准备

1.设置集群 osd 状态为 noout，nodeep-scrub

```bash
ceph osd set noout
ceph osd set nodeep-scrub
```

## 升级 mon 节点

203节点：

```bash
docker rm -f mon
docker run -d --net=host     --name=mon  --restart=always   -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     -e MON_IP=192.168.5.203     -e CEPH_PUBLIC_NETWORK=192.168.5.0/24     ceph/daemon:v4.0.22-stable-4.0-nautilus-centos-7-x86_64  mon
```

204节点：

```bash
docker rm -f mon
docker run -d --net=host     --name=mon  --restart=always  -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     -e MON_IP=192.168.5.204     -e CEPH_PUBLIC_NETWORK=192.168.5.0/24     ceph/daemon:v4.0.22-stable-4.0-nautilus-centos-7-x86_64  mon
```

205节点：

```bash
docker rm -f mon
docker run -d --net=host     --name=mon  --restart=always  -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     -e MON_IP=192.168.5.205     -e CEPH_PUBLIC_NETWORK=192.168.5.0/24     ceph/daemon:v4.0.22-stable-4.0-nautilus-centos-7-x86_64  mon
```

## 升级 mgr 节点

所有节点执行：

```bash
docker rm -f mgr
docker run -d --net=host    --name=mgr --restart=always  -v /etc/localtime:/etc/localtime:ro   -v /etc/ceph:/etc/ceph   -v /var/lib/ceph:/var/lib/ceph   -v /var/log/ceph:/var/log/ceph   ceph/daemon:v4.0.22-stable-4.0-nautilus-centos-7-x86_64 mgr
```

确认 mgr 功能正常

```bash
ceph -s 

...
  services:
    mon: 3 daemons, quorum ceph001,ceph002,ceph003 (age 5s)
    mgr: ceph001(active, since 89s)
...
```

## 升级 osd 节点

203节点：

```bash
docker rm -f osd
docker run -d     --name=osd     --net=host     --restart=always     --privileged=true     --pid=host     -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     -v /dev/:/dev/ -v /run/udev/:/run/udev/ -e OSD_ID=0 ceph/daemon:v4.0.22-stable-4.0-nautilus-centos-7-x86_64  osd_ceph_volume_activate
```

204节点：

```bash
docker rm -f osd
docker run -d     --name=osd     --net=host     --restart=always     --privileged=true     --pid=host     -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     -v /dev/:/dev/ -v /run/udev/:/run/udev/ -e OSD_ID=1 ceph/daemon:v4.0.22-stable-4.0-nautilus-centos-7-x86_64  osd_ceph_volume_activate
```

205节点：

```bash
docker rm -f osd
docker run -d     --name=osd     --net=host     --restart=always     --privileged=true     --pid=host     -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     -v /dev/:/dev/ -v /run/udev/:/run/udev/ -e OSD_ID=2 ceph/daemon:v4.0.22-stable-4.0-nautilus-centos-7-x86_64  osd_ceph_volume_activate
```

可通过 `ceph versions` 观察升级进度

## 升级 mds 节点

将 cephfs 的集群节点数量设置成 1

```bash
ceph status
ceph fs set <fs_name> max_mds 1
```

等待 cephfs 集群只剩一个节点为 active

```bash
ceph status
```

将所有 standby 的 MDS 容器删掉

204、205节点：

```bash
docker rm -f mds
```

确保只有一个 MDS 服务在线，并且为 rank 0

```bash
ceph status
```

升级 active（203节点） 节点的 mds 容器

```bash
docker rm -f mds
docker run -d --net=host --name=mds --restart=always -v /etc/localtime:/etc/localtime:ro -v /etc/ceph:/etc/ceph -v /var/lib/ceph:/var/lib/ceph -v /var/log/ceph:/var/log/ceph -e CEPHFS_CREATE=1 -e MDS_NAME=ceoh001 ceph/daemon:v4.0.22-stable-4.0-nautilus-centos-7-x86_64 mds
```

启动升级 standby 节点的 mds 容器

```bash
#204节点：
docker run -d --net=host --name=mds --restart=always -v /etc/localtime:/etc/localtime:ro -v /etc/ceph:/etc/ceph -v /var/lib/ceph:/var/lib/ceph -v /var/log/ceph:/var/log/ceph -e CEPHFS_CREATE=1 -e MDS_NAME=ceoh002 ceph/daemon:v4.0.22-stable-4.0-nautilus-centos-7-x86_64 mds
#205节点：
docker run -d --net=host --name=mds --restart=always -v /etc/localtime:/etc/localtime:ro -v /etc/ceph:/etc/ceph -v /var/lib/ceph:/var/lib/ceph -v /var/log/ceph:/var/log/ceph -e CEPHFS_CREATE=1 -e MDS_NAME=ceoh003 ceph/daemon:v4.0.22-stable-4.0-nautilus-centos-7-x86_64 mds
```

将 cephfs 集群的 max_mds 数量还原

```bash
ceph fs set <fs_name> max_mds <original_max_mds>
```

## 升级 rgw 节点

所有节点执行：

```bash
docker rm -f rgw
docker run     -d --net=host     --name=rgw  --restart=always   -v /etc/localtime:/etc/localtime:ro     -v /etc/ceph:/etc/ceph     -v /var/lib/ceph:/var/lib/ceph     -v /var/log/ceph:/var/log/ceph     ceph/daemon:v4.0.22-stable-4.0-nautilus-centos-7-x86_64 rgw
```

## 升级后操作

```bash
ceph osd unset noout
ceph osd unset nodeep-scrub
```

https://docs.ceph.com/en/latest/releases/nautilus/#upgrading-from-mimic-or-luminous
Ceph Nautilus 升级方案 - https://blog.csdn.net/zzboat0422/article/details/112787626?spm=1001.2014.3001.5501

---
title: ceph 操作命令
author: starifly
date: 2021-10-06T19:43:27+08:00
lastmod: 2021-10-06T19:43:27+08:00
categories: [ceph]
tags: [ceph]
draft: false
slug: ceph-commands
---

```bash
查看ceph的实时运行状态
ceph -w

查看ceph存储空间
ceph df/ceph df detail

查看集群的详细配置
ceph daemon mon.ceph001 config show | more
or
ceph --admin-daemon /var/run/ceph/ceph-mon.ceph001.asok config show

查看集群版本信息
ceph versions

查看集群认证信息
ceph auth list

查看gc时间
radosgw-admin gc list --include-all | grep time

mon命令
ceph mon stat#查看mon的状态信息

mds命令
ceph mds stat #查看msd状态

osd命令
ceph osd stat #查看osd状态

ceph osd df #查看osd磁盘

ceph osd dump #osd的映射信息

ceph osd perf #查看osd性能和延迟

ceph osd tree#查看osd目录树

rgw命令
radosgw-admin bucket list #查看所有桶

radosgw-admin bucket list --bucket=asdas #查看桶内对象

radosgw-admin bucket stats --bucket=asdas #查看桶信息

radosgw-admin bucket rm --bucket=s3test1 #删除一个桶

radosgw-admin bucket rm --bucket=s3test1 --purge-objects #默认只能删空的bucket，强制删除非空的bucket需要加上“--purge-objects ”选项

radosgw-admin bucket check --bucket=s3test1 #查看桶的索引信息

radosgw-admin object rm --object=1.jpg --bucket=s3test3 #删除桶中的对象

radosgw-admin object unlink --bucket=s3test3 —object=1.jpg #从桶索引里去除对象

radosgw-admin bucket limit check [--uid test] #查看Sharding Stat

radosgw-admin bucket stats --bucket=bucketname|grep -w 'num_objects' #查询桶的对象个数

创建realm：
realm里所有的元數據名稱都是全局唯一的，無法創建同名的用戶（uid）和bucket， container；
radosgw-admin realm create --rgw-realm=Giant --default

查看存在的realm：
radosgw-admin realm list

查看bucket 的信息
radosgw-admin metadata get bucket:bucketname

获取bucket id 并根据 bucket id 从bucket.instace中获取真实shards，可以看到num_shards的值
radosgw-admin  metadata get bucket.instance:api_image_test:4fb0079d-88d8-404e-9e3f-1375ba40a870.324279.2

查看index 下保留的shard数量（包含历史的分片）
rados ls -p default.rgw.buckets.index

查看bucket : bucketname 当前的shard分片,首先找到 当前bucket id
找到bucket id后查询具体的shard编号，如下
rados ls -p default.rgw.buckets.index | grep "555c6dcf-7320-4b34-8922-5e005dedc130.164465.1"

统计当前使用的shard数
rados ls -p default.rgw.buckets.index | grep "555c6dcf-7320-4b34-8922-5e005dedc130.164465.1" | wc -l

查看当前shard 存在的index数据
rados -p default.rgw.buckets.index listomapkeys .dir.555c6dcf-7320-4b34-8922-5e005dedc130.164465.1.769
```

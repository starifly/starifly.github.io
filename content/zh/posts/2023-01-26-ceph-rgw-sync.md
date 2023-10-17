---
title: ceph rgw同步
author: starifly
date: 2023-01-26T18:53:34+08:00
lastmod: 2023-01-26T18:53:34+08:00
categories: [ceph]
tags: [ceph,rgw]
draft: false
slug: ceph-rgw-sync
---

## 环境

- 源 ceph 集群（192.168.5.203:20003）
- 基于源集群创建一个新的 rgw 端点(192.168.5.128:7480)，用于将数据同步到另一个 S3提供者
- S3 目标（192.168.4.13:7480）

## 操作步骤

**准备存储池**

```bash
for pool in sync.rgw.meta sync.rgw.log sync.rgw.control sync.rgw.buckets.non-ec sync.rgw.buckets.index sync.rgw.buckets.data; do ceph osd pool create $pool 16 16 replicated; done
```

**创建新区域**

```bash
radosgw-admin zone create --rgw-zonegroup=default --rgw-zone=sync --endpoints=http://192.168.5.128:7480/ --tier-type=cloud
```

**修改现有区域**

```bash
radosgw-admin zone modify --rgw-zonegroup=default --rgw-zone=default --endpoints=http://192.168.5.203:20003
```

**配置同步区域以使用此系统用户**

我们将更改两个区域以使用我们的新系统用户。

```bash
radosgw-admin zone modify --rgw-zonegroup=default --rgw-zone=default --access-key=GVGIA33TY7G86W1QDYKV --secret=0t1IyppTAdQHfpdrUJh1NfPJPBTF9Qb4weByuK8L

radosgw-admin zone modify --rgw-zonegroup=default --rgw-zone=sync --access-key=GVGIA33TY7G86W1QDYKV --secret=0t1IyppTAdQHfpdrUJh1NfPJPBTF9Qb4weByuK8L
```

**确保默认区域是主区域**

```bash
radosgw-admin zonegroup get，实际上虽然查询default zone是master，但是下面这条语句还是要执行一次。
```

如果defaultzone 不是 master，则可以通过执行强制它radosgw-admin zone modify --rgw-zonegroup=default --rgw-zone=default --master --default

**提交更改并验证配置**

```bash
radosgw-admin period update --commit
```

**配置新区域以同步数据到目的集群**

```bash
radosgw-admin zone modify --rgw-zonegroup=default --rgw-zone=sync --tier-config=connection.endpoint=http://192.168.4.13:7480,connection.access_key=JO4RQ1787A6OGI6XMFDW,connection.secret=Dx5kKGUUeR0DaSRYueBWhV6oDRvJ9oXH2gPcVJ6s，target_path=\$\{bucket\}
```

> 其中的 target_path 代表同步的目的位置，这里我们配置成源 bucket 对应的位置。

**提交更改**

```bash
radosgw-admin zone get --rgw-zone=sync
```

**配置 RGW**

我们须修改源集群 rgw（192.168.5.203） 的配置和 同步 rgw（192.168.5.128） 实例的配置

在 ceph 配置 rgw 段增加如下配置：

```cfg
host = ceph001
rgw zone = default

[client.rgw.vm128]
host = vm128
rgw zone = sync
```

重启 rgw 以使更改生效。

至此我们就可以测试两个集群之间的数据同步了。

## Reference
- [【大咖专栏】如何配置CEPH RGW对象存储与公有云同步](https://blog.csdn.net/NewTyun/article/details/120735475)
- [https://docs.ceph.com/en/latest/radosgw/multisite/](https://docs.ceph.com/en/latest/radosgw/multisite/)
- [https://docs.ceph.com/en/latest/radosgw/cloud-sync-module/#cloud-sync-tier-type-configuration](https://docs.ceph.com/en/latest/radosgw/cloud-sync-module/#cloud-sync-tier-type-configuration)
- [SETTING UP CEPH CLOUD SYNC MODULE](https://croit.io/blog/setting-up-ceph-cloud-sync-module)

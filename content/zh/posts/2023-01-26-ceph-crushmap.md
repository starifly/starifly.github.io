---
title: ceph故障域
author: starifly
date: 2023-01-26T19:01:44+08:00
lastmod: 2023-01-26T19:01:44+08:00
categories: [ceph]
tags: [ceph]
draft: false
slug: ceph-crushmap
---

## 准备

1、执行ceph -s确认存储集群状态，保证为健康状态。

```bash
[root@ceph001 ~]# ceph -s
  cluster:
    id:     d00c744a-17f6-4768-95de-1243202557f2
    health: HEALTH_OK

  services:
    mon: 3 daemons, quorum ceph001,ceph002,ceph003 (age 41m)
    mgr: ceph001(active, since 43m), standbys: ceph003
    mds: cephfs:1 {0=ceoh002=up:active} 2 up:standby
    osd: 6 osds: 6 up (since 3m), 6 in (since 3m)
    rgw: 3 daemons active (ceph001, ceph002, ceph003)

  task status:

  data:
    pools:   10 pools, 96 pgs
    objects: 803 objects, 2.1 GiB
    usage:   13 GiB used, 62 GiB / 75 GiB avail
    pgs:     96 active+clean
```

2、执行ceph osd tree ,记录变更前的结构。以及存储池pool及其他信息。

```bash
[root@ceph001 ~]# ceph osd tree
ID CLASS WEIGHT  TYPE NAME        STATUS REWEIGHT PRI-AFF
-1       0.07315 root default
-3       0.02438     host ceph001
 0   hdd 0.01949         osd.0        up  1.00000 1.00000
 3   hdd 0.00490         osd.3        up  1.00000 1.00000
-5       0.02438     host ceph002
 1   hdd 0.01949         osd.1        up  1.00000 1.00000
 4   hdd 0.00490         osd.4        up  1.00000 1.00000
-7       0.02438     host ceph003
 2   hdd 0.01949         osd.2        up  1.00000 1.00000
 5   hdd 0.00490         osd.5        up  1.00000 1.00000
```

查看存储池pool规则及其他详细信息

```bash
[root@ceph001 ~]# ceph osd pool ls detail
pool 1 '.rgw.root' replicated size 3 min_size 2 crush_rule 0 object_hash rjenkins pg_num 8 pgp_num 8 autoscale_mode warn last_change 1108 lfor 0/1108/1106 flags hashpspool stripe_width 0 application rgw
pool 2 'default.rgw.control' replicated size 3 min_size 2 crush_rule 0 object_hash rjenkins pg_num 8 pgp_num 8 autoscale_mode warn last_change 1472 lfor 0/1472/1470 flags hashpspool stripe_width 0 application rbd
pool 3 'default.rgw.meta' replicated size 3 min_size 2 crush_rule 0 object_hash rjenkins pg_num 8 pgp_num 8 autoscale_mode warn last_change 1691 lfor 0/1691/1689 flags hashpspool stripe_width 0 application rgw
pool 4 'default.rgw.log' replicated size 3 min_size 2 crush_rule 0 object_hash rjenkins pg_num 8 pgp_num 8 autoscale_mode warn last_change 1580 lfor 0/1580/1578 flags hashpspool stripe_width 0 application rgw
pool 5 'cephfs_data' replicated size 3 min_size 2 crush_rule 0 object_hash rjenkins pg_num 8 pgp_num 8 autoscale_mode warn last_change 29 flags hashpspool stripe_width 0 application cephfs
pool 6 'cephfs_metadata' replicated size 3 min_size 2 crush_rule 0 object_hash rjenkins pg_num 8 pgp_num 8 autoscale_mode warn last_change 29 flags hashpspool stripe_width 0 pg_autoscale_bias 4 pg_num_min 16 recovery_priority 5 application cephfs
pool 7 'testpool' replicated size 3 min_size 2 crush_rule 0 object_hash rjenkins pg_num 8 pgp_num 8 autoscale_mode warn last_change 908 lfor 0/908/906 flags hashpspool,selfmanaged_snaps stripe_width 0 application rbd
	removed_snaps [1~3]
pool 11 'default.rgw.buckets.index' replicated size 3 min_size 1 crush_rule 0 object_hash rjenkins pg_num 16 pgp_num 16 autoscale_mode warn last_change 1247 lfor 0/1247/1245 flags hashpspool stripe_width 0 application rgw
pool 12 'default.rgw.buckets.data' replicated size 3 min_size 1 crush_rule 0 object_hash rjenkins pg_num 16 pgp_num 16 autoscale_mode warn last_change 1177 lfor 0/1177/1175 flags hashpspool stripe_width 0 application rgw
pool 13 'default.rgw.buckets.non-ec' replicated size 3 min_size 1 crush_rule 0 object_hash rjenkins pg_num 8 pgp_num 8 autoscale_mode warn last_change 1362 lfor 0/1362/1360 flags hashpspool stripe_width 0 application rgw
```

3、备份集群的crushmap文件。

```bash
ceph osd getcrushmap -o crushmap.bak
```

## 配置 crush class

默认情况下，所有 osd 都会 class 的类型是 hdd：

```bash
[root@ceph001 ~]# ceph osd crush class ls
[
    "hdd"
]
```

当前3个节点，每个节点上有2个OSD，因为没有挂SSD硬盘，所以把OSD3-5模拟为一组SSD。首先，需要将 osd3-5 从 hdd 组中去除掉：

```bash
[root@ceph001 ~]# for i in {3..5};do ceph osd crush rm-device-class osd.$i;done
done removing class of osd(s): 3
done removing class of osd(s): 4
done removing class of osd(s): 5
```

查看 osd

```bash
[root@ceph001 ~]# ceph osd tree
ID CLASS WEIGHT  TYPE NAME        STATUS REWEIGHT PRI-AFF
-1       0.07315 root default
-3       0.02438     host ceph001
 3       0.00490         osd.3        up  1.00000 1.00000
 0   hdd 0.01949         osd.0        up  1.00000 1.00000
-5       0.02438     host ceph002
 4       0.00490         osd.4        up  1.00000 1.00000
 1   hdd 0.01949         osd.1        up  1.00000 1.00000
-7       0.02438     host ceph003
 5       0.00490         osd.5        up  1.00000 1.00000
 2   hdd 0.01949         osd.2        up  1.00000 1.00000
```

可以看到 osd3-5  class 列已经没有 hdd 标识了。此时就可以通过命令将osd3-5添加到 ssd 组了，如下：

```bash
[root@ceph001 ~]# for i in {3..5}; do ceph osd crush set-device-class ssd osd.$i;done
set osd(s) 3 to class 'ssd'
set osd(s) 4 to class 'ssd'
set osd(s) 5 to class 'ssd'
# 查看 osd
[root@ceph001 ~]# ceph osd tree
ID CLASS WEIGHT  TYPE NAME        STATUS REWEIGHT PRI-AFF
-1       0.07315 root default
-3       0.02438     host ceph001
 0   hdd 0.01949         osd.0        up  1.00000 1.00000
 3   ssd 0.00490         osd.3        up  1.00000 1.00000
-5       0.02438     host ceph002
 1   hdd 0.01949         osd.1        up  1.00000 1.00000
 4   ssd 0.00490         osd.4        up  1.00000 1.00000
-7       0.02438     host ceph003
 2   hdd 0.01949         osd.2        up  1.00000 1.00000
 5   ssd 0.00490         osd.5        up  1.00000 1.00000
# 查看 class
[root@ceph001 ~]# ceph osd crush class ls
[
    "hdd",
    "ssd"
]
```

可以发现 osd3-5 的 class 列都变为 ssd，查看 crush class 也多出一个 ssd 的组，接下来就需要创建ssd 的规则。

## 命令生成osd树形结构

```bash
# 创建数据中心：datacenter0
ceph osd crush add-bucket datacenter0 datacenter
​
# 创建机房：room0
ceph osd crush add-bucket room0 room
​
# 创建机架：rack0、rack1、rack2（模拟3个机架）
ceph osd crush add-bucket rack0 rack
ceph osd crush add-bucket rack1 rack
ceph osd crush add-bucket rack2 rack
​
# 把机房room0移动到数据中心datacenter0下
ceph osd crush move room0 datacenter=datacenter0
​
# 把机架rack0、rack1、rack2移动到机房room0下
ceph osd crush move rack0 room=room0
ceph osd crush move rack1 room=room0
ceph osd crush move rack2 room=room0
​
# 把主机ceph001移动到：datacenter0/room0/rack0下
ceph osd crush move ceph001 datacenter=datacenter0 room=room0 rack=rack0
​
# 把主机ceph002移动到：datacenter0/room0/rack1下
ceph osd crush move ceph002 datacenter=datacenter0 room=room0 rack=rack1
​
# 把主机ceph003移动到：datacenter0/room0/rack2下
ceph osd crush move ceph003 datacenter=datacenter0 room=room0 rack=rack2
```

查看osd

```bash
[root@ceph001 ~]# ceph osd tree
ID  CLASS WEIGHT  TYPE NAME                STATUS REWEIGHT PRI-AFF
-13       0.07315 datacenter datacenter0
-14       0.07315     room room0
-15       0.02438         rack rack0
 -3       0.02438             host ceph001
  0   hdd 0.01949                 osd.0        up  1.00000 1.00000
  3   ssd 0.00490                 osd.3        up  1.00000 1.00000
-16       0.02438         rack rack1
 -5       0.02438             host ceph002
  1   hdd 0.01949                 osd.1        up  1.00000 1.00000
  4   ssd 0.00490                 osd.4        up  1.00000 1.00000
-17       0.02438         rack rack2
 -7       0.02438             host ceph003
  2   hdd 0.01949                 osd.2        up  1.00000 1.00000
  5   ssd 0.00490                 osd.5        up  1.00000 1.00000
 -1             0 root default
```

## 规则

crushmap配置中最核心的当属rule了，crush rule决定了三点重要事项：

- 1、从OSDMap中的哪个节点开始查找
- 2、使用那个节点作为故障隔离域
- 3、定位副本的搜索模式（广度优先 or 深度优先）。

> pg 选择osd的过程，首先要知道在rules中 指明从osdmap中哪个节点开始查找，入口点默认为default也就是root节点，
然后隔离域为host节点(也就是同一个host下面不能选择两个子节点)。由default到3个host的选择过程，
这里由default根据节点的bucket类型选择下一个子节点，由子节点再根据本身的类型继续选择，直到选择到host，然后在host下选择一个osd。

这里创建如下两个规则：

replicated_rule_rack：用于hdd分组的osd，以rack为故障域的应用规则；
replicated_rule_rack_ssd：用于普通ssd分区为osd，以rack为故障域的应用规则；

```bash
ceph osd crush rule create-replicated {name} {root} {failure-domain-type} {class}
//root，The name of the node under which data should be placed.即应该放置数据的root bucket的名称,例如default。

ceph osd crush rule create-replicated replicated_rule_rack datacenter0 rack hdd
ceph osd crush rule create-replicated replicated_rule_rack_ssd datacenter0 rack ssd
```

查看 rule

```bash
[root@ceph001 ~]# ceph osd crush rule ls
replicated_rule
replicated_rule_rack
replicated_rule_rack_ssd
# 查看规则具体信息
ceph osd crush rule dump rule_name
```

for i in $(rados lspools | grep -v ssdpool);do  ceph osd pool set $i crush_rule replicated_rule_rack ;done

存储池应用规则

应用上一步创建的replicated_rule_rack_ssd规则到除了数据池以外的存储池

```bash
for i in $(rados lspools | grep -v cephfs_data | grep -v default.rgw.buckets.data | grep -v testpool);do  ceph osd pool set $i crush_rule replicated_rule_rack_ssd ;done
```

应用上一步创建的replicated_rule_rack规则到数据池

```bash
ceph osd pool set testpool crush_rule replicated_rule_rack
ceph osd pool set default.rgw.buckets.data crush_rule replicated_rule_rack
ceph osd pool set cephfs_data crush_rule replicated_rule_rack
```

## Reference

- [ceph rack故障域调整](https://blog.51cto.com/wendashuai/2509156)
- [[ ceph ] CEPH 部署完整版（CentOS 7 + luminous）](https://www.cnblogs.com/hukey/p/11975109.html)
- [03 分布式存储ceph之crush规则配置](https://zhuanlan.zhihu.com/p/386560600)÷

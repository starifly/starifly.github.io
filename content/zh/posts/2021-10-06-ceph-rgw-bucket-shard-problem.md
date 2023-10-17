---
title: ceph rgw桶分片问题
author: starifly
date: 2021-10-06T19:31:32+08:00
lastmod: 2021-10-06T19:31:32+08:00
categories: [ceph]
tags: [ceph,rgw,bucket,shard]
draft: false
slug: ceph-rgw-bucket-shard-problem
---

## 背景说明

RGW 为每个 bucket 维护了一份索引，里面保存了 bucket 中全部对象的元数据。RGW 本身并没有足够有效的遍历对象的能力，所以在处理请求时，这些索引数据非常重要，比如遍历 bucket 中全部对象时。bucket 索引信息还有其他用处，比如为版本控制的对象维护日志、bucket 配额元数据和跨区同步的日志。bucket 索引不会影响对象的读操作，但写和修改确实会增加一些额外的操作。

这隐含了两层意思：其一，在单个 bucket 索引对象上能存储的数据总量有限，默认情况下，每个 bucket 是只有一个索引对象的，所以每个 bucket 中能存储的对象数量就是有限的了。超大的索引对象会造成性能和可靠性的问题，极端情况下，可能因为缓慢的恢复操作，造成 OSD 进程挂掉。其二，这成了性能瓶颈，因为所有对同一 bucket 的写操作，都会对一个索引对象做修改和序列化操作。

为此 ceph 引入bucket 分片功能来解决 bucket 中存储大量数据的问题，bucket 的索引数据可以存储在多个 RADOS 对象上了，这样 bucket 中存储对象的数量就可以随着索引数据的分片数量的增加而增加了。而且从 Luminous 开始最终引入了动态 bucket 分片，现在随着存储对象的增加，bucket 可以自动分片了。这项功能是默认打开的，将变量 rgw dynamic resharding 设置为 false（默认为 true），关闭自动分片；每个分片可存储的对象数量由该变量控制，rgw max objs per shard，默认是十万；自动分片线程扫描的间隔可以通过 rgw reshard thread interval 选项配置，默认为十分钟。虽然官方宣称自动分片是自动化的且不会阻塞业务的读写，但是实际上自动分片存在bug，会造成IO阻塞，所以建议最好关闭此项功能。

## 解决方案

1.首先index pool建议上SSD，即rgw中的pool除data池外，其它都采用ssd。

2.关闭动态resharding，不管怎么样不能影响到生产服务质量

```conf
rgw dynamic resharding = false
```

3.预估单个bucket需要存放的对象数量，按照每个shard 十万数据，提前做好分片，这有个好处是，后期不需要再进行resharding。设置shard的两个方法

- 方法1：rgw_override_bucket_index_max_shards

```conf
rgw_override_bucket_index_max_shards ：
配置在配置文件中，指定新建的bucket的shard数量，配置了这个参数，需要重启才能让参数生效。
```

- 方法2：multisite 中的 bucket_index_max_shards

```conf
这个参数需要配置在multisite zonegroup中的配置中,会覆盖rgw_override_bucket_index_max_shards 参数
```

获取zonegroup

```json
# radosgw-admin zonegroup --rgw-zonegroup=default get > /root/zonegroup.json
    
{
    "id": "3c5cc9c6-7e31-46cd-ac6d-652ce466c879",
    "name": "default",
    "api_name": "",
    "is_master": "true",
    "endpoints": [],
    "hostnames": [],
    "hostnames_s3website": [],
    "master_zone": "b70f0b56-08a0-472a-880f-6114f593c972",
    "zones": [
        {
            "id": "b70f0b56-08a0-472a-880f-6114f593c972",
            "name": "default",
            "endpoints": [],
            "log_meta": "false",
            "log_data": "false",
            "bucket_index_max_shards": 0,   // 这个参数
            "read_only": "false",
            "tier_type": "",
            "sync_from_all": "true",
            "sync_from": []
        }
    ],
    "placement_targets": [
        {
            "name": "default-placement",
            "tags": []
        }
    ],
    "default_placement": "default-placement",
    "realm_id": "f43494f8-bfb6-4723-b169-6ba929cdca1c"
}
```

> 修改 bucket_index_max_shards 参数

重新导入zonegroup

导入前先确认默认zonegroup有没有realm，如果没有realm，要先创建并修改zonegroup的realm

```bash
radosgw-admin realm create --rgw-realm=my-realm --default
radosgw-admin zonegroup modify  --rgw-zonegroup=default --rgw-realm=my-realm
```

```json
# radosgw-admin zonegroup --rgw-zonegroup=default set < /root/zonegroup.json
{
    "id": "3c5cc9c6-7e31-46cd-ac6d-652ce466c879",
    "name": "default",
    "api_name": "",
    "is_master": "true",
    "endpoints": [],
    "hostnames": [],
    "hostnames_s3website": [],
    "master_zone": "b70f0b56-08a0-472a-880f-6114f593c972",
    "zones": [
        {
            "id": "b70f0b56-08a0-472a-880f-6114f593c972",
            "name": "default",
            "endpoints": [],
            "log_meta": "false",
            "log_data": "false",
            "bucket_index_max_shards": 4,
            "read_only": "false",
            "tier_type": "",
            "sync_from_all": "true",
            "sync_from": []
        }
    ],
    "placement_targets": [
        {
            "name": "default-placement",
            "tags": []
        }
    ],
    "default_placement": "default-placement",
    "realm_id": "f43494f8-bfb6-4723-b169-6ba929cdca1c"
```

提交配置，让配置生效，这一步不能少

```bash
# radosgw-admin period update --commit
```

> 这个参数会对所有在这个zonegroup下面的zone其效果，而且不需要重启

方法一手动配置了rgw_override_bucket_index_max_shards参数，使得bucket在一定程度上保证不会有索引分片方面的性能问题，那么问题来了，手动设置该值，是让集群的bucket在一开始就分好片，产生的大量分片为即将进行的数据存储做好了准备，但是，线上业务反映，每天写入的文件量远不及我们一开始的配置，在分片数设置为768的情况下，单个bucket可以容纳7千6百80万个文件，而线上的业务进来的文件每天只有百万级别，事先分好的片不但浪费掉了，而且在持续写入之后会有性能不友好的问题。所以最好采用方法二，在创建bucket的时候就可以设置分片值，这样能很大程度减少浪费问题。

另外腾讯的cos还有阿里的oss对象存储理论上单桶可以存储无限个对象，据说是做了多桶联合，对外展示为一个桶，这个可能需要研发能力支撑，这里不做过多讨论。

## Reference

- [Ceph RGW bucket 自动分片介绍和存在的问题](https://www.jianshu.com/p/583d880b8a15)
- [动态设置bucket_index_max_shards参数](http://www.strugglesquirrel.com/2018/06/27/%E5%8A%A8%E6%80%81%E8%AE%BE%E7%BD%AEbucket-index-max-shards%E5%8F%82%E6%95%B0/)
- [Rgw设置shard避免单bucket下数量过大导致性能下降的方法](https://www.dovefi.com/post/rgw%E8%AE%BE%E7%BD%AEshard%E9%81%BF%E5%85%8D%E5%8D%95bucket%E4%B8%8B%E6%95%B0%E9%87%8F%E8%BF%87%E5%A4%A7%E5%AF%BC%E8%87%B4%E6%80%A7%E8%83%BD%E4%B8%8B%E9%99%8D%E7%9A%84%E6%96%B9%E6%B3%95/)

---
title: redis主从搭建
author: starifly
date: 2020-11-10T21:18:37+08:00
lastmod: 2020-11-10T21:18:37+08:00
categories: [redis]
tags: [redis]
draft: true
slug: redis-master-slave-build
---

## 多实例

复制一份原有的配置，重命名为 redis_6380.conf，并修改如下几处配置：

```conf
port 6380
pidfile /var/run/redis_6380.pid
dbfilename dump_6380.rdb
```

## 主从

```conf
masterauth <master-password>
slaveof <masterip> <masterport>
```

修改完配置后，重启从redis实例。

## 验证

登录master查看主从状态

```
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=6380,state=online,offset=29,lag=0
master_repl_offset:29
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:28
```

登录slave查看主从状态

```
127.0.0.1:6380> info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:6379
master_link_status:up
master_last_io_seconds_ago:3
master_sync_in_progress:0
slave_repl_offset:15
slave_priority:100
slave_read_only:1
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```

测试：在master上设定键值对，在slave上能够准确的查出来，证明主从配置成功。

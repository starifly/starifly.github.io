---
title: 单机部署 redis 伪集群
author: starifly
date: 2023-01-26T16:58:46+08:00
lastmod: 2023-01-26T16:58:46+08:00
categories: [redis]
tags: [redis]
draft: true
slug: deploy-redis-pseudo-cluster
---

单机部署 redis 伪集群

## 编译安装

```
tar xzf redis-5.0.14.tar.gz
cd redis-5.0.14
make PREFIX=/opt/redis-cluster install
```

## 集群配置

```
mkdir -p /opt/redis-cluster/conf
cp redis.conf /opt/redis-cluster/conf/redis.conf.default # 用作集群其他配置文件的蓝本
mkdir -p /opt/redis-cluster/data/6379
```

修改默认配置文件

```
port 6379
daemonize yes
pidfile /var/run/redis_6379.pid
loglevel notice
logfile "/var/log/redis_6379.log"
dir /opt/redis-cluster/data/6379
bind 0.0.0.0
protected-mode no
masterauth 123456
requirepass 123456
replica-read-only yes
cluster-enabled yes
cluster-config-file /opt/redis-cluster/data/6379/nodes-6379.conf
cluster-node-timeout 5000 # 取消注释
appendonly yes # 将 no 修改为 yes
```

创建配置文件

```
cd /opt/redis-cluster/conf
echo 9001.conf 9002.conf 9003.conf 9004.conf 9005.conf 9006.conf | xargs -n 1 cp -v redis.conf.default
sed -i 's/6379/9001/g'  9001.conf 
sed -i 's/6379/9002/g'  9002.conf 
sed -i 's/6379/9003/g'  9003.conf 
sed -i 's/6379/9004/g'  9004.conf 
sed -i 's/6379/9005/g'  9005.conf 
sed -i 's/6379/9006/g'  9006.conf
```

创建数据存储文件

```
cd /opt/redis-cluster/data
mkdir -p 9001 9002 9003 9004 9005 9006
# 后期可能需要删除该文件件下的文件，用于重建集群，所以，删除命令也写一下
rm -rf 900*/*
```

## 启动Redis cluster 节点

```
/opt/redis-cluster/bin/redis-server /opt/redis-cluster/conf/9001.conf
/opt/redis-cluster/bin/redis-server /opt/redis-cluster/conf/9002.conf
/opt/redis-cluster/bin/redis-server /opt/redis-cluster/conf/9003.conf
/opt/redis-cluster/bin/redis-server /opt/redis-cluster/conf/9004.conf
/opt/redis-cluster/bin/redis-server /opt/redis-cluster/conf/9005.conf
/opt/redis-cluster/bin/redis-server /opt/redis-cluster/conf/9006.conf
```

## 创建集群


快速创建法

```
/opt/redis-cluster/bin/redis-cli --cluster create 192.168.4.13:9001 192.168.4.13:9002 192.168.4.13:9003 192.168.4.13:9004 192.168.4.13:9005 192.168.4.13:9006 --cluster-replicas 1
```

> --cluster-replicas 1 这个指的是从机的数量，表示我们希望为集群中的每个主节点创建一个从节点。

通过该方式创建的带有从节点的机器不能够自己手动指定主节点，所以如果需要指定的话，需要自己手动指定，先创建或添加好主节点后，再通过添加集群从节点来处理。

```
/opt/redis-cluster/bin/redis-cli -a 123456 --cluster create 120.78.62.103:9001 120.78.62.103:9003 120.78.62.103:9005
# 注意同一个主从节点不要在同一台服务器上，要交叉创建，避免一台服务器宕机导致整个集群不可用。
/opt/redis-cluster/bin/redis-cli -a 123456--cluster add-node 192.168.163.132:6382 192.168.163.132:6379 --cluster-slave --cluster-master-id 117457eab5071954faab5e81c3170600d5192270
```

## 测试

```
/opt/redis-cluster/bin/redis-cli -c -h 192.168.4.13 -p 9001
192.168.4.13:9001> cluster info
cluster_state:ok
cluster_slots_assigned:16384
cluster_slots_ok:16384
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:6
cluster_size:3
cluster_current_epoch:6
cluster_my_epoch:1
cluster_stats_messages_ping_sent:94
cluster_stats_messages_pong_sent:103
cluster_stats_messages_sent:197
cluster_stats_messages_ping_received:98
cluster_stats_messages_pong_received:94
cluster_stats_messages_meet_received:5
cluster_stats_messages_received:197
192.168.4.13:9001> 
192.168.4.13:9001> 
192.168.4.13:9001> cluster nodes
26478e4a6faa0ada3f16cbf7130d9eb559de5adc 192.168.4.13:9006@19006 slave 7cc43b8fd3e692f3cb40cdb40620ac35a1a34d50 0 1640938537860 6 connected
dff87e8ba71ea7dd12a91883f134bfd8b22aee9c 192.168.4.13:9003@19003 master - 0 1640938538862 3 connected 10923-16383
dc43025b34ee78fbcef42f7425694bad67425eff 192.168.4.13:9005@19005 slave acca2c18e8b5a6e580ae7fc9fd2a77e1ab317f2c 0 1640938536859 5 connected
acca2c18e8b5a6e580ae7fc9fd2a77e1ab317f2c 192.168.4.13:9001@19001 myself,master - 0 1640938537000 1 connected 0-5460
7cc43b8fd3e692f3cb40cdb40620ac35a1a34d50 192.168.4.13:9002@19002 master - 0 1640938536000 2 connected 5461-10922
2d78d7f2f7da4fa34bc85df5fca466bc5f59edb7 192.168.4.13:9004@19004 slave dff87e8ba71ea7dd12a91883f134bfd8b22aee9c 0 1640938535000 4 connected
192.168.4.13:9001> set name Jack
-> Redirected to slot [5798] located at 192.168.4.13:9002
OK
192.168.4.13:9002> get name
"Jack"
```

## 启动服务

```
# /usr/lib/systemd/system/redis_9001.service
[Unit]
Description=Redis Datastore Server
After=network.target
[Service]
Type=notify
PIDFile=/var/run/redis_9001.pid
PermissionsStartOnly=true
ExecStart=/opt/redis-cluster/bin/redis-server /opt/redis-cluster/conf/9001.conf --supervised systemd
ExecReload=/bin/kill -USR2 
ExecStop=/opt/redis-cluster/bin/redis-cli -a foo -p 9001 shutdown
Restart=always
[Install]
WantedBy=multi-user.target
```

同样地创建 9002-9006 启动服务。

## Reference

- [必须是全网最全的Redis集群搭建教程](https://segmentfault.com/a/1190000015795054)
- [超详细，多图文介绍redis集群方式并搭建redis伪集群](https://baijiahao.baidu.com/s?id=1660566302486460631&wfr=spider&for=pc)
- [Redis 5.0 redis-cli --cluster help说明 ](https://www.cnblogs.com/zhoujinyi/p/11606935.html)
- [redis cluster部署和故障模拟](https://www.jianshu.com/p/545d1f649c45)
- [redis集群部署及java连接](https://blog.csdn.net/virtual251/article/details/113145215)
- [Redis Cluster集群原理+三主三从交叉复制实战+故障切换（十）](https://blog.csdn.net/weixin_44953658/article/details/120879397)

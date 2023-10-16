---
title: kafka部署
author: starifly
date: 2023-01-26T17:55:03+08:00
lastmod: 2023-01-26T17:55:03+08:00
categories: [kafka]
tags: [kafka]
draft: true
slug: deploy-kafka
---

## 介绍

> Kafka是分布式发布-订阅消息系统，最初由LinkedIn公司开发，之后成为之后成为Apache基金会的一部分，由Scala和Java编写。Kafka是一种快速、可扩展的、设计内在就是分布式的，分区的和可复制的提交日志服务。
>
> 它与传统系统相比，有以下不同：
>
> - 它被设计为一个分布式系统，易于向外扩展；
> - 它同时为发布和订阅提供高吞吐量；
> - 它支持多订阅者，当失败时能自动平衡消费者；
> - 它将消息持久化到磁盘，因此可用于批量消费，例如ETL，以及实时应用程序。

Kafka官网： <http://kafka.apache.org/>

学习推荐：<http://orchome.com/kafka/index>

官网下载：<http://kafka.apache.org/downloads>

## 基础概念

- Broker：Kafka集群包含一个或多个服务器，这些服务器就是Broker
- Topic：每条发布到Kafka集群的消息都必须有一个Topic
- Partition：是物理概念上的分区，为了提供系统吞吐率，在物理上每个Topic会分成一个或多个Partition，每个Partition对应一个文件夹
- Producer：消息产生者，负责生产消息并发送到Kafka Broker
- Consumer：消息消费者，向kafka broker读取消息并处理的客户端。
- Consumer Group：每个Consumer属于一个特定的组，组可以用来实现一条消息被组内多个成员消费等功能。

## 安装准备

1. 系统防火墙最好关闭，如果firewalld需要打开，要确保指定的端口开放规则在在全部放开规则的前面，不然会出现奇怪的问题。

```
    0     0 ACCEPT     tcp  --  *      *       192.168.4.14         0.0.0.0/0            tcp dpt:2181 ctstate NEW,UNTRACKED
   10   600 ACCEPT     tcp  --  *      *       192.168.4.14         0.0.0.0/0            tcp dpts:2888:3888 ctstate NEW,UNTRACKED
    0     0 ACCEPT     tcp  --  *      *       192.168.4.17         0.0.0.0/0            tcp dpt:2181 ctstate NEW,UNTRACKED
   36  2160 ACCEPT     tcp  --  *      *       192.168.4.17         0.0.0.0/0            tcp dpts:2888:3888 ctstate NEW,UNTRACKED
	0     0 ACCEPT     tcp  --  *      *       192.168.4.14         0.0.0.0/0            tcp dpt:9092 ctstate NEW,UNTRACKED
	0     0 ACCEPT     tcp  --  *      *       192.168.4.17         0.0.0.0/0            tcp dpt:9092 ctstate NEW,UNTRACKED
    0     0 ACCEPT     all  --  *      *       192.168.4.14         0.0.0.0/0
    1    60 ACCEPT     all  --  *      *       192.168.4.17         0.0.0.0/0
```

2. jdk安装

```
[root@vm413 opt]# java -version
java version "1.8.0_191"
Java(TM) SE Runtime Environment (build 1.8.0_191-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.191-b12, mixed mode)
```


3. zookeeper和kafka最好用独立的包进行安装，因为独立安装包有状态检查脚本，方便排查问题。

## 安装zookeeper

1. 去官网下载apache-zookeeper-3.6.3-bin.tar.gz，解压到指定目录，然后修改配置。

2. 创建data存储目录

`mkdir -p /opt/zookeeper/data`

3. 新增zoo.cfg配置文件

`cp conf/zoo_sample.cfg conf/zoo.cfg`

```
#修改data存储目录
dataDir=/opt/zookeeper/data
#增加集群配置
server.0=192.168.4.13:2888:3888
server.1=192.168.4.14:2888:3888
server.2=192.168.4.17:2888:3888
```

4. 新增myid文件

```
192.168.4.13：
   echo '0' > /opt/zookeeper/data/myid
192.168.4.14：
   echo '1' > /opt/zookeeper/data/myid
192.168.4.17：
   echo '2' > /opt/zookeeper/data/myid
```

5. 启动Zookeeper集群

手动启动方式：

`/opt/zookeeper/bin/zkServer.sh start`

systemd服务启动启动：

```
[Unit]
Requires=network.target
After=network.target
[Service]
Type=forking
LimitNOFILE=1048576
User=zookeeper
Group=zookeeper
ExecStart=/opt/zookeeper/bin/zkServer.sh start
ExecStop=/opt/zookeeper/bin/zkServer.sh stop
Restart=Always
[Install]
WantedBy=multi-user.target
```

启动并加入开机启动

`systemctl start zookeeper && systemctl enable zookeeper`

6. 查看Zookeeper集群状态

`/opt/zookeeper/bin/zkServer.sh status`，正常状态是一leader两follow。

## 安装kafka

1. 去官网下载kafka_2.13-2.6.2，解压到指定目录，然后修改配置。

2. 创建data存储目录

`mkdir -p /opt/kafka/data/kafka-logs`

3. kafka配置修改

```
vim config/server.properties

# 修改id值，三节点分别为0、1、2
broker.id=0
# 修改日志存放路径
log.dirs=/opt/kafka/data/kafka-logs
# 修改分区数
num.partitions=2
# 修改zookeeper连接地址
zookeeper.connect=192.168.4.13:2181,192.168.4.14:2181,192.168.4.17:2181
# 以下配置为新增
listeners = PLAINTEXT://192.168.4.13:9092
delete.topic.enable=true
default.replication.factor=3
auto.create.topics.enable=false
#min.insync.replicas=2
```

注意：如果是配置集群，下面信息必须修改：

（1）、broker.id：同一个集群中，每台机器均不能一样

（2）、zookeeper.connect：因为我有3台zookeeper服务器，所以在这里zookeeper.connect设置为3台，必须全部加进去

（3）、listeners：在配置集群的时候，必须设置，不然以后的操作会报找不到leader的错误

4. 启动kafka集群

手动启动方式：

`/opt/kafka/bin/kafka-server-start.sh -daemon /opt/kafka/config/server.properties`

systemd服务启动启动：

```
[Unit]
Requires=zookeeper.service
After=zookeeper.service
[Service]
Type=simple
LimitNOFILE=1048576
ExecStart=/opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties
ExecStop=/opt/kafka/bin/kafka-server-stop.sh
Restart=always
[Install]
WantedBy=multi-user.target
```

启动并加入开机启动

`systemctl start kafka && systemctl enable kafka`

## 验证

流程命令会连接不同的节点，可以顺带检测集群消息同步情况:

在node1上创建Topic -> 在node3上查看Topic -> 在node1上生产消息 -> 在node2上消费消息

1. 先创建一个Topic

```
#高版本命令 ./kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 3 --partitions 2 --topic test
/opt/kafka_2.13-2.6.2/bin/kafka-topics.sh --create --zookeeper 192.168.4.13:2181 --replication-factor 1 --partitions 1 --topic test
```

2. 查看一下建好的Topic

```
#./kafka-topics.sh --bootstrap-server localhost:9092 --list
/opt/kafka_2.13-2.6.2/bin/kafka-topics.sh --list --zookeeper 192.168.4.14:2181
```

3. 在建好的Topic下生产消息

```
[root@vm413 bin]# ./kafka-console-producer.sh --broker-list localhost:9092 --topic test
>Apple
>Banana
>Cat
>Dog
>Rerui
```

4. 消费指定Topic下的消息

```
[root@vm417 opt]# ./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
Apple
Cat
Banana
Dog
Rerui
```

5. 查看test topic消息

```
/opt/kafka_2.13-2.6.2/bin/kafka-topics.sh --describe --zookeeper 192.168.4.13:2181 --topic test
```

说明：

leader:负责处理消息的读和写，leader是从所有节点中随机选择的.

Replicas:列出了所有的副本节点，不管节点是否在服务中.

Lsr:是正在服务中的节点.

6. 删除指定Topic

```
./kafka-topics.sh --delete --bootstrap-server localhost:9092 --topic test
```
## Reference

- [centos7安装部署kafka_2.13-2.4.1集群 ](https://www.cnblogs.com/zhangmingcheng/p/13032460.html)

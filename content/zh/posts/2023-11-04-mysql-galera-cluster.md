---
keywords:
- 
title: "Mysql Galera Cluster"
date: 2023-11-04T10:58:51+08:00
lastmod: 2023-11-04T10:58:51+08:00
description: ""
draft: false
author: starifly
hideToc: false
enableToc: true
enableTocContent: false
tocFolding: false
tocLevels: ["h2", "h3", "h4"]
categories: [mysql]
tags: [mysql]
slug: mysql-galera-cluster
img:
---

## 介绍

**Galera Cluster主要功能**

1. 同步复制
2. 真正的multi-master，即所有节点可以同时读写数据库
3. 自动的节点成员控制，失效节点自动被清除
4. 新节点加入数据自动复制
5. 真正的并行复制，行级
6. 用户可以直接连接集群，使用感受上与MySQL完全一致

**优势**

1. 因为是多主，所以不存在Slavelag(延迟)
2. 不存在丢失事务的情况
3. 同时具有读和写的扩展能力
4. 更小的客户端延迟
5. 节点间数据是同步的，而Master/Slave模式是异步的，不同slave上的binlog可能是不同的

**缺点**

1. 加入新节点时开销大，需要复制完整的数据
2. 不能有效地解决写扩展的问题，所有的写操作都发生在所有的节点
3. 有多少个节点，就有多少份重复的数据
4. 由于事务提交需要跨节点通信，即涉及分布式事务操作，因此写入会比主从复制慢很多，节点越多，写入越慢，死锁和回滚也会更加频繁
5. 对网络要求比较高，如果网络出现波动不稳定，则可能会造成两个节点失联，Galera Cluster集群会发生脑裂，服务将不可用

**存在局限**

1. 仅支持InnoDB/XtraDB存储引擎，任何写入其他引擎的表，包括mysql.*表都不会被复制。但是DDL语句可以复制，但是insert into mysql.user(MyISAM存储引擎)之类的插入数据不会被复制
2. Delete操作不支持没有主键的表，因为没有主键的表在不同的节点上的顺序不同，如果执行select … limit …将出现不同的结果集
3. LOCK/UNLOCK TABLES/FLUSH TABLES WITH READ LOCKS不支持单表所锁，以及锁函数GET_LOCK()、RELEASE_LOCK()，但FLUSH TABLES WITH READ LOCK支持全局表锁
4. General Query Log日志不能保存在表中，如果开始查询日志，则只能保存到文件中
5. 不能有大事务写入，不能操作wsrep_max_ws_rows=131072(行)，且写入集不能超过wsrep_max_ws_size=1073741824(1GB)，否则客户端直接报错
6. 由于集群是乐观锁并发控制，因此，在commit阶段会有事务冲突发生。如果两个事务在集群中的不同节点上对同一行写入并提交，则失败的节点将回滚，客户端返回死锁报错
7. XA分布式事务不支持Codership Galera Cluster，在提交时可能会回滚
8. 整个集群的写入吞吐量取决于最弱的节点限制，集群要使用同一的配置

没有特别说明的，均是三节点都需执行

## 准备工作

禁用selinux

```sh
setenforce 0
sed -i 's/^SELINUX=.*$/SELINUX=disabled/' /etc/selinux/config
```

卸载系统自带的mariadb

```sh
yum erase `rpm -qa | grep mariadb`
```

添加yum源

```cfg
cat /etc/yum.repos.d/galera.repo
[galera4]
name = Galera
baseurl = https://releases.galeracluster.com/galera-4/centos/7/x86_64
gpgkey = https://releases.galeracluster.com/GPG-KEY-galeracluster.com
gpgcheck = 1
[mysql-wsrep8]
name = MySQL-wsrep
baseurl = https://releases.galeracluster.com/mysql-wsrep-8.0/centos/7/x86_64
gpgkey = https://releases.galeracluster.com/GPG-KEY-galeracluster.com
gpgcheck = 1
```

## 安装

```sh
yum install galera-4 mysql-wsrep-8.0
```

## 数据库配置

节点一：

```cfg
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/8.0/en/server-configuration-defaults.html

[mysqld]
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove the leading "# " to disable binary logging
# Binary logging captures changes between backups and is enabled by
# default. It's default setting is log_bin=binlog
# disable_log_bin
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M
#
# Remove leading # to revert to previous value for default_authentication_plugin,
# this will increase compatibility with older clients. For background, see:
# https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_default_authentication_plugin
default-authentication-plugin=mysql_native_password

datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

server_id=1
log_timestamps=SYSTEM
lower_case_table_names=1
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
innodb_flush_log_at_trx_commit=0
innodb_buffer_pool_size=128M
binlog_format=ROW
character_set_server = utf8mb4
wsrep_on=ON
wsrep_provider=/usr/lib64/galera-4/libgalera_smm.so
wsrep_node_name="node1"
wsrep_node_address="192.168.5.31"
wsrep_cluster_name="galera"
wsrep_cluster_address="gcomm://192.168.5.31,192.168.5.32,192.168.5.36"
wsrep_provider_options="gcache.size=128M; gcache.page_size=128M"
wsrep_slave_threads=4
wsrep_sst_method=rsync
wsrep_sst_auth=rsync:rsync123

[client]
default-character-set=utf8mb4
socket=/var/lib/mysql/mysql.sock

[mysql]
default-character-set=utf8mb4
socket=/var/lib/mysql/mysql.sock

#[mysqldump]
#socket=/var/lib/mysql/mysql.sock
#max_allowed_packet = 512M
#
#[mysqld_safe]
## 内存分配算法调优（默认malloc）
#malloc-lib=/usr/lib64/libjemalloc.so.1
#
#[mysqladmin]
#socket=/var/lib/mysql/mysql.sock
```

节点二：

只是`server_id`、`wsrep_node_name`和`wsrep_node_address`配置和节点一不一样

节点三：

只是`server_id`、`wsrep_node_name`和`wsrep_node_address`配置和节点一不一样

## 启动galera

节点一：

如果数据目录下存在galera.cache和gvwstate.dat文件，要先删掉这两个文件，再执行命令

```sh
mysqld_bootstrap
```

节点二：

```sh
systemctl start mysqld
```

节点三：

```sh
systemctl start mysqld
```

## 数据库初始化，在节点一执行即可

```sh
grep -i 'temporary password' /var/log/mysqld.log
mysql_secure_installation
```

完成初始化后，登录mysql，新增远程用户访问或直接允许root用户远程访问，此处使用root直接远程登陆。

```sql
use mysql;
update user set host='%' where user='root';
flush privileges;
```

## 集群状态监控

```sql
mysql> SHOW GLOBAL STATUS LIKE 'wsrep_%';
+------------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------+
| Variable_name                | Value                                                                                                                                           |
+------------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------+
| wsrep_local_state_uuid       | 7b94331e-35d1-11ee-85c9-8657a425f634                                                                                                            |
| wsrep_protocol_version       | 10                                                                                                                                              |
| wsrep_last_committed         | 344                                                                                                                                             |
| wsrep_replicated             | 0                                                                                                                                               |
| wsrep_replicated_bytes       | 0                                                                                                                                               |
| wsrep_repl_keys              | 0                                                                                                                                               |
| wsrep_repl_keys_bytes        | 0                                                                                                                                               |
| wsrep_repl_data_bytes        | 0                                                                                                                                               |
| wsrep_repl_other_bytes       | 0                                                                                                                                               |
| wsrep_received               | 4                                                                                                                                               |
| wsrep_received_bytes         | 440                                                                                                                                             |
| wsrep_local_commits          | 0                                                                                                                                               |
| wsrep_local_cert_failures    | 0                                                                                                                                               |
| wsrep_local_replays          | 0                                                                                                                                               |
| wsrep_local_send_queue       | 0                                                                                                                                               |
| wsrep_local_send_queue_max   | 1                                                                                                                                               |
| wsrep_local_send_queue_min   | 0                                                                                                                                               |
| wsrep_local_send_queue_avg   | 0                                                                                                                                               |
| wsrep_local_recv_queue       | 0                                                                                                                                               |
| wsrep_local_recv_queue_max   | 1                                                                                                                                               |
| wsrep_local_recv_queue_min   | 0                                                                                                                                               |
| wsrep_local_recv_queue_avg   | 0                                                                                                                                               |
| wsrep_local_cached_downto    | 1                                                                                                                                               |
| wsrep_flow_control_paused_ns | 0                                                                                                                                               |
| wsrep_flow_control_paused    | 0                                                                                                                                               |
| wsrep_flow_control_sent      | 0                                                                                                                                               |
| wsrep_flow_control_recv      | 0                                                                                                                                               |
| wsrep_flow_control_active    | false                                                                                                                                           |
| wsrep_flow_control_requested | false                                                                                                                                           |
| wsrep_cert_deps_distance     | 0                                                                                                                                               |
| wsrep_apply_oooe             | 0                                                                                                                                               |
| wsrep_apply_oool             | 0                                                                                                                                               |
| wsrep_apply_window           | 1                                                                                                                                               |
| wsrep_apply_waits            | 0                                                                                                                                               |
| wsrep_commit_oooe            | 0                                                                                                                                               |
| wsrep_commit_oool            | 0                                                                                                                                               |
| wsrep_commit_window          | 1                                                                                                                                               |
| wsrep_local_state            | 4                                                                                                                                               |
| wsrep_local_state_comment    | Synced                                                                                                                                          |
| wsrep_cert_index_size        | 0                                                                                                                                               |
| wsrep_causal_reads           | 0                                                                                                                                               |
| wsrep_cert_interval          | 0                                                                                                                                               |
| wsrep_open_transactions      | 0                                                                                                                                               |
| wsrep_open_connections       | 0                                                                                                                                               |
| wsrep_incoming_addresses     | AUTO,AUTO,AUTO                                                                                                                                  |
| wsrep_cluster_weight         | 3                                                                                                                                               |
| wsrep_desync_count           | 0                                                                                                                                               |
| wsrep_evs_delayed            |                                                                                                                                                 |
| wsrep_evs_evict_list         |                                                                                                                                                 |
| wsrep_evs_repl_latency       | 0/0/0/0/0                                                                                                                                       |
| wsrep_evs_state              | OPERATIONAL                                                                                                                                     |
| wsrep_gcomm_uuid             | 1c71deac-374b-11ee-b8be-7b9e717a4383                                                                                                            |
| wsrep_gmcast_segment         | 0                                                                                                                                               |
| wsrep_cluster_capabilities   |                                                                                                                                                 |
| wsrep_cluster_conf_id        | 18446744073709551615                                                                                                                            |
| wsrep_cluster_size           | 3                                                                                                                                               |
| wsrep_cluster_state_uuid     | 7b94331e-35d1-11ee-85c9-8657a425f634                                                                                                            |
| wsrep_cluster_status         | Primary                                                                                                                                         |
| wsrep_connected              | ON                                                                                                                                              |
| wsrep_local_bf_aborts        | 0                                                                                                                                               |
| wsrep_local_index            | 0                                                                                                                                               |
| wsrep_provider_capabilities  | :MULTI_PRIMARY:CERTIFICATION:PARALLEL_APPLYING:TRX_REPLAY:ISOLATION:PAUSE:CAUSAL_READS:INCREMENTAL_WRITESET:UNORDERED:PREORDERED:STREAMING:NBO: |
| wsrep_provider_name          | Galera                                                                                                                                          |
| wsrep_provider_vendor        | Codership Oy <info@codership.com>                                                                                                               |
| wsrep_provider_version       | 4.15(r86ced4c6)                                                                                                                                 |
| wsrep_ready                  | ON                                                                                                                                              |
+------------------------------+-------------------------------------------------------------------------------------------------------------------------------------------------+
66 rows in set (0.01 sec)
```

参数说明：

- wsrep_local_index = 2 在集群中的索引值
- wsrep_cluster_status为Primary，表示节点为主节点，正常读写。
- wsrep_ready为ON，表示集群正常运行。
- wsrep_connected: 如果该值为Off,且wsrep_ready的值也为Off,则说明该节点没有连接到集群
- wsrep_cluster_size为3，表示集群有三个节点。
- wsrep_cluster_state_uuid:在集群所有节点的值应该是相同的,有不同值的节点,说明其没有连接入集群。
- wsrep_cluster_conf_id:正常情况下所有节点上该值是一样的.如果值不同,说明该节点被临时”分区”了.当节点之间网络连接恢复的时候应该会恢复一样的值。
- wsrep_flow_control_paused:表示复制停止了多长时间.即表明集群因为Slave延迟而慢的程度.值为0~1,越靠近0越好,值为1表示复制完全停止.可优化wsrep_slave_threads的值来改善.
- wsrep_flow_control_sent:表示该节点已经停止复制了多少次.

常用命令

```sh
# 集群状态监控：
show global status like 'wsrep_%';

# 查看集群节点数：
show status like 'wsrep_cluster_size'；

# 查看集群同步状态：
show status like 'wsrep_local_state_comment'；
```

## Reference

- [centos8安装配置galeracluster集群](https://wbinks.com/archives/centos8%E5%AE%89%E8%A3%85%E9%85%8D%E7%BD%AEgaleracluster%E9%9B%86%E7%BE%A4)
- [MariaDB Galera Cluster - MySQL - dbaselife](http://dbaselife.com/doc/946/)
- [MariaDB Galera Cluster部署实战-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1120419)
- [绿色记忆:Galera学习笔记](https://blog.gmem.cc/galera-study-note)

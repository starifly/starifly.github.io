---
title: MySQL基于GTID的双主架构搭建
author: starifly
date: 2020-05-31T22:56:02+08:00
lastmod: 2020-05-31T22:56:02+08:00
categories: [mysql]
tags: [mysql,数据库,gtid]
draft: true
slug: mysql-dual-master-based-on-gtid
---

**MySQL双主（主主）架构方案思路**

- 两台mysql都可读写，互为主备，默认只使用一台（masterA）负责数据的写入，另一台（masterB）备用
- masterA是masterB的主库，masterB又是masterA的主库，它们互为主从
- 两台主库之间做高可用,可以采用keepalived等方案（使用VIP对外提供服务）

此方案masterB正常情况下会一直处于空闲状态，更进一步的方案是结合haproxy实现负载均衡。

**相关要求**

- 关闭selinux
- keepliaved两台机器时间需同步
- 建议采用高可用策略的时候，masterA或masterB均不因宕机恢复后而抢占VIP（非抢占模式），这样也能避免因为网络延迟，超过心跳检查时间，发生脑裂情况相互抢占MASTER导致写入相同数据引发的冲突。
- 两个节点的auto_increment_incremenet(自增步长)和auto_increment_offset(自增起始点)设为不同值。目的为了避免master意外宕机，可能会有部分binlog未能及时复制到slave上被应用，从而导致slave新写入数据的自增值和原先的master冲突，从offset起始点开始就错开了，避免了主键id的冲突，当然，如有合适的容错机制解决冲突话，也可以不这么设置。

**什么是GTID？**

- 全局唯一，一个事务对应一个GTID
- 替代传统的binlog+pos复制；使用master_auto_position=1自动匹配GTID断点进行复制
- MySQL5.6开始支持
- 在传统的主从复制中，slave端不用开启binlog；但是在GTID主从复制中，必须开启binlog
- slave端在接受master的binlog时，会校验GTID值
- 为了保证主从数据的一致性，多线程同时执行一个GTID

## mysql安装配置

masterA节点/etc/my.cnf配置

```conf
[mysqld]
port=3306
socket=/data/mysql/run/mysql.sock
pid_file=/data/mysql/run/mysql.pid
basedir=/data/mysql
datadir=/data/mysql/data
default_storage_engine=InnoDB
max_allowed_packet=128M
max_connections=1024
open_files_limit=65535

skip-name-resolve
lower_case_table_names=1

character-set-client-handshake=FALSE
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
init_connect='SET NAMES utf8mb4'


innodb_buffer_pool_size=128M
innodb_log_file_size=128M
innodb_file_per_table=1
innodb_flush_log_at_trx_commit=0


key_buffer_size=16M

# server-id是节点标识，主、从节点不能相同，必须全局唯一
server-id=1
# 起始值。一般填第n台主MySQL。此时为第一台主MySQL
auto_increment_offset=1
# 步进值auto_imcrement。一般有n台主MySQL就填n
auto_increment_increment=2
log-error=/data/mysql/log/mysql_error.log
log-bin=/data/mysql/log/mysql_bin.log
relay-log=/data/mysql/log/relay_log.log
gtid-mode=ON
enforce-gtid-consistency=ON
#log_slave_updates=ON # MySQL5.7可以不启用此参数，5.7版本使用了gtid_executed表记录同步复制的信息，避免两次写入relay-log和binlog，降低了从库磁盘I/O
master-info-repository=TABLE
relay-log-info-repository=TABLE
sync-master-info=1
sync_relay_log=1
sync_relay_log_info=1
binlog_format=row
binlog_row_image=minimal
binlog_checksum=CRC32
master_verify_checksum=1
slave_sql_verify_checksum=1
binlog_rows_query_log_events=1
#expire_logs_days=7
max_binlog_size=512m
binlog_cache_size=4m
max_binlog_cache_size=512m
sync_binlog=0
slow_query_log=1
slow_query_log_file=/data/mysql/log/mysql_slow_query.log
long_query_time=5


tmp_table_size=16M
max_heap_table_size=16M
query_cache_type=0
query_cache_size=0

# MySQL5.7新增加的值，配置基于表的组提交并行复制，默认值为database（基于库进行多线程复制，MySQL5.6是基于库的方式进行多线程方式复制）建议改为logical_clock，基于表的组方式复制，提高复制的效率。
slave_parallel_type=logical_clock

log_timestamps=SYSTEM
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

[client]
socket=/data/mysql/run/mysql.sock
#default-character-set=utf8mb4

[mysql]
#default-character-set=utf8mb4
socket=/data/mysql/run/mysql.sock
```

masterB节点/etc/my.cnf配置

```conf
[mysqld]
port=3306
socket=/data/mysql/run/mysql.sock
pid_file=/data/mysql/run/mysql.pid
basedir=/data/mysql
datadir=/data/mysql/data
default_storage_engine=InnoDB
max_allowed_packet=128M
max_connections=1024
open_files_limit=65535

skip-name-resolve
lower_case_table_names=1

character-set-client-handshake=FALSE
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
init_connect='SET NAMES utf8mb4'


innodb_buffer_pool_size=128M
innodb_log_file_size=128M
innodb_file_per_table=1
innodb_flush_log_at_trx_commit=0


key_buffer_size=16M

# server-id是节点标识，主、从节点不能相同，必须全局唯一
server-id=2
# 起始值。一般填第n台主MySQL。此时为第二台主MySQL
auto_increment_offset=2
# 步进值auto_imcrement。一般有n台主MySQL就填n
auto_increment_increment=2
log-error=/data/mysql/log/mysql_error.log
log-bin=/data/mysql/log/mysql_bin.log
relay-log=/data/mysql/log/relay_log.log
gtid-mode=ON
enforce-gtid-consistency=ON
#log_slave_updates=ON # MySQL5.7可以不启用此参数，5.7版本使用了gtid_executed表记录同步复制的信息，避免两次写入relay-log和binlog，降低了从库磁盘I/O
master-info-repository=TABLE
relay-log-info-repository=TABLE
sync-master-info=1
sync_relay_log=1
sync_relay_log_info=1
binlog_format=row
binlog_row_image=minimal
binlog_checksum=CRC32
master_verify_checksum=1
slave_sql_verify_checksum=1
binlog_rows_query_log_events=1
#expire_logs_days=7
max_binlog_size=512m
binlog_cache_size=4m
max_binlog_cache_size=512m
sync_binlog=0
slow_query_log=1
slow_query_log_file=/data/mysql/log/mysql_slow_query.log
long_query_time=5


tmp_table_size=16M
max_heap_table_size=16M
query_cache_type=0
query_cache_size=0

# MySQL5.7新增加的值，配置基于表的组提交并行复制，默认值为database（基于库进行多线程复制，MySQL5.6是基于库的方式进行多线程方式复制）建议改为logical_clock，基于表的组方式复制，提高复制的效率。
slave_parallel_type=logical_clock

log_timestamps=SYSTEM
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

[client]
socket=/data/mysql/run/mysql.sock
#default-character-set=utf8mb4

[mysql]
#default-character-set=utf8mb4
socket=/data/mysql/run/mysql.sock
```

## keepalived安装配置

masterA节点/etc/keepalived/keepalived.conf配置

```conf
! Configuration File for keepalived

global_defs {
   router_id mysql_76
   script_user root
   enable_script_security
}

vrrp_instance VI_1 {
    state BACKUP
    interface eno16777728
    virtual_router_id 50
    priority 100
    advert_int 1
    nopreempt
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        172.28.72.87 dev eno16777728
    }
}

virtual_server 172.28.72.87 3306 {
    delay_loop 2
    lb_algo wrr
    lb_kind DR
    persistence_timeout 60
    protocol TCP
    real_server 172.28.72.76 3306 {
        weight 3
    notify_down /etc/keepalived/mysql.sh
    TCP_CHECK {
            connect_timeout 3
            retry 3
            delay_before_retry 3
            connect_port 3306
        }
    }
}
```

masterA节点/etc/keepalived/keepalived.conf配置

```conf
! Configuration File for keepalived

global_defs {
   router_id mysql_84
   script_user root
   enable_script_security
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens32
    virtual_router_id 50
    priority 90
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        172.28.72.87 dev ens32
    }
}

virtual_server 172.28.72.87 3306 {
    delay_loop 2
    lb_algo wrr
    lb_kind DR
    persistence_timeout 60
    protocol TCP
    real_server 172.28.72.84 3306 {
        weight 3
    notify_down /etc/keepalived/mysql.sh
    TCP_CHECK {
            connect_timeout 3
            retry 3
            delay_before_retry 3
            connect_port 3306
        }
    }
}
```

notify_down脚本内容

```bash
#!/bin/bash
pkill keepalived
```

## 总结

**如果遇到主从延迟怎么解决？**

- 首先需要通过show slave statusG中 Seconds_Behind_Master观察主从之间延迟的状态
- slave节点服务器硬件配置不能与master节点相差太大，会大大导致复制的延迟
- 如果对延迟问题很敏感，可以考虑更换mariadb分支版本，或者直接上线mysql5.7最新版本，利用多线程复制的方式可以很大程度降低复制延迟

**其它**

- 如果主数据库已有数据，要先备份，备份之前要锁表（防止在从服务器还原的时候出错），FLUSH TABLES WITH READ LOCK;，备份完后就可以unlock tables;
- 在调试之前要加上skip-slave-start=1，调试完后再注释
- 调试当中利用reset master、reset slave、start master和start salve，它们会重置bin log和relay log，生产环境要谨慎使用

## Reference

- [MySQL双主（主主）架构方案](https://www.cnblogs.com/ygqygq2/p/6045279.html)
- [MySQL基于GTID的主从复制](https://blog.51cto.com/13434336/2178937)
- [MySQL5.7双主架构搭建（基于GTID方式）](https://blog.csdn.net/xiaoyi23000/article/details/80525625)
- [基于Mysql 5.7 GTID 搭建双主Keepalived 高可用](https://www.cnblogs.com/rayment/p/7249849.html)
- [MySQL+keepalived双主Gtid复制同步](https://segmentfault.com/a/1190000014484218)
- [企业级-Mysql双主互备高可用负载均衡架构（基于GTID主从复制模式）](https://my.oschina.net/u/588516/blog/756962?from=timeline)
- [高性能MySQL集群详解（二）](https://blog.51cto.com/6284444/2145990?source=dra)
- [mysql高可用集群部署方案（mha+atlas+keepalived）](https://blog.csdn.net/m0_37328475/article/details/82684699)
- [搭建MySQL高可用负载均衡集群](https://www.cnblogs.com/jpfss/p/8145742.html#_labelTop)
- [MySQL高可用负载均衡集群部署](https://blog.csdn.net/tony_328427685/article/details/86518340)
- [MySQL主备、主从、读写分离详解](https://blog.csdn.net/qq_40378034/article/details/91125768)

---
title: 部署cassandra集群
author: starifly
date: 2023-01-26T16:53:50+08:00
lastmod: 2023-01-26T16:53:50+08:00
categories: [cassandra]
tags: [cassandra]
draft: true
slug: deploy-cassandra-cluster
---

部署版本为3.11.9，其它版本目前发现默认账号密码cassandra/cassandra都没有权限。

安装jdk

- 192.168.4.13
- 192.168.4.14
- 192.168.4.17

其中 192.168.4.13 和 192.168.4.14 192.168.4.17 都作为seed(种子)节点

开放服务器端口7000、7001、9160、9042

## 配置

```
#集群名称替换，如有必要
sed -i 's$Test Cluster$CSD cluster$g' cassandra/conf/cassandra.yaml
```

```
#节点分配:192.168.4.13,192.168.4.14分配为seeds，三节点配置完全一样
sed -i 's$seeds: "127.0.0.1:7000"$seeds: "192.168.4.13,192.168.4.14，192.168.4.17"$g' cassandra/conf/cassandra.yaml
```

```
#下面两步替换为当前节点ip
sed -i 's$listen_address: localhost$listen_address: 192.168.4.13$g' cassandra/conf/cassandra.yaml
sed -i 's$rpc_address: localhost$rpc_address: 192.168.4.13$g' cassandra/conf/cassandra.yaml

# 注释broadcast_rpc_address: 127.0.0.1
```

```
#新建目录
mkdir /var/lib/cassandra
```

设置为开机启动

```
cat >/usr/lib/systemd/system/cassandra.service <<"EOF"
[Unit]  
Description=Cassandra Server Service  
After=network.service  
  
[Service]
Type=simple
User=cassandra
Group=cassandra
PIDFile=/var/run/cassandra.pid    
ExecStart=/opt/cassandra/bin/cassandra -f -p /var/run/cassandra.pid  
StandardOutput=journal  
StandardError=journal  
LimitNOFILE=100000  
LimitMEMLOCK=infinity  
LimitNPROC=32768  
LimitAS=infinity
RestartSec=5
Restart=always  
  
[Install]  
WantedBy=multi-user.target
EOF
```
`systemctl daemon-reload`

## 启动

#先启动seeds节点（192.168.4.13,192.168.4.14）再启动非seeds节点

`systemctl start cassandra`

查看集群状态

```
/opt/cassandra/bin/nodetool status
```

连接测试

```
[root@vm414 cassandra]# ./bin/cqlsh 192.168.4.17
Python 2.7 support is deprecated. Install Python 3.6+ or set CQLSH_NO_WARN_PY2 to suppress this message.

Connected to Test Cluster at 192.168.4.17:9042
[cqlsh 6.0.0 | Cassandra 4.0.1 | CQL spec 3.4.5 | Native protocol v5]
Use HELP for help.
cqlsh> describe keyspaces;

system       system_distributed  system_traces  system_virtual_schema
system_auth  system_schema       system_views
```

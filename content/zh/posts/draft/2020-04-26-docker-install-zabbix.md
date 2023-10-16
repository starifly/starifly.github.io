---
title: Docker安装zabbix
author: starifly
date: 2020-04-26T22:08:38+08:00
lastmod: 2020-04-26T22:08:38+08:00
categories: [docker,zabbix]
tags: [docker,zabbix,linux]
draft: true
slug: docker-install-zabbix
---

本次使用 docker 搭建 zabbix 的组合是 docker+mysql+zabbix+nginx。

**docker 部署 mysql**

```shell
docker run --name mysql-server -e MYSQL_ROOT_PASSWORD="root" -e MYSQL_USER="zabbix" -e MYSQL_PASSWORD="zabbix" -e MYSQL_DATABASE="zabbix" -d mysql:5.7 --character-set-server=utf8 --collation-server=utf8_bin
```

**安装zabbix-server**

```shell
docker run --name zabbix-server-mysql --link mysql-server:mysql -e DB_SERVER_HOST="mysql-server" -e MYSQL_USER="zabbix" -e MYSQL_PASSWORD="zabbix" -p 10051:10051 -d zabbix/zabbix-server-mysql:alpine-4.4.6
```

如果要利用 iostat 去监控磁盘，应该使用基于 centos 或者其它的镜像。

**安装zabbix-web-nginx**

```shell
docker run --name zabbix-web-nginx-mysql --link zabbix-server-mysql:zabbix-server --link mysql-server:mysql -e DB_SERVER_HOST="mysql-server" -e MYSQL_USER="zabbix" -e MYSQL_PASSWORD="zabbix" -e ZBX_SERVER_HOST="zabbix-server-mysql" -e PHP_TZ="Asia/Shanghai" -p 8000:80 -d zabbix/zabbix-web-nginx-mysql:alpine-4.4.6
```

至此可以登录访问测试，浏览器访问ip:8000查看。这里说明下，mysql、nginx在实际生产环境下，需要做数据卷的映射，防止数据丢失。

**安装zabbbix-agent**

```shell
docker run --name zabbix-agent -e ZBX_HOSTNAME="zabbix-agent" -e ZBX_SERVER_HOST="zabbix-server-mysql" --link zabbix-server-mysql:zabbix-server -e ZBX_SERVER_PORT=10051 -v /dev/sda:/dev/sda --privileged -p 10050:10050 -d zabbix/zabbix-agent:alpine-4.4.6
```

默认情况下，Docker容器是“无特权的”，并且无权访问大多数主机资源。 Zabbix代理旨在监视系统资源，为此必须授予Zabbix代理容器特权，或者也可以挂载某些系统范围的卷。 例如添加以下参数：-v /dev/sda:/dev/sda，--privileged。

实际生产环境建议 zabbix-agent 不要以容器运行。比如要监控宿主机的某个文件就不方便。

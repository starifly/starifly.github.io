---
title: centos安装yapi
author: starifly
date: 2021-02-24T21:45:38+08:00
lastmod: 2021-02-24T21:45:38+08:00
categories: [linux]
tags: [yapi,supervisor,linux,centos]
draft: true
slug: centos-setup-yapi
---

## 前言

Yapi是去哪儿网开源的一款接口管理工具。旨在为开发、产品、测试人员提供更优雅的接口管理服务。可以帮助开发者轻松创建、发布、维护 API。以及自动化生成接口文档。

## 环境准备

操作系统：centos 7 环境要求：(若已有一下环境则可跳过)

1.nodejs>7.6  
2.mongodb>2.6  
3.git

## 安装nodejs

1.获取资源（部署nodejs尽可能选择偶数版本，因为偶数版本官方有较长的维护时间，故这次选择8.x。）

```
curl -sL https://rpm.nodesource.com/setup_8.x | bash -
```

2.安装

```
yum install -y nodejs
```

3.查看版本

```
node -v
npm -v
```

## 安装mongodb

1.更新yum源，非必须但是更新一下无害

```
yum -y update
```

2.添加mongodb源文件

```
vim /etc/yum.repos.d/mongodb-org-3.4.repo
```

3.添加文件内容

```
[mongodb-org-3.4]  
name=MongoDB Repository  
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.4/x86_64/  
gpgcheck=1  
enabled=1  
gpgkey=https://www.mongodb.org/static/pgp/server-3.4.asc
```

4.安装mongodb

```
yum -y install mongodb-org
```

5.配置远程访问，修改mongod.conf配置文件

```
vim /etc/mongod.conf

注释 bindIp: 127.0.0.1 ： #bindIp: 127.0.0.1
```

6.启动MongoDB

```
systemctl start mongod.service
systemctl enable mongod.service
```

## 安装git

官方推荐，在内网部署的时候安装git，可以根据自己所需安装

```
yum -y install git
```

## 安装YApi

1.安装yapi

```
npm install -g yapi-cli --registry https://registry.npm.taobao.org
```

2.启动yapi

```
yapi server
```

根据提示，浏览器访问 http://部署YApi服务器的IP:9090，填写完信息后（注意部署路径选择/opt/my-yapi并赋予777权限，否则部署时可能因权限不足而报错），点击“开始部署”。（大概等待1分钟）。

3.启动服务

```
cd /opt/my-yapi
node vendors/server/app.js &
```

浏览器打开部署日志上的访问地址http://部署YApi服务器的IP:3000就可以访问搭建的YApi工具了。

## 部署Supervisor

Supervisor是守护进程服务，在没有守护进程之前存在一些问题：

应用程序运行在当前终端发起的子shell中，hangup信号中断后会导致应用退出，我们不可能在长期使用的环境中用一个终端去做长连接。  
当服务器重启后，还是需要人工连上服务器启动服务。  
进程出现意外终止，等人为发现再连上去开启，这样的反应显然已经慢了。

1.安装

```
yum install python-setuptools -y
easy_install supervisor
```

2.修改配置

创建目录/etc/supervisor

```
mkdir /etc/supervisor
```

创建supervisord.conf模板文件

```
echo_supervisord_conf > /etc/supervisor/supervisord.conf
```

修改文件supervisord.conf

```
vim /etc/supervisor/supervisord.conf
```

增加下面的内容，wq保存。

```
[include]
files = conf.d/*.conf
```

创建目录/etc/supervisor/conf.d/

```
mkdir -p /etc/supervisor/conf.d
```

修改文件YApi.conf

```
vim /etc/supervisor/conf.d/YApiGhost.conf
```

增加下面的内容，wq保存。

```
[program: YApiGhost]
command=node vendors/server/app.js ; 运行程序的命令
directory=/opt/my-yapi ; 命令执行的目录
autorestart=true ; 程序意外退出是否自动重启
stderr_logfile=/var/log/YApiGhost.err.log ; 错误日志文件
stdout_logfile=/var/log/YApiGhost.out.log ; 输出日志文件
environment=ASPNETCORE_ENVIRONMENT=Production ; 进程环境变量
user=root ; 进程执行的用户身份
stopsignal=INT
```

3.启动

```
supervisord -c /etc/supervisor/supervisord.conf
```

4.设置为开机启动

修改文件supervisord.service

```
vim /usr/lib/systemd/system/supervisord.service
```

添加下面的内容，wq保存。

```
[Unit]
Description=Supervisor daemon

[Service]
Type=forking
ExecStart=/usr/bin/supervisord -c /etc/supervisor/supervisord.conf
ExecStop=/usr/bin/supervisorctl shutdown
ExecReload=/usr/bin/supervisorctl reload
KillMode=process Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
```

开机启动

```
systemctl enable supervisord
```

## Reference

- [linux搭建Yapi环境](https://www.cnblogs.com/lbky/articles/11662281.html)
- [CentOS 7 部署 YApi](https://www.linuxidc.com/Linux/2018-01/150513.htm)
- [centos7安装MongoDB3.4](https://www.cnblogs.com/web424/p/6928992.html)

---
title: nginx单机安装
author: starifly
date: 2023-01-26T18:38:54+08:00
lastmod: 2023-01-26T18:38:54+08:00
categories: [nginx]
tags: [nginx]
draft: true
slug: nginx-install
---

## 安装

系统自带的openssl可能版本比较低有漏洞，所以安装的时候用比较新的版本

```
cd /src/
wget https://www.openssl.org/source/openssl-1.1.1m.tar.gz
tar zxfv openssl-1.1.1m.tar.gz
```

```
wget https://nginx.org/download/nginx-1.20.2.tar.gz
tar -zxvf nginx-1.20.2.tar.gz
cd nginx-1.20.2
./configure --user=nginx --group=nginx --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --conf-path=/etc/nginx/nginx.conf --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --with-http_gzip_static_module --with-http_stub_status_module --with-http_ssl_module --with-pcre --with-http_v2_module --with-openssl=/src/openssl-1.1.1m
make && make install
useradd -s /sbin/nologin -M nginx
```

## 添加启动服务

```
# /etc/systemd/system/nginx.service
[Unit]
Description=nginx server daemon
Documentation=http://nginx.org/en/docs/
After=network.target

[Service]
Type=forking
PIDFile=/var/run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
ExecReload=/usr/sbin/nginx -s reload
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true
Restart=always
RestartSec=20

[Install]
WantedBy=multi-user.target
```

```
systemctl daemon-reload
systemctl start nginx
systemctl enable nginx
```

## Reference

- [Nginx升级加固SSL/TLS协议信息泄露漏洞(CVE-2016-2183)和HTTP服务器的缺省banner漏洞](https://www.cnblogs.com/you-men/p/13642078.html)

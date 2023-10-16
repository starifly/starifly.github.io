---
title: nginx添加https配置
author: starifly
date: 2023-01-26T18:41:04+08:00
lastmod: 2023-01-26T18:41:04+08:00
categories: [nginx]
tags: [nginx]
draft: true
slug: nginx-add-https-configure
---

nginx需要安装ssl模块来使用https，查看nginx版本与编译安装了哪些模块

`/usr/local/nginx/sbin/nginx -V`

如果出现 (configure arguments: --with-http_ssl_module), 则已安装。

## 配置ssl证书

这里创建的是自签名证书（浏览器访问会提示不安全），进入nginx配置目录

1.创建服务器私钥，命令会让你输入一个口令：

`openssl genrsa -des3 -out server.key 1024`

2.创建签名请求的证书（CSR）

`openssl req -new -key server.key -out server.csr`

3.在加载SSL支持的Nginx并使用上述私钥时除去必须的口令

```
cp server.key server.key.org
openssl rsa -in server.key.org -out server.key
```

4.最后标记证书使用上述私钥和CSR

`openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt`

## 添加证书到ngix

修改nginx.conf

```
server {
	#加入443端口
    listen  443 ssl;
    server_name  localhost;
    #启用ssl
    ssl  on;
    #root html;
    #index index.html index.htm;
    ssl_certificate  ssl_cert/server.pem;
    ssl_certificate_key  ssl_cert/server.key;
    ssl_session_timeout 5m;
    #ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    #ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    #ssl_prefer_server_ciphers on;
    location / { root html; index index.html index.htm; }
}

server {
    listen 80;
    server_name blog.top;
    #将请求转成https
    rewrite ^(.*)$ https://$host$1 permanent;
}
```

重新加载配置

`./nginx -s reload`

## Reference

- [nginx添加https支持](http://www.voidcn.com/article/p-vkefslgr-bqw.html)

---
title: 使用Webhook和Github实现hugo的自动部署
author: zhoucaiqi@gmail.com
date: 2018-08-18T12:18:32+08:00
categories: [CI]
tags: [CI, webhook, github, hugo]
slug: webhook-github-autodeploy
---

之前[《通过github和caddy实现hugo的自动部署》](https://itlaws.cn/post/hugo-caddy-autodeplay/)，使用的是 caddy 的 http.git 插件。最近发现，那方法不管用了，不知道原因。

所以决定使用传统的 webhook 。原理是：在 vps 运行 webhook 监听程序，github 收到 push 事件后，

就通知该监听程序，由该监听程序执行相应的命令。现在记录一下。

<!--more-->

## 安装 webhook

```bash
sudo apt install webhook
```

也可以到该[项目](https://github.com/adnanh/webhook)的github地址手动[下载](https://github.com/adnanh/webhook/releases)。

## 编写 webhook 的配置文件

我的`itlaws.cn`的markdown源码放在`/var/www/itlaws.cn`

```bash
mkdir /var/www/itlaws.cn/webhook
touch itlaws.cn-config.json
touch itlaws.cn-deploy.sh
chmod +x itlaws.cn-deploy.sh
```

`itlaws.cn-config.json`内容为：

```json
[
  {
    "id": "itlaws.cn-deploy",
    "execute-command": "/var/www/itlaws.cn/webhook/itlaws.cn-deploy.sh",
    "command-working-directory": "/var/www/itlaws.cn"
  }
]
```

其中：

- `id` 是监听程序名称，自定义。
- `execute-command` 是监听程序接收到 github 的通知后执行的命令，自定义。
- `command-working-directory` 指定目录运行上述命令。

更多的参数见官方的[Hook definition](https://github.com/adnanh/webhook/blob/master/docs/Hook-Definition.md)。

`itlaws.cn-deploy.sh`内容为：

```bash
#!/bin/bash

cd /var/www/itlaws.cn && git pull --recurse-submodules && rm -rf public && hugo
```



## 测试监听程序

```bash
webhook -hooks /var/www/itlaws.cn/webhook/itlaws.cn-config.json -verbose -port=9000
```

其中：

- `-hooks` 后面要接配置文件；

- `-verbose`表示输出日志；

- `-port`表示监听端口，自定义。

更多的内容见[Webhook parameters](https://github.com/adnanh/webhook/blob/master/docs/Webhook-Parameters.md)。

上述命令将在9000端口运行webhook监听程序，即：

```bash
http://itlaws.cn:9000/hooks/itlaws.cn-deploy
```

在 github 端测试，正常。该监听程序目前是 http ，而不是 https 。

## 通过 supervisor 运行监听程序

`supervisor`是用于启动某种服务并监控该服务，在该服务异常退出时重启该服务。

```bash
sudo apt install supervisor
cd /etc/supervisor/conf.d/
touch webhook.conf
```

`webhook.conf`的内容为：

```bash
[program:webhook]
command=/var/www/itlaws.cn/webhook/webhook -hooks  /var/www/itlaws.cn/webhook/itlaws.cn-config.json -verbose -port=9000 -secure -cert ssl证书的路径 -key ssl密钥路径
user=www-data
directory=/var/www/itlaws.cn/webhook
autorestart=true
redirect_stderr=true

```

其中，`-secure`表示启用 `https`，`-cert `和`-key`分别是 https 的证书和密钥的路径，从而把监听服务从 http 改为 https：

```bash
https://itlaws.cn:9000/hooks/itlaws.cn-deploy
```

 好了，大功告成。


 [原文链接](https://itlaws.cn/post/webhook-github-autodeplay/)

---
title: 使用 supervisor在一个 docker container 中开启多个服务
author: starifly
date: 2020-03-14T19:46:19+08:00
lastmod: 2020-03-14T19:46:19+08:00
categories: [docker]
tags: [docker,supervisor]
draft: true
slug: docker-supervisor
---

## Dockerfile

```conf
FROM centos:7
MAINTAINER xxx "xxx@123.com"

RUN yum -y install libedit sudo && \
    yum install -y epel-release && \
    yum install -y iproute net-tools python-setuptools hostname inotify-tools yum-utils which && \
    yum clean all && \
    easy_install supervisor

RUN mv /etc/localtime /etc/localtime_bak
RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

RUN mkdir -p /var/log/supervisor/ /etc/supervisord.conf.d/
RUN echo -e '[include]\nfiles = /etc/supervisord.conf.d/*.ini\n[supervisord]\npidfile = /var/run/supervisord.pid\nlogfile = /var/log/supervisor/supervisord.log\nloglevel = debug\n[supervisorctl]\nserverurl = unix:///tmp/supervisor.sock\n[rpcinterface:supervisor]\nsupervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface\n[unix_http_server]\nfile = /tmp/supervisor.sock\n' > /etc/supervisord.conf
#COPY supervisord.ini /etc/supervisord.d/
COPY supervisord.ini /etc/supervisord.conf.d/

#ENTRYPOINT /usr/bin/supervisord  -c /etc/supervisord.conf
CMD ["/usr/bin/supervisord", "-n", "-c", "/etc/supervisord.conf"]
```

## supervisord.ini

```conf
[supervisord]
nodaemon=true

[program:ServerName]
command=/start.sh
autostart=true
startsecs=10
startretries=3
stdout_logfile_maxbytes=10MB
stderr_logfile_maxbytes=10MB
stdout_logfile=/var/log/server.out.log
stderr_logfile=/var/log/server.err.log
```

autorestart 没有设置为 true，因为如果服务由于某些因素起不来，supervisor 会无限重启服务。

> supervisor 是不能监控守护进程, 那么应该也是无法监控后台启动的程序。所以监控的程序应前台运行。

## Reference

- https://hub.docker.com/r/caio2k/centos7/dockerfile

---
title: Centos6和Centos7设置程序开机启动
author: starifly
date: 2019-07-29T22:32:04+08:00
categories: [linux]
tags: [linux,centos,redis]
draft: true
slug: centos6-and-centos7-setup-program-boot
---

## Centos 6

### Redis

进入`redis`安装目录，`cp utils/redis_init_script /etc/init.d/redis`。

修改`/etc/init.d/redis`中的以下选项（改为与本机`redis`实例相同配置即可）

```
#!/bin/sh
#
# Simple Redis init.d script conceived to work on Linux systems
# as it does use of the /proc filesystem.

# chkconfig: 2345 10 90
# description: Start and Stop redis 

REDISPORT=6379
EXEC=/usr/local/bin/redis-server
CLIEXEC=/usr/local/bin/redis-cli

PIDFILE=/var/run/redis_${REDISPORT}.pid
#CONF="/etc/redis/${REDISPORT}.conf"
CONF="/xxx/redis.conf"

case "$1" in
    start)
        if [ -f $PIDFILE ]
        then
                echo "$PIDFILE exists, process is already running or crashed"
        else
                echo "Starting Redis server..."
                $EXEC $CONF
        fi
        ;;
    stop)
        if [ ! -f $PIDFILE ]
        then
                echo "$PIDFILE does not exist, process is not running"
        else
                PID=$(cat $PIDFILE)
                echo "Stopping ..."
                $CLIEXEC -p $REDISPORT -a password shutdown
                while [ -x /proc/${PID} ]
                do
                    echo "Waiting for Redis to shutdown ..."
                    sleep 1
                done
                echo "Redis stopped"
        fi
        ;;
    *)
        echo "Please use start or stop as first argument"
        ;;
esac
```

## Centos 7

### Redis

```
[Unit]
Description=Redis daemon
After=network.target

[Service]
Type=forking
ExecStart=/xxx/src/redis-server /xxx/redis.conf
ExecStop=/xxx/src/redis-cli -a password shutdown

[Install]
WantedBy=multi-user.target
```

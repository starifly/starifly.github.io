---
title: centos7上安装配置tomcat8.5
author: starifly
date: 2019-08-03T21:50:34+08:00
categories: [tomcat,linux]
tags: [tomcat,centos,linux]
draft: true
slug: centos7-setup-tomcat8
---

## 安装

### 创建tomcat专有用户

```shell
groupadd tomcat
useradd -g tomcat -s /sbin/nologin tomcat
```

### 下载安装与配置&设置用户组权限并创建软连接

```shell
# wget http://mirror.bit.edu.cn/apache/tomcat/tomcat-8/v8.5.42/bin/apache-tomcat-8.5.42.tar.gz
# tar -zxvf apache-tomcat-8.5.42.tar.gz
# mv apache-tomcat-8.5.42 /usr/local/
# cd /usr/local/
# chown -hR tomcat:tomcat apache-tomcat-8.5.42
# ln -s apache-tomcat-8.5.42 tomcat
```

## 添加tomcat自启动systemd服务单元文件

1、为Tomcat添加启动参数

catalina.sh 在执行的时候会调用同级路径下的 setenv.sh 来设置额外的环境变量，因此在 setenv.sh 文件中添加如下内容：

```conf
JAVA_HOME=$(dirname $(dirname $(readlink $(readlink $(which javac)))))
PATH=$JAVA_HOME/bin:$PATH
CATALINA_HOME=/opt/tomcat
CATALINA_BASE=/opt/tomcat
CATALINA_PID="$CATALINA_BASE/tomcat.pid"
export JAVA_HOME
export PATH
export CATALINA_HOME
export CATALINA_BASE
```

2、在/lib/systemd/system路径下添加tomcat.service文件，内容如下：

```conf
[Unit]
Description=tomcat
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/opt/tomcat/tomcat.pid
ExecStart=/opt/tomcat/bin/startup.sh
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

重载systemd服务单元，给予软连接目录权限，启动Apache Tomcat服务并设置Tomcat为开机自启动

```shell
# systemctl daemon-reload
# systemctl start tomcat
# systemctl enable tomcat
```

然后可以查看状态，`systemctl status tomcat`

## Reference

- [centos7上安装配置tomcat8.5](https://blog.51cto.com/11555417/2408060)
- [Linux 平台下 Tomcat 的安装与优化](https://mp.weixin.qq.com/s/EclV2Yk4uUh-PhcL8qYdeg)
- [linux学习-将seafile启动脚本设置为开机启动服务](https://blog.51cto.com/11555417/2151938)
- [CentOS7设置tomcat开机自启动](https://www.jianshu.com/p/f0f8458e1631)

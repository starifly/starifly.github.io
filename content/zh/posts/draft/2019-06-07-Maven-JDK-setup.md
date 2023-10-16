---
title: Maven和JDK环境安装
author: starifly
date: 2019-06-07T21:47:24+08:00
categories: [jdk]
tags: [jdk,maven,jenkins,linux]
draft: true
slug: Maven-JDK-setup
---

## JDK 环境

1. 卸载自带的jdk

```shell
rpm -qa | grep java | xargs rpm -e --nodeps
rpm -qa | grep jdk | xargs rpm -e --nodeps
rpm -qa | grep qcj | xargs rpm -e --nodeps
```

2. 安装JDK

直接yum安装1.8.0版本openjdk

```shell
yum install java-1.8.0-openjdk* -y
```

3. 添加环境变量

```shell
vim /etc/profile
```

```Bash
JAVA_HOME=/usr/lib/jvm/java
JRE_HOME=$JAVA_HOME/jre
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/jre/lib/rt.jar
M2_HOME=/usr/local/maven3
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin:$M2_HOME/bin
export JAVA_HOME JRE_HOME CLASS_PATH M2_HOME PATH
```

使得配置生效

```shell
. /etc/profile
```

## 安装maven

1. 下载安装包

```shell
wget http://mirrors.hust.edu.cn/apache/maven/maven-3/3.1.1/binaries/apache-maven-3.1.1-bin.tar.gz
```

2. 解压

```shell
tar zxf apache-maven-3.1.1-bin.tar.gz
```

3. 移动到系统目录

```shell
mv apache-maven-3.1.1 /usr/local/maven3
```

4. 设置环境变量  
如上

Ps:  
jenkins 在使用 maven 自动构建时提示找不到 mvn 命令  
Jenkins 通过 shell 脚本调用 mvn 命令的时候，是从 /usr/bin 文件夹中找命令的，这个时候需要做个软链接  

```shell
ln -s /usr/local/maven3/bin/mvn /usr/bin/mvn
```

## Reference

- <https://www.cnblogs.com/fb010001/p/8522356.html>

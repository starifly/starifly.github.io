---
title: centos7卸载mariadb安装mysql
author: starifly
date: 2019-09-22T16:55:49+08:00
categories: [mysql]
tags: [mysql,数据库,centos]
draft: true
slug: centos7-remove-mariadb-setup-mysql
---

## 卸载mariadb

```shell
rpm -qa | grep mariadb    //当前安装列表
rpm -e --nodeps mariadb-libs-5.5.56-2.el7.x86_64    //卸载
rpm -qa | grep mariadb    //检查卸载干净没
find / -name mysql    //查找有没有残余
```

## 安装mysql

```shell
wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm    //下载mysql的repo源
rpm -ivh mysql-community-release-el7-5.noarch.rpm    //安装mysql-community-release-el7-5.noarch.rpm包
yum install mysql-server    //安装mysql
```

## 启动mysql并加入开机启动

```shell
systemctl enable mysqld
systemctl start mysqld
```

## 配置mysql（设置密码等）

```shell
mysql_secure_installation
```

## 开启远程访问权限

```shell
mysql -u root -p
mysql> use mysql;
mysql> grant all privileges  on *.* to root@'%' identified by "password";
mysql> flush privileges;
```

>如果远程访问不了，查询端口是不是被防火墙阻挡。

## Reference

- [centos7卸载mariadb安装mysql](https://www.cnblogs.com/lic1005/p/7989775.html)
- [CentOS 7 yum安装MySQL5.6](https://blog.csdn.net/xizaihui/article/details/52962057)
- [开启MySQL远程访问权限 允许远程连接](https://www.cnblogs.com/weifeng1463/p/7941625.html)

---
title: MySQL修改密码方法
author: starifly
date: 2019-09-22T21:50:40+08:00
categories: [mysql]
tags: [mysql,数据库]
draft: true
slug: MySQL-changes-password-methods
---

## 用mysqladmin

```shell
mysqladmin -u root password 'newpass'
# 如果 root 已经设置过密码，采用如下方法
# root 为用户名,password 后面是新密码，回车后输入旧密码即可
mysqladmin -u root -p password root@root
```

## 用SET PASSWORD命令

```shell
mysql -u root
mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('newpass');
```

## 用UPDATE直接编辑user表

>如果是5.7版本，首次修改密码是不能使用update语句修改密码，可用以上两种方法

```shell
mysql -u root
mysql> use mysql;
mysql> UPDATE user SET Password = PASSWORD('newpass') WHERE user = 'root';
# 新版本是 authentication_string 字段
mysql> update user set authentication_string=password('HEpan693640.') where user='root';
mysql> FLUSH PRIVILEGES;
```

## 用ALTER

```shell
ALTER USER 'root'@'localhost' IDENTIFIED BY 'NewPassword';
```

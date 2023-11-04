---
keywords:
- 
title: "mysql授予账户权限问题"
date: 2023-11-04T10:49:13+08:00
lastmod: 2023-11-04T10:49:13+08:00
description: "mysql授予账户权限问题"
draft: false
author: starifly
hideToc: false
enableToc: true
enableTocContent: false
tocFolding: false
tocLevels: ["h2", "h3", "h4"]
categories: [mysql]
tags: [mysql]
slug: mysql-account-permission-grant-problem
img:
---

mysql通过root账号登录，新建账户并授予权限会报错：

```sql
mysql> GRANT SELECT ON *.* TO 'select_user'@'%';
ERROR 1045 (28000): Access denied for user 'root'@'%' (using password: YES)
```

解决方案：

```sql
mysql> update mysql.user set Grant_priv='Y' where user = 'root' and host = '%';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

然后退出登录，重启mysql服务，之后就能正常授权了。

## Reference

- [MySQL错误：Access denied for user &#39;root&#39;@&#39;%&#39; to database &#39;xxx&#39; - 我想养只狗 - 博客园](https://www.cnblogs.com/Dog1363786601/p/17226101.html)

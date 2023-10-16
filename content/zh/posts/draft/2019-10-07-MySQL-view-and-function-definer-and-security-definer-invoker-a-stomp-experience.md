---
title: MySQL视图和函数定义者和安全性definer/invoker的一次踩坑经历
author: starifly
date: 2019-10-09T22:01:57+08:00
categories: [mysql]
tags: [mysql,数据库,视图]
draft: true
slug: MySQL-view-and-function-definer-and-security-definer-invoker-a-stomp-experience
---

## 视图

最近服务器迁移，同时关闭了mysql数据库远程访问，就是root只能本地访问，但是同事在访问一个视图的时候一直报错

The user specified as a definer('root'@'%') does not exist

![](https://img-blog.csdnimg.cn/20190128153328437.png)

原本只想打开访问权限，后面是想办法把%改为localhost,一种方法是直右键视图名，点击设计选择高级把%改为localhost就可以

![](https://img-blog.csdnimg.cn/20190128153248669.png)

或者是直接用SQL语句修改：

```shell
SELECT TABLE_SCHEMA '数据库',DEFINER '原定义者',CONCAT('ALTER ALGORITHM=UNDEFINED DEFINER=`root`@`%` SQL SECURITY DEFINER VIEW ',TABLE_SCHEMA,'.', TABLE_NAME, ' AS ',VIEW_DEFINITION,';') '修改视图定义者SQL'
FROM information_schema.`views`
WHERE DEFINER<>'root@%' and TABLE_SCHEMA='db_name';
```

把修正的SQL复制出来运行，视图定义者就会修改为“root@%”。

参考文档：[http://pdf.us/2018/02/24/679.html](http://pdf.us/2018/02/24/679.html)

### 坑

昨天在做MySQL数据库用户权限规范的时候，将原来业务帐号'user'@'%'修改为'user'@'192.168.1.%'时，忽然发现业务系统部分业务报404错误，后面找到程序员排查原因，发现是因为报错的业务在调用数据库时，使用了视图，而视图的定义者是原来的'user'@'%'，并且安全性设置的是definer，现在这个用户不存在了，虽然新用户'user'@'192.168.1.%’拥有和原来相同的数据库权限，但是依然不能使用这些视图。

找到了原因，那么解决办法也就出来了：一是将所有视图的定义者修改为'user'@'192.168.1.%'；二是将安全性定义由definer设置为invoker。因为是生产环境，不能随便搞，所以采用的是第一种方法，第二种方法没有再试。

![](http://pdf.us/wp-content/uploads/2018/02/TIM%E6%88%AA%E5%9B%BE20180224103412.png)

### 原理

**definer和invoker的区别**

在创建视图或者是存储过程的时候，是需要定义安全验证方式的(也就是安全性SQL SECURITY)，其值可以为definer或invoker，表示在执行过程中，使用谁的权限来执行。

definer:由definer(定义者)指定的用户的权限来执行

invoker:由调用这个视图(存储过程)的用户的权限来执行

* * *

definer

当定义为DEFINER时，必须数据库中存在DEFINER指定的用户，并且该用户拥有对应的操作权限，才能成功执行。与当前用户是否有权限无关。

示例：

CREATE DEFINER=`dev`@`%` PROCEDURE `p_user_login`(IN u\_name VARCHAR(25), IN u\_password VARCHAR(100))  
BEGIN  
SELECT u.id, u.name, u.tid, u.status, u.is\_report FROM v\_user u WHERE u.name=u\_name AND u.password=u\_password AND u.status=1;  
END;

* * *

invoker

当定义为INVOKER时，只要执行者有执行权限，就可以成功执行。

示例：

CREATE DEFINER=`dev`@`%` PROCEDURE `p_user_login`(IN u\_name VARCHAR(25), IN u\_password VARCHAR(100))  
SQL SECURITY INVOKER  
BEGIN  
SELECT u.id, u.name, u.tid, u.status, u.is\_report FROM v\_user u WHERE u.name=u\_name AND u.password=u\_password AND u.status=1;  
END;

## 函数

可以用 navicat 工具修改定义者，或者用下面语句直接在 MySQL 中修改：

```shell
mysql> UPDATE mysql.proc set definer='root@%' where db='db_name';
```


## Reference

- [MySQL视图定义者和安全性definer/invoker 的一次问题](https://blog.csdn.net/chen3888015/article/details/86678278)

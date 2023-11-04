---
keywords:
- 
title: "mysql慢查询分析"
date: 2023-11-04T10:54:39+08:00
lastmod: 2023-11-04T10:54:39+08:00
description: ""
draft: true 
author: starifly
hideToc: false
enableToc: true
enableTocContent: false
tocFolding: false
tocLevels: ["h2", "h3", "h4"]
categories: [mysql]
tags: [mysql]
slug: mysql-slow-query-analysis
img:
---

mysql没有那么多性能视图可以查询SQL执行情况，但是也提供mysqldumpslow命令来解析日志来获取数据库SQL语句执行情况，为运维、开发人员提供了获取需要具体优化SQL语句的一个方法。

## mysqldumpslow

### 主要参数：

A: -s, 是sort的意思，表示按照何种方式排序，c、t、l、r分别是按照记录次数、时间、查询时间、返回的记录数来排序，ac、at、al、ar，表示相应的倒序；
B: -t, 是top n的意思，即为返回前面多少条的数据；
C: -g, 是grep的意思，后边可以写一个正则匹配模式，大小写不敏感的。

### 用法举例：

- 耗时最长的10个 SQL

`mysqldumpslow -t 10 slow.log`

- 执行字数最多的10个sql语句

`mysqldumpslow -s c -t 10 slow.log`

- 获取返回条数最多的10个sql

`mysqldumpslow -s r -t 10 slow.log`

## 监控慢查询

查询大于20秒的查询：

```sql
select * from information_schema.processlist where command='query' and time > 20 ORDER BY time desc;
```

列出所有的慢查询：

```sql
SELECT * FROM information_schema.SLOW_QUERY_LOG ORDER BY start_time DESC;`
```

通过查询到的慢查询语句再结合explain关键字可以进一步的优化。

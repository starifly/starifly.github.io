---
title: mysql批量修改 字段/表/数据库 字符集和排序规则
author: starifly
date: 2019-10-07T21:26:46+08:00
categories: [mysql]
tags: [mysql,数据库,字符集]
draft: true
slug: mysql-batch-changes-field-table-database-character-set-and-collation
---

有时候我们需要修改数据库的字符编码，比如之前的数据库、表和字段（类型为CHAR、VARCHAR、TEXT的列，可以指定字符集/字符序）的字符集字符序为utf8，现在需要把它们全部修改为utf8mb4，如果一个一个手工去修改肯定费时费力，简便的方法是批量修改，但是批量修改会遇到一些坑，这个后面会讲到。

修正的顺序是 表字段 > 表 > 数据库。

## 表字段修复

```shell
#改变字段数据
SELECT TABLE_SCHEMA '数据库',TABLE_NAME '表',COLUMN_NAME '字段',CHARACTER_SET_NAME '原字符集',COLLATION_NAME '原排序规则',CONCAT('ALTER TABLE ', TABLE_SCHEMA,'.',TABLE_NAME, ' MODIFY COLUMN ',COLUMN_NAME,' ',COLUMN_TYPE,' CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;') '修正SQL'
FROM information_schema.`COLUMNS` 
WHERE COLLATION_NAME RLIKE 'latin1';
```

可以利用 navicat 工具执行上面的查询语句。latin1 是我demo的模糊匹配排序规则，这里需要替换为你数据库中需要替换的字段的排序规则，utf8mb4设置的是替换的字符集，utf8mb4_general_ci设置的是替换的排序规则，可以换为需要修正为什么标准。

把修正的SQL复制出来运行，字段标准就修复了。

## 表修复

```shell
#改变表
SELECT TABLE_SCHEMA '数据库',TABLE_NAME '表',TABLE_COLLATION '原排序规则',CONCAT('ALTER TABLE ',TABLE_SCHEMA,'.', TABLE_NAME, ' COLLATE=utf8mb4_general_ci;') '修正SQL'
FROM information_schema.`TABLES`
WHERE TABLE_COLLATION RLIKE 'latin1';
```

表修复只需要设置排序规则，字符集会自动设置到正确的标准，替换的话，跟上面一样。

## 数据库修复

```shell
#修改数据库
SELECT SCHEMA_NAME '数据库',DEFAULT_CHARACTER_SET_NAME '原字符集',DEFAULT_COLLATION_NAME '原排序规则',CONCAT('ALTER DATABASE ',SCHEMA_NAME,' CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;') '修正SQL'
FROM information_schema.`SCHEMATA`
WHERE DEFAULT_CHARACTER_SET_NAME RLIKE 'utf8';
```

数据库修复，跟字段修复一样，需要设置字符集和排序规则，这里utf8是模糊匹配需要修正的数据库的字符集，也可以用DEFAULT_COLLATION_NAME 排序规则来筛选，这些都很灵活。

如果需要选定到数据库，就用SCHEMA_NAME 匹配一下就行了。

## 踩到的坑

前面提到批量修改会碰到一些坑，下面说一说。

> 1.从 utf8 修改为 utf8mb4，因为所占的字节会变大，可能导致超过一行的限制长度，这时需要灵活将某些字段的类型进行修改，比如从 varchar 修改为 tinytext。  
> 2.有些字段会有关键字冲突，这时需要将此字段反引号起来。  
> 3.批量修改字段的字符集会把视图也带进来，但是视图是虚表，所以无需修改，注释即可。

## Reference

- [mysql 批量修改 表字段/表/数据库 字符集和排序规则](https://www.cnblogs.com/-renyu/p/10776020.html)
- [再见乱码：5分钟读懂MySQL字符集设置](https://www.cnblogs.com/chyingp/p/mysql-character-set-collation.html)

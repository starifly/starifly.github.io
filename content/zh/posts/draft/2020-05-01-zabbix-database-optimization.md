---
title: zabbix数据库优化
author: starifly
date: 2020-05-01T18:07:43+08:00
lastmod: 2020-05-01T18:07:43+08:00
categories: [zabbix]
tags: [zabbix,mysql]
draft: true
slug: zabbix-database-optimization
---

## mysql设置

在[mysqld]下增加下面配置 `innodb_file_per_table=1` ，参考 [处理MySQL的ibdata1文件过大问题](https://www.cnblogs.com/beyang/p/10681761.html)

## 关闭Housekeeper

ZABBIX 2.2后所有的housekeeper配置在zabbix的web端更改，"Administration" -> "General" ，选择"Housekeeping" ，确保history和trends栏的"Enable internal housekeeping"的对勾去掉。

## 启用分区表

1.从[https://www.zabbix.org/wiki/File:Partitioning.zip]下载Partitioning.zip

2.`unzip Partitioning.zip`

3.导入解压后的sql文件

`mysql -uzabbix -p zabbix < *.sql`

4.查看数据库的存储过程

```
MariaDB [(none)]> show procedure status;
+--------+---------------------------+-----------+----------------+---------------------+---------------------+---------------+---------+----------------------+----------------------+--------------------+
| Db     | Name                      | Type      | Definer        | Modified            | Created             | Security_type | Comment | character_set_client | collation_connection | Database Collation |
+--------+---------------------------+-----------+----------------+---------------------+---------------------+---------------+---------+----------------------+----------------------+--------------------+
| zabbix | partition_create          | PROCEDURE | root@localhost | 2019-12-11 18:56:37 | 2019-12-11 18:56:37 | DEFINER       |         | utf8                 | utf8_general_ci      | utf8_general_ci    |
| zabbix | partition_delete            | PROCEDURE | root@localhost | 2019-12-11 18:56:56 | 2019-12-11 18:56:56 | DEFINER       |         | utf8                 | utf8_general_ci      | utf8_general_ci    |
| zabbix | partition_maintenance     | PROCEDURE | root@localhost | 2019-12-11 18:57:08 | 2019-12-11 18:57:08 | DEFINER       |         | utf8                 | utf8_general_ci      | utf8_general_ci    |
| zabbix | partition_maintenance_all | PROCEDURE | root@localhost | 2019-12-11 19:39:48 | 2019-12-11 19:39:48 | DEFINER       |         | utf8                 | utf8_general_ci      | utf8_general_ci    |
| zabbix | partition_verify          | PROCEDURE | root@localhost | 2019-12-11 20:13:44 | 2019-12-11 20:13:44 | DEFINER       |         | utf8                 | utf8_general_ci      | utf8_general_ci    |
+--------+---------------------------+-----------+----------------+---------------------+---------------------+---------------+---------+----------------------+----------------------+--------------------+
5 rows in set (0.01 sec)
```

5.查看看过程的创建代码

```
MariaDB [(none)]> use zabbix;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A
 
Database changed
MariaDB [zabbix]> show create procedure  partition_maintenance_all;
+---------------------------+----------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------+----------------------+--------------------+
| Procedure                 | sql_mode | Create Procedure                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  | character_set_client | collation_connection | Database Collation |
+---------------------------+----------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------+----------------------+--------------------+
| partition_maintenance_all |          | CREATE DEFINER=`root`@`localhost` PROCEDURE `partition_maintenance_all`(SCHEMA_NAME VARCHAR(32))
BEGIN
                CALL partition_maintenance(SCHEMA_NAME, 'history', 28, 24, 14);
                CALL partition_maintenance(SCHEMA_NAME, 'history_log', 28, 24, 14);
                CALL partition_maintenance(SCHEMA_NAME, 'history_str', 28, 24, 14);
                CALL partition_maintenance(SCHEMA_NAME, 'history_text', 28, 24, 14);
                CALL partition_maintenance(SCHEMA_NAME, 'history_uint', 28, 24, 14);
                CALL partition_maintenance(SCHEMA_NAME, 'trends', 730, 24, 14);
                CALL partition_maintenance(SCHEMA_NAME, 'trends_uint', 730, 24, 14);
END | utf8                 | utf8_general_ci      | utf8_general_ci    |
+---------------------------+----------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+----------------------+----------------------+--------------------+
1 row in set (0.00 sec)
```

6.自动执行分表脚本
```
[root@bogon Partitioning]# cat partition_maintenance_all.sh
#!/bin/bash

echo 'Partitioned table maintenance tasks begin at '`date +%Y%m%d%H%M%S`'.' > /tmp/partition_maintenance_all_mail.tmp
echo '' >> /tmp/partition_maintenance_all_mail.tmp

#mysql -uzabbix -pzabbix -D zabbix -p -e 'CALL partition_maintenance_all("zabbix");' >> /tmp/partition_maintenance_all_mail.tmp 2>&1
mysql -uzabbix -pzabbix zabbix -e "CALL partition_maintenance_all('zabbix');" >> /tmp/partition_maintenance_all_mail.tmp 2>&1


echo '' >> /tmp/partition_maintenance_all_mail.tmp
echo 'Partitioned table maintenance tasks finish at '`date +%Y%m%d%H%M%S`'.' >> /tmp/partition_maintenance_all_mail.tmp

#cat /tmp/partition_maintenance_all_mail.tmp | mailx -r 'ITmonitor@didichuxing.com' -s 'Partitioned table maintenance' gaoyuebruce@didiglobal.com

#CURL_DATA="{
#  \"content\": \""`sed ":a;N;s/\n/<br \/>/;s/\t/ /g;ta" /tmp/partition_maintenance_all_mail.tmp`"\",
#  \"sender\": \"erpmonitor@didiglobal.com\",
#  \"subject\": \"Partitioned table maintenance\",
#  \"tos\": [
#    \"gaoyuebruce@didiglobal.com\"
#  ]
#}
#"
#
#curl -X POST --header "Content-Type: application/json" --header "Accept: */*" -d "$CURL_DATA" "http://10.89.139.46:8088/api/v2/notification/email/send?token=【token值】"
```

`sh partition_maintenance_all.sh`

7.创建定时执行任务

```
[root@bogon Partitioning]# crontab -l
30 1 * * *  /usr/bin/sh /root/Partitioning/partition_maintenance_all.sh
```

## Reference

- [Zabbix优化： 数据库表分区](https://blog.51cto.com/lihuipeng/1561221)
- [zabbix 启用分区表](https://blog.csdn.net/h106140873/article/details/103499075)
- [Table partitioning](https://www.zabbix.org/wiki/Docs/howto/mysql_partition#partition_create)
- [处理MySQL的ibdata1文件过大问题](https://www.cnblogs.com/beyang/p/10681761.html)
- [zabbix4.0的基础性能调优和测试小工具](https://blog.51cto.com/11555417/2348416)

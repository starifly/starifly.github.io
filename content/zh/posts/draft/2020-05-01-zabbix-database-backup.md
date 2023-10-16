---
title: zabbix数据库备份
author: starifly
date: 2020-05-01T17:02:08+08:00
lastmod: 2020-05-01T17:02:08+08:00
categories: [zabbix]
tags: [zabbix,mysql]
draft: true
slug: zabbix-database-backup
---

```
[root@bogon backup]# cat backup.sh 
#!/bin/bash
DBPATH=/etc/zabbix/backup

mysqldump -uzabbix -pzabbix --quick --single-transaction zabbix |gzip >$DBPATH/`date +%F_%H%M%S`_zabbix.sql.gz
if [ $? -eq 0 ];then
  echo "`date` Backup zabbix database successd!" >> /tmp/zabbix_backup.tmp
else
  echo "`date` Backup zabbix database failed!" >> /tmp/zabbix_backup.tmp
fi
find $DBPATH -mtime +1 -type d|grep zabbix|xargs rm -rf
```

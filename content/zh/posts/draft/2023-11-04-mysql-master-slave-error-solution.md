---
keywords:
- 
title: "mysql主从同步报错解决办法"
date: 2023-11-04T10:45:17+08:00
lastmod: 2023-11-04T10:45:17+08:00
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
slug: mysql-master-slave-error-solution
img:
---

这种情况通常是由于从库与主库数据不一致导致的，要解决这类问题，通常需要在从库执行反向操作，比如删掉这些错误增改的数据，让主从数据恢复到之前一致状态。当然也可以先忽略错误，继续同步后面的数据。

slave检查同步报错

```sh
mysql> show slave status\G
ERROR 4031 (HY000): The client was disconnected by the server because of inactivity. See wait_timeout and interactive_timeout for configuring this behavior.
No connection. Trying to reconnect...
Enter password:
Connection id:    55303
Current database: test

*************************** 1. row ***************************
               Slave_IO_State: Waiting for source to send event
                  Master_Host: 192.168.5.31
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: binlog.000002
          Read_Master_Log_Pos: 4808094
               Relay_Log_File: node2-relay-bin.000005
                Relay_Log_Pos: 808011
        Relay_Master_Log_File: binlog.000002
             Slave_IO_Running: Yes
            Slave_SQL_Running: No
              Replicate_Do_DB:
          Replicate_Ignore_DB:
           Replicate_Do_Table:
       Replicate_Ignore_Table:
      Replicate_Wild_Do_Table:
  Replicate_Wild_Ignore_Table:
                   Last_Errno: 1032
                   Last_Error: Coordinator stopped because there were error(s) in the worker(s). The most recent failure being: Worker 1 failed executing transaction '1c6ca2b1-cd3b-11ed-85fa-000c29c08c81:1544' at master log binlog.000002, end_log_pos 886311. See error log and/or performance_schema.replication_applier_status_by_worker table for more details about this failure or others, if any.
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 885925
              Relay_Log_Space: 4731241
              Until_Condition: None
               Until_Log_File:
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File:
           Master_SSL_CA_Path:
              Master_SSL_Cert:
            Master_SSL_Cipher:
               Master_SSL_Key:
        Seconds_Behind_Master: NULL
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error:
               Last_SQL_Errno: 1032
               Last_SQL_Error: Coordinator stopped because there were error(s) in the worker(s). The most recent failure being: Worker 1 failed executing transaction '1c6ca2b1-cd3b-11ed-85fa-000c29c08c81:1544' at master log binlog.000002, end_log_pos 886311. See error log and/or performance_schema.replication_applier_status_by_worker table for more details about this failure or others, if any.
  Replicate_Ignore_Server_Ids:
             Master_Server_Id: 100
                  Master_UUID: 1c6ca2b1-cd3b-11ed-85fa-000c29c08c81
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State:
           Master_Retry_Count: 86400
                  Master_Bind:
      Last_IO_Error_Timestamp:
     Last_SQL_Error_Timestamp: 230620 01:30:06
               Master_SSL_Crl:
           Master_SSL_Crlpath:
           Retrieved_Gtid_Set: 1c6ca2b1-cd3b-11ed-85fa-000c29c08c81:8-8966
            Executed_Gtid_Set: 1c6ca2b1-cd3b-11ed-85fa-000c29c08c81:1-1543,
95f54340-e02c-11ed-881b-000c29da1ba4:1-201
                Auto_Position: 1
         Replicate_Rewrite_DB:
                 Channel_Name:
           Master_TLS_Version:
       Master_public_key_path:
        Get_master_public_key: 0
            Network_Namespace:
1 row in set, 1 warning (2.46 sec)
```

根据Last_SQL_Error中的报错信息获取具体出错的SQL，通过主库binlog查询

```sh
root@node1:/var/lib/mysql# mysqlbinlog -v --base64-output=DECODE-ROWS binlog.000002|grep 'end_log_pos 886311' -C 6
SET TIMESTAMP=1687195806/*!*/;
BEGIN
/*!*/;
# at 886098
#230620  1:30:06 server id 100  end_log_pos 886167 CRC32 0xa59cb99a 	Table_map: `table1`.`value` mapped to number 148
# at 886167
#230620  1:30:06 server id 100  end_log_pos 886311 CRC32 0x056b3958 	Update_rows: table id 148 flags: STMT_END_F
### UPDATE `table1`.`value`
### WHERE
###   @1=3322113
### SET
###   @1=3322114
### UPDATE `table1`.`value`
```

可以看到end_log_pos 886311 对应的事务就是更新table1表的value值，这一行数据在slave端无法更新，所以先停掉两台数据库的slave，再把slave上table1表中的value字段的值修改为“3322113”，最后通过`start slave`重新启动slave进程即可解决。



## Reference

- [MySQL主从复制 - 常见故障与处理办法 - 墨天轮](https://www.modb.pro/db/554625)

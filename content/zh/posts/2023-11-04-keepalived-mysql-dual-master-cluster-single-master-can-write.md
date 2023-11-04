---
keywords:
- 
title: "keepalived+mysql双主集群单主可写"
date: 2023-11-04T11:11:17+08:00
lastmod: 2023-11-04T11:11:17+08:00
description: "mysql双主高可用介绍及改造方案"
draft: false
author: starifly
hideToc: false
enableToc: true
enableTocContent: false
tocFolding: false
tocLevels: ["h2", "h3", "h4"]
categories: [mysql,keepalived]
tags: [mysql,keepalived]
slug: keepalived-mysql-dual-master-cluster-single-master-can-write
img:
---

## keepalived高可用介绍及改造方案

![](/images/keealived_01.png)

![](/images/keealived_02.png)

![](/images/keealived_03.png)

![](/images/keealived_04.png)

![](/images/keealived_05.png)

![](/images/keealived_06.png)

![](/images/keealived_07.png)

![](/images/keealived_08.png)

![](/images/keealived_09.png)

![](/images/keealived_10.png)

![](/images/keealived_11.png)

![](/images/keealived_12.png)

![](/images/keealived_13.png)

## 具体配置-passwd方式

### 192.168.5.31

```cfg
# keepalived.conf

! Configuration File for keepalived

global_defs {
   router_id db01
}

vrrp_sync_group G1 {
 group {
  VI_1
}
}

vrrp_script check_run {
    script "/etc/keepalived/mysql_check.sh"
    interval 10
    fall 3
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens32
    virtual_router_id 51
    priority 100
    nopreempt
    advert_int 3
    authentication {
        auth_type PASS
        auth_pass admin#2023
    }
    track_interface {
          ens32
     }

    unicast_src_ip  192.168.5.31
    unicast_peer {
      192.168.5.32
    }
    track_script {
        check_run
    }
    virtual_ipaddress {
        192.168.5.190
    }
    notify_master "/etc/keepalived/change_to_master.sh"
    notify_backup "/etc/keepalived/change_to_backup.sh"
    notify_fault "/etc/keepalived/change_to_backup.sh"
}
```

```sh
# mysql_check.sh

#!/bin/bash

###############################################
#修改此处配置
#用户需要权限
#grant REPLICATION CLIENT  on *.* to mon_user@'%' ;
export HOME='/root'
master_ip='192.168.5.31'
master_port='3306'
master_mycom='mysql'
rep_ip='192.168.5.32'
rep_port='3306'
#记录日志位置
log_time=`date  '+%Y%m%d'`
check_log='/etc/keepalived/keepalived_'${log_time}'.log'
check_log_error='/var/log/messages'
find /etc/keepalived/  -name "keepalived_*.log" -mtime +7|xargs rm -rf
#################################################

logging_log()
#input:$1 输入警告级别 $2 输入警告日志字符串
{
 dt_time=`date  '+%Y-%m-%d-%H:%M:%S'`
 echo "keepalive logging ${dt_time} $1:$2" >> ${check_log}
}

logging_log_error()
#input:$1 输入警告级别 $2 输入警告日志字符串
{
 dt_time=`date  '+%Y-%m-%d-%H:%M:%S'`
 echo "keepalive logging ${dt_time} $1:$2" >> ${check_log_error}
}


check_pid()
#input:$1检查的进程关键字比如mysqld
#return:1 检测成功，但是需要注意有多个mysqld进程(物理服务器跑了多个实例)  0 检测失败，没有mysqld进程
{
  mysqld_cnt=`ps -ef|grep -w $1|grep -v 'grep'|wc -l`

  if test ${mysqld_cnt} -ge 2 ;then
     logging_log 'warn' 'more than one mysqld process!'
     return 1  #由于mysqld检测出有多个那么，只是告警而不做任何动作
  fi

  if test ${mysqld_cnt} -lt 1 ;then
     logging_log 'err' 'mysqld process is crash!!'
     logging_log_error 'err' 'mysqld process is crash!!'
     return 0 #检测mysqld已经不存在则直接进入从库检测流程
  fi

  logging_log 'info' 'one mysqld process check sucess.'
  return 1 #检测成功，进入语句检测
}


check_sta()
#input:NULL
#return:0 失败，主库测试语句不能执行 1 成功，主库测试语句可以执行
{
  #将stderr 写入到日志文件中，比如连接失败
  #${master_mycom} -h ${master_ip} -P ${master_port}  -u ${master_user} -p${master_password} -e 'select now()'
  ${master_mycom} -uroot -p123456 -h ${master_ip} -e 'select now()'
  if [ $? -eq 0 ]; then
    logging_log 'info' 'master:test select now() sucess.'
    return 1 #检测成功，测试语句能够执行
  else
    logging_log 'err' master:'test select now() fail!!'
    logging_log_error 'err' master:'test select now() fail!!'
    return 0 #检测失败，测试语句不能够执行
  fi

}


check_rep_apply_all()
#input:NULL
#return value:0 失败，检测不能切换 1 成功，检测可以切换
{
  #先测试从库是否可以连接
  ${master_mycom}  -uroot -p123456 -h ${rep_ip}  -e 'select now()'

  if [ $? -eq 0 ]; then
    logging_log 'info' 'rep:test select now() sucess.' #检测成功，测试语句能够执行,继续执行
  else
    logging_log 'err' 'rep:test select now() fail!!'
    logging_log_error 'err' 'rep:test select now() fail!!'
    return 0 #检测失败，测试语句不能够执行
  fi

  #不放到一条语句进行检测，多次检测可以提高检测的准确性
  Rep_Master_ip=`${master_mycom}  -uroot -p123456 -h ${rep_ip} -e "show slave status\G" | grep -w Master_Host | awk -F": " '{print $2}'`
  Rep_Master_port=`${master_mycom}  -uroot -p123456 -h ${rep_ip} -e "show slave status\G" | grep -w Master_Port | awk -F": " '{print $2}'`
  Slave_IO_Running=`${master_mycom}  -uroot -p123456 -h ${rep_ip} -e "show slave status\G" | grep -w Slave_IO_Running | awk -F": " '{print $2}'`
  Master_Log_File=`${master_mycom}  -uroot -p123456 -h ${rep_ip} -e "show slave status\G" | grep -w Master_Log_File | awk -F": " '{print $2}'`
  Relay_Master_Log_File=`${master_mycom}  -uroot -p123456 -h ${rep_ip} -e "show slave status\G" | grep -w Relay_Master_Log_File | awk -F": " '{print $2}'`
  Read_Master_Log_Pos=`${master_mycom}  -uroot -p123456 -h ${rep_ip} -e "show slave status\G" | grep -w Read_Master_Log_Pos | awk -F": " '{print $2}'`
  Exec_Master_Log_Pos=`${master_mycom}  -uroot -p123456 -h ${rep_ip} -e "show slave status\G" | grep -w Exec_Master_Log_Pos | awk -F": " '{print $2}'`
  Seconds_Behind_Master=`${master_mycom}  -uroot -p123456 -h ${rep_ip} -e  "show slave status\G" | grep -w Seconds_Behind_Master | awk -F": " '{print $2}'`

  if [ ${Rep_Master_ip} != ${master_ip} ]; then
     logging_log 'err' "rep:this rep ip ${Rep_Master_ip} is not master ip:${master_ip} !!"
     logging_log_error 'err' "rep:this rep ip ${Rep_Master_ip} is not master ip:${master_ip} !!"
     return 0 #检测失败，从库连接主库的ip和填写的主库ip不一致
  fi

  if [ ${Rep_Master_port} != ${master_port} ]; then
     logging_log 'err' "rep:this rep port ${Rep_Master_port} is not master port:${master_port} !!"
     logging_log_error 'err' "rep:this rep port ${Rep_Master_port} is not master port:${master_port} !!"
     return 0 #检测失败，从库连接主库的端口和填写的主库端口不一致
  fi

  if [ ${Master_Log_File} != ${Relay_Master_Log_File} ]; then
     logging_log 'err' "rep:this rep read binlog:${Master_Log_File} is not eq execute binlog:${Relay_Master_Log_File} !!"
     logging_log_error 'err' "rep:this rep read binlog:${Master_Log_File} is not eq execute binlog:${Relay_Master_Log_File} !!"
     return 0 #检测失败，从库读取的binlog文件和执行的binlog文件不一致
  fi

  if [ ${Read_Master_Log_Pos} != ${Exec_Master_Log_Pos} ]; then
     logging_log 'err' "rep:this rep read pos:${Read_Master_Log_Pos} is not eq execute pos:${Exec_Master_Log_Pos} !!"
     logging_log_error 'err' "rep:this rep read pos:${Read_Master_Log_Pos} is not eq execute pos:${Exec_Master_Log_Pos} !!"
     return 0 #检测失败，从库执行的binlog pos和执行的binlog pos不一致
  fi
  sleep 5
  ##是否需要检测IO线程状态，只有在IO线程运行异常的情况下进行切换
  if [ ${Slave_IO_Running} == 'Yes' ]; then
     logging_log 'err' "rep:this rep IO thread is running !!"
     logging_log_error 'err' "rep:this rep IO thread is running !!"
     return 0
  fi

  if [ ${Seconds_Behind_Master} == 'NULL' -o ${Seconds_Behind_Master} == '0' ] ; then #检测成功什么都不做
     echo 'normal'
  else
     logging_log 'err' "rep:this rep Behind Master:${Seconds_Behind_Master}s blocking switch !!"
     logging_log_error 'err' "rep:this rep Behind Master:${Seconds_Behind_Master}s blocking switch !!"
     return 0 #检测失败,延迟不为0
  fi



  logging_log 'info' "rep:check sucess.master ip:${Rep_Master_ip} master port:${Rep_Master_port} io status:${Slave_IO_Running} read_pos:${Master_Log_File} ${Read_Master_Log_Pos} exe_pos:${Relay_Master_Log_File} ${Exec_Master_Log_Pos}  Seconds_Behind_Master:${Seconds_Behind_Master}."
  return 1 #返回检测从库应用状态成功

}



#main 脚本开始


#第一步检查主库状态
#1、检测主库进程是否存在，调用check_pid
#2、检测主库是否可以跑简单的语句，调用check_sta
#如果检测成功说明不需要切换，直接返回 0给 keepdalived
#通常不会进入第二步判断
#第二步检测从库是否可以切换
#如果不能切换，同样返回 0给 keepalived
#如果可以切换，则返回 1给 keepalived

ret=0


echo '---' >> ${check_log}
#第一步检查主库状态

check_pid "mysqld"
if [ $? == '1' ] ;then  #1.mysqld检测成功 2.多个mysqld实例 都返回为1
   check_sta
   if [ $? == '0' ];then #检测sql命令失败
     ret=1 #失败
   else
     ret=0 #成功
   fi
else #检测没有mysqld存在
  ret=1 #失败
fi

#检测主库状态正常，不需要进行切换，返回1给keepalived

if [ $ret == '0' ];then
 echo "主库正常"
 exit 0
fi

#第二步检测从库是否可以切换
#检测主库状态不正常，需要检查从库状态是否满足切换条件

if [ $ret == '1' ];then
   check_rep_apply_all
   if [ $? == '1' ];then #检测从库状态成功
      echo "从库可以切换"
      exit 1
   else
      echo "从库不可以切换"  #检测失败不能切换
      exit 0
   fi
fi
```

```sh
# change_to_master.sh

#!/bin/bash

###############################################
#修改此处配置
#用户需要权限
#查看从库状态需要REPLICATION CLIENT权限
#修改参数需要super权限
export HOME='/root/'
NIC='ens32'
VIP='192.168.5.190'
GATEWAY='192.168.5.1'
ip='192.168.5.31'
port='3306'
mycom='mysql'
#记录日志位置
log_time=`date  '+%Y%m%d'`
check_log='/etc/keepalived/keepalived_'${log_time}'.log'
check_log_error='/var/log/messages'
#循环检测次数
check_loop=10

#循环检测间隔秒数
loop_time=5
#################################################

logging_log()
#input:$1 输入警告级别 $2 输入警告日志字符串
{
 dt_time=`date  '+%Y-%m-%d-%H:%M:%S'`
 echo "keepalive loggong ${dt_time} $1:$2" >> ${check_log}
}


logging_log_error()
#input:$1 输入警告级别 $2 输入警告日志字符串
{
 dt_time=`date  '+%Y-%m-%d-%H:%M:%S'`
 echo "keepalive logging ${dt_time} $1:$2" >> ${check_log_error}
}


check_rep_apply_all()
#input:NULL
#return value:0 失败，检测不能切换 1 成功，检测可以切换
{
  #先测试从库是否可以连接
  ${mycom}  -uroot -p123456 -h ${ip}  -e 'select now()'

  if [ $? -eq 0 ]; then
    logging_log 'info' 'master check:test select now() sucess.' #检测成功，测试语句能够执行,继续执行
  else
    logging_log 'err' 'master check:test select now() fail!!'
    logging_log_error 'err' 'master check:test select now() fail!!'
    return 0 #检测失败，测试语句不能够执行
  fi

  #不放到一条语句进行检测，多次检测可以提高检测的准确性
  Slave_IO_Running=`${mycom}  -uroot -p123456 -h ${ip} -e "show slave status\G" | grep -w Slave_IO_Running | awk -F": " '{print $2}'`
  Master_Log_File=`${mycom} -uroot -p123456 -h ${ip} -e "show slave status\G" | grep -w Master_Log_File | awk -F": " '{print $2}'`
  Relay_Master_Log_File=`${mycom} -uroot -p123456 -h ${ip} -e "show slave status\G" | grep -w Relay_Master_Log_File | awk -F": " '{print $2}'`
  Read_Master_Log_Pos=`${mycom} -uroot -p123456 -h ${ip} -e "show slave status\G" | grep -w Read_Master_Log_Pos | awk -F": " '{print $2}'`
  Exec_Master_Log_Pos=`${mycom} -uroot -p123456 -h ${ip} -e "show slave status\G" | grep -w Exec_Master_Log_Pos | awk -F": " '{print $2}'`
  Seconds_Behind_Master=`${mycom} -uroot -p123456 -h ${ip} -e "show slave status\G" | grep -w Seconds_Behind_Master | awk -F": " '{print $2}'`


  if [ ${Master_Log_File} != ${Relay_Master_Log_File} ]; then
     logging_log 'err' "rep:this rep read binlog:${Master_Log_File} is not eq execute binlog:${Relay_Master_Log_File} !!"
     logging_log_error 'err' "rep:this rep read binlog:${Master_Log_File} is not eq execute binlog:${Relay_Master_Log_File} !!"
     return 0 #检测失败，从库读取的binlog文件和执行的binlog文件不一致
  fi

  if [ ${Read_Master_Log_Pos} != ${Exec_Master_Log_Pos} ]; then
     logging_log 'err' "master check:this rep read pos:${Read_Master_Log_Pos} is not eq execute pos:${Exec_Master_Log_Pos} !!"
     logging_log_error 'err' "master check:this rep read pos:${Read_Master_Log_Pos} is not eq execute pos:${Exec_Master_Log_Pos} !!"
     return 0 #检测失败，从库执行的binlog pos和执行的binlog pos不一致
  fi

  ##是否需要检测IO线程状态，只有在IO线程运行异常的情况下进行切换
  if [ ${Slave_IO_Running} == 'Yes' ]; then
     logging_log 'err' "master check:this rep IO thread is running !!"
     logging_log_error 'err' "master check:this rep IO thread is running !!"
     return 0
  fi

  if [ ${Seconds_Behind_Master} == 'NULL' -o ${Seconds_Behind_Master} == '0' ] ; then #检测成功什么都不做
     echo "1"
  else
     logging_log 'err' "rep:this rep Behind Master:${Seconds_Behind_Master}s blocking switch !!"
     logging_log_error 'err' "rep:this rep Behind Master:${Seconds_Behind_Master}s blocking switch !!"
     return 0 #检测失败,延迟不为0
  fi

  logging_log 'info' "master check:check sucess.io status:${Slave_IO_Running} read_pos:${Master_Log_File} ${Read_Master_Log_Pos} exe_pos:${Relay_Master_Log_File} ${Exec_Master_Log_Pos} Seconds_Behind_Master:${Seconds_Behind_Master}."
  return 1 #返回从库检测状态正常

}

with_out_readonly()
#input:NULL
#returnvalue:NULL
{
  logging_log 'warn' "set read_only=off"
  logging_log_error 'warn' "set read_only=off"
  ${mycom} -uroot -p123456 -h ${ip} -e "set global read_only=0"
  ${mycom} -uroot -p123456 -h ${ip} -e "set global super_read_only=0"
}


#main 脚本开始
#用于检测切换后是否可以打开read only选项
echo '---master-check---' >>${check_log}


i=1

while [ $i -le ${check_loop} ] ##循环次数
do
 check_rep_apply_all
 if [ $? -eq 1 ];then
    with_out_readonly #打开readonly选项
    break #跳出循环
 else
   echo "--check fail,loop $i times--" >>${check_log} #输出检测失败次数
   sleep ${loop_time} #睡眠时间
 fi
 let i+=1 #判断值+1
done

logging_log 'info' 'start to arping on change_to_master'
logging_log_error 'info' 'start to arping on change_to_master'
/sbin/arping -I $NIC -c 5 -s $VIP $GATEWAY
```

```sh
# change_to_backup.sh

#!/bin/bash
export HOME='/root/'
NIC='ens32'
VIP='192.168.5.190'
GATEWAY='192.168.5.1'
ip='192.168.5.31'
port='3306'
mycom='mysql'
log_time=`date  '+%Y%m%d'`
check_log='/etc/keepalived/keepalived_'${log_time}'.log'
check_log_error='/var/log/messages'

logging_log()
#input:$1 输入警告级别 $2 输入警告日志字符串
{
 dt_time=`date  '+%Y-%m-%d-%H:%M:%S'`
 echo "keepalive logging ${dt_time} $1:$2" >> ${check_log}
}

logging_log_error()
#input:$1 输入警告级别 $2 输入警告日志字符串
{
 dt_time=`date  '+%Y-%m-%d-%H:%M:%S'`
 echo "keepalive logging ${dt_time} $1:$2" >> ${check_log_error}
}

with_readonly()
#input:NULL
#returnvalue:NULL
{
  logging_log 'warn' "set read_only=on"
  logging_log_error 'warn' "set read_only=on"
  ${mycom} -uroot -p123456 -h ${ip} -e "set global read_only=1"
  ${mycom} -uroot -p123456 -h ${ip} -e "set global super_read_only=1"
}


#main 脚本开始
#用于检测切换后是否可以打开read only选项
echo '---backup-set---' >>${check_log}

with_readonly #设置readonly

logging_log 'info' 'start to arping on change_to_backup'
logging_log_error 'info' 'start to arping on change_to_backup'
/sbin/arping -I $NIC -c 5 -s $VIP $GATEWAY
```

### 192.168.5.32

```cfg
# keepalived.conf

! Configuration File for keepalived

global_defs {
   router_id db02
}

vrrp_sync_group G1 {
 group {
  VI_1
}
}

vrrp_script check_run {
    script "/etc/keepalived/mysql_check.sh"
    interval 10
    fall 3
}

vrrp_instance VI_1 {
    state BACKUP
    interface ens32
    virtual_router_id 51
    priority 90
    nopreempt
    advert_int 3
    authentication {
        auth_type PASS
        auth_pass admin#2023
    }
    track_interface {
          ens32
     }
    unicast_src_ip  192.168.5.32
    unicast_peer {
      192.168.5.31
    }
    track_script {
        check_run
    }
    virtual_ipaddress {
        192.168.5.190
    }
    notify_master "/etc/keepalived/change_to_master.sh"
    notify_backup "/etc/keepalived/change_to_backup.sh"
    notify_fault "/etc/keepalived/change_to_backup.sh"
}
```

```sh
# mysql_check.sh

#!/bin/bash

###############################################
#修改此处配置
#用户需要权限
#grant REPLICATION CLIENT  on *.* to mon_user@'%' ;
export HOME='/root'
master_ip='192.168.5.32'
master_port='3306'
master_mycom='mysql'
rep_ip='192.168.5.31'
rep_port='3306'
#记录日志位置
log_time=`date  '+%Y%m%d'`
check_log='/etc/keepalived/keepalived_'${log_time}'.log'
check_log_error='/var/log/messages'
find /etc/keepalived/  -name "keepalived_*.log" -mtime +7|xargs rm -rf
#################################################

logging_log()
#input:$1 输入警告级别 $2 输入警告日志字符串
{
 dt_time=`date  '+%Y-%m-%d-%H:%M:%S'`
 echo "keepalive logging ${dt_time} $1:$2" >> ${check_log}
}

logging_log_error()
#input:$1 输入警告级别 $2 输入警告日志字符串
{
 dt_time=`date  '+%Y-%m-%d-%H:%M:%S'`
 echo "keepalive logging ${dt_time} $1:$2" >> ${check_log_error}
}


check_pid()
#input:$1检查的进程关键字比如mysqld
#return:1 检测成功，但是需要注意有多个mysqld进程(物理服务器跑了多个实例)  0 检测失败，没有mysqld进程
{
  mysqld_cnt=`ps -ef|grep -w $1|grep -v 'grep'|wc -l`

  if test ${mysqld_cnt} -ge 2 ;then
     logging_log 'warn' 'more than one mysqld process!'
     return 1  #由于mysqld检测出有多个那么，只是告警而不做任何动作
  fi

  if test ${mysqld_cnt} -lt 1 ;then
     logging_log 'err' 'mysqld process is crash!!'
     logging_log_error 'err' 'mysqld process is crash!!'
     return 0 #检测mysqld已经不存在则直接进入从库检测流程
  fi

  logging_log 'info' 'one mysqld process check sucess.'
  return 1 #检测成功，进入语句检测
}


check_sta()
#input:NULL
#return:0 失败，主库测试语句不能执行 1 成功，主库测试语句可以执行
{
  #将stderr 写入到日志文件中，比如连接失败
  #${master_mycom} -h ${master_ip} -P ${master_port}  -u ${master_user} -p${master_password} -e 'select now()'
  ${master_mycom} -uroot -p123456 -h ${master_ip} -e 'select now()'
  if [ $? -eq 0 ]; then
    logging_log 'info' 'master:test select now() sucess.'
    return 1 #检测成功，测试语句能够执行
  else
    logging_log 'err' master:'test select now() fail!!'
    logging_log_error 'err' master:'test select now() fail!!'
    return 0 #检测失败，测试语句不能够执行
  fi

}


check_rep_apply_all()
#input:NULL
#return value:0 失败，检测不能切换 1 成功，检测可以切换
{
  #先测试从库是否可以连接
  ${master_mycom}  -uroot -p123456 -h ${rep_ip}  -e 'select now()'

  if [ $? -eq 0 ]; then
    logging_log 'info' 'rep:test select now() sucess.' #检测成功，测试语句能够执行,继续执行
  else
    logging_log 'err' 'rep:test select now() fail!!'
    logging_log_error 'err' 'rep:test select now() fail!!'
    return 0 #检测失败，测试语句不能够执行
  fi

  #不放到一条语句进行检测，多次检测可以提高检测的准确性
  Rep_Master_ip=`${master_mycom}  -uroot -p123456 -h ${rep_ip} -e "show slave status\G" | grep -w Master_Host | awk -F": " '{print $2}'`
  Rep_Master_port=`${master_mycom}  -uroot -p123456 -h ${rep_ip} -e "show slave status\G" | grep -w Master_Port | awk -F": " '{print $2}'`
  Slave_IO_Running=`${master_mycom}  -uroot -p123456 -h ${rep_ip} -e "show slave status\G" | grep -w Slave_IO_Running | awk -F": " '{print $2}'`
  Master_Log_File=`${master_mycom}  -uroot -p123456 -h ${rep_ip} -e "show slave status\G" | grep -w Master_Log_File | awk -F": " '{print $2}'`
  Relay_Master_Log_File=`${master_mycom}  -uroot -p123456 -h ${rep_ip} -e "show slave status\G" | grep -w Relay_Master_Log_File | awk -F": " '{print $2}'`
  Read_Master_Log_Pos=`${master_mycom}  -uroot -p123456 -h ${rep_ip} -e "show slave status\G" | grep -w Read_Master_Log_Pos | awk -F": " '{print $2}'`
  Exec_Master_Log_Pos=`${master_mycom}  -uroot -p123456 -h ${rep_ip} -e "show slave status\G" | grep -w Exec_Master_Log_Pos | awk -F": " '{print $2}'`
  Seconds_Behind_Master=`${master_mycom}  -uroot -p123456 -h ${rep_ip} -e  "show slave status\G" | grep -w Seconds_Behind_Master | awk -F": " '{print $2}'`

  if [ ${Rep_Master_ip} != ${master_ip} ]; then
     logging_log 'err' "rep:this rep ip ${Rep_Master_ip} is not master ip:${master_ip} !!"
     logging_log_error 'err' "rep:this rep ip ${Rep_Master_ip} is not master ip:${master_ip} !!"
     return 0 #检测失败，从库连接主库的ip和填写的主库ip不一致
  fi

  if [ ${Rep_Master_port} != ${master_port} ]; then
     logging_log 'err' "rep:this rep port ${Rep_Master_port} is not master port:${master_port} !!"
     logging_log_error 'err' "rep:this rep port ${Rep_Master_port} is not master port:${master_port} !!"
     return 0 #检测失败，从库连接主库的端口和填写的主库端口不一致
  fi

  if [ ${Master_Log_File} != ${Relay_Master_Log_File} ]; then
     logging_log 'err' "rep:this rep read binlog:${Master_Log_File} is not eq execute binlog:${Relay_Master_Log_File} !!"
     logging_log_error 'err' "rep:this rep read binlog:${Master_Log_File} is not eq execute binlog:${Relay_Master_Log_File} !!"
     return 0 #检测失败，从库读取的binlog文件和执行的binlog文件不一致
  fi

  if [ ${Read_Master_Log_Pos} != ${Exec_Master_Log_Pos} ]; then
     logging_log 'err' "rep:this rep read pos:${Read_Master_Log_Pos} is not eq execute pos:${Exec_Master_Log_Pos} !!"
     logging_log_error 'err' "rep:this rep read pos:${Read_Master_Log_Pos} is not eq execute pos:${Exec_Master_Log_Pos} !!"
     return 0 #检测失败，从库执行的binlog pos和执行的binlog pos不一致
  fi
  sleep 5
  ##是否需要检测IO线程状态，只有在IO线程运行异常的情况下进行切换
  if [ ${Slave_IO_Running} == 'Yes' ]; then
     logging_log 'err' "rep:this rep IO thread is running !!"
     logging_log_error 'err' "rep:this rep IO thread is running !!"
     return 0
  fi

  if [ ${Seconds_Behind_Master} == 'NULL' -o ${Seconds_Behind_Master} == '0' ] ; then #检测成功什么都不做
     echo 'normal'
  else
     logging_log 'err' "rep:this rep Behind Master:${Seconds_Behind_Master}s blocking switch !!"
     logging_log_error 'err' "rep:this rep Behind Master:${Seconds_Behind_Master}s blocking switch !!"
     return 0 #检测失败,延迟不为0
  fi



  logging_log 'info' "rep:check sucess.master ip:${Rep_Master_ip} master port:${Rep_Master_port} io status:${Slave_IO_Running} read_pos:${Master_Log_File} ${Read_Master_Log_Pos} exe_pos:${Relay_Master_Log_File} ${Exec_Master_Log_Pos}  Seconds_Behind_Master:${Seconds_Behind_Master}."
  return 1 #返回检测从库应用状态成功

}



#main 脚本开始


#第一步检查主库状态
#1、检测主库进程是否存在，调用check_pid
#2、检测主库是否可以跑简单的语句，调用check_sta
#如果检测成功说明不需要切换，直接返回 0给 keepdalived
#通常不会进入第二步判断
#第二步检测从库是否可以切换
#如果不能切换，同样返回 0给 keepalived
#如果可以切换，则返回 1给 keepalived

ret=0


echo '---' >> ${check_log}
#第一步检查主库状态

check_pid "mysqld"
if [ $? == '1' ] ;then  #1.mysqld检测成功 2.多个mysqld实例 都返回为1
   check_sta
   if [ $? == '0' ];then #检测sql命令失败
     ret=1 #失败
   else
     ret=0 #成功
   fi
else #检测没有mysqld存在
  ret=1 #失败
fi

#检测主库状态正常，不需要进行切换，返回1给keepalived

if [ $ret == '0' ];then
 echo "主库正常"
 exit 0
fi

#第二步检测从库是否可以切换
#检测主库状态不正常，需要检查从库状态是否满足切换条件

if [ $ret == '1' ];then
   check_rep_apply_all
   if [ $? == '1' ];then #检测从库状态成功
      echo "从库可以切换"
      exit 1
   else
      echo "从库不可以切换"  #检测失败不能切换
      exit 0
   fi
fi
```

```sh
# change_to_master.sh

#!/bin/bash

###############################################
#修改此处配置
#用户需要权限
#查看从库状态需要REPLICATION CLIENT权限
#修改参数需要super权限
export HOME='/root/'
NIC='ens32'
VIP='192.168.5.190'
GATEWAY='192.168.5.1'
ip='192.168.5.32'
port='3306'
mycom='mysql'
#记录日志位置
log_time=`date  '+%Y%m%d'`
check_log='/etc/keepalived/keepalived_'${log_time}'.log'
check_log_error='/var/log/messages'
#循环检测次数
check_loop=10

#循环检测间隔秒数
loop_time=5
#################################################

logging_log()
#input:$1 输入警告级别 $2 输入警告日志字符串
{
 dt_time=`date  '+%Y-%m-%d-%H:%M:%S'`
 echo "keepalive loggong ${dt_time} $1:$2" >> ${check_log}
}


logging_log_error()
#input:$1 输入警告级别 $2 输入警告日志字符串
{
 dt_time=`date  '+%Y-%m-%d-%H:%M:%S'`
 echo "keepalive logging ${dt_time} $1:$2" >> ${check_log_error}
}


check_rep_apply_all()
#input:NULL
#return value:0 失败，检测不能切换 1 成功，检测可以切换
{
  #先测试从库是否可以连接
  ${mycom}  -uroot -p123456 -h ${ip}  -e 'select now()'

  if [ $? -eq 0 ]; then
    logging_log 'info' 'master check:test select now() sucess.' #检测成功，测试语句能够执行,继续执行
  else
    logging_log 'err' 'master check:test select now() fail!!'
    logging_log_error 'err' 'master check:test select now() fail!!'
    return 0 #检测失败，测试语句不能够执行
  fi

  #不放到一条语句进行检测，多次检测可以提高检测的准确性
  Slave_IO_Running=`${mycom}  -uroot -p123456 -h ${ip} -e "show slave status\G" | grep -w Slave_IO_Running | awk -F": " '{print $2}'`
  Master_Log_File=`${mycom} -uroot -p123456 -h ${ip} -e "show slave status\G" | grep -w Master_Log_File | awk -F": " '{print $2}'`
  Relay_Master_Log_File=`${mycom} -uroot -p123456 -h ${ip} -e "show slave status\G" | grep -w Relay_Master_Log_File | awk -F": " '{print $2}'`
  Read_Master_Log_Pos=`${mycom} -uroot -p123456 -h ${ip} -e "show slave status\G" | grep -w Read_Master_Log_Pos | awk -F": " '{print $2}'`
  Exec_Master_Log_Pos=`${mycom} -uroot -p123456 -h ${ip} -e "show slave status\G" | grep -w Exec_Master_Log_Pos | awk -F": " '{print $2}'`
  Seconds_Behind_Master=`${mycom} -uroot -p123456 -h ${ip} -e "show slave status\G" | grep -w Seconds_Behind_Master | awk -F": " '{print $2}'`


  if [ ${Master_Log_File} != ${Relay_Master_Log_File} ]; then
     logging_log 'err' "rep:this rep read binlog:${Master_Log_File} is not eq execute binlog:${Relay_Master_Log_File} !!"
     logging_log_error 'err' "rep:this rep read binlog:${Master_Log_File} is not eq execute binlog:${Relay_Master_Log_File} !!"
     return 0 #检测失败，从库读取的binlog文件和执行的binlog文件不一致
  fi

  if [ ${Read_Master_Log_Pos} != ${Exec_Master_Log_Pos} ]; then
     logging_log 'err' "master check:this rep read pos:${Read_Master_Log_Pos} is not eq execute pos:${Exec_Master_Log_Pos} !!"
     logging_log_error 'err' "master check:this rep read pos:${Read_Master_Log_Pos} is not eq execute pos:${Exec_Master_Log_Pos} !!"
     return 0 #检测失败，从库执行的binlog pos和执行的binlog pos不一致
  fi

  ##是否需要检测IO线程状态，只有在IO线程运行异常的情况下进行切换
  if [ ${Slave_IO_Running} == 'Yes' ]; then
     logging_log 'err' "master check:this rep IO thread is running !!"
     logging_log_error 'err' "master check:this rep IO thread is running !!"
     return 0
  fi

  if [ ${Seconds_Behind_Master} == 'NULL' -o ${Seconds_Behind_Master} == '0' ] ; then #检测成功什么都不做
     echo "1"
  else
     logging_log 'err' "rep:this rep Behind Master:${Seconds_Behind_Master}s blocking switch !!"
     logging_log_error 'err' "rep:this rep Behind Master:${Seconds_Behind_Master}s blocking switch !!"
     return 0 #检测失败,延迟不为0
  fi

  logging_log 'info' "master check:check sucess.io status:${Slave_IO_Running} read_pos:${Master_Log_File} ${Read_Master_Log_Pos} exe_pos:${Relay_Master_Log_File} ${Exec_Master_Log_Pos} Seconds_Behind_Master:${Seconds_Behind_Master}."
  return 1 #返回从库检测状态正常

}

with_out_readonly()
#input:NULL
#returnvalue:NULL
{
  logging_log 'warn' "set read_only=off"
  logging_log_error 'warn' "set read_only=off"
  ${mycom} -uroot -p123456 -h ${ip} -e "set global read_only=0"
  ${mycom} -uroot -p123456 -h ${ip} -e "set global super_read_only=0"
}


#main 脚本开始
#用于检测切换后是否可以打开read only选项
echo '---master-check---' >>${check_log}


i=1

while [ $i -le ${check_loop} ] ##循环次数
do
 check_rep_apply_all
 if [ $? -eq 1 ];then
    with_out_readonly #打开readonly选项
    break #跳出循环
 else
   echo "--check fail,loop $i times--" >>${check_log} #输出检测失败次数
   sleep ${loop_time} #睡眠时间
 fi
 let i+=1 #判断值+1
done

logging_log 'info' 'start to arping on change_to_master'
logging_log_error 'info' 'start to arping on change_to_master'
/sbin/arping -I $NIC -c 5 -s $VIP $GATEWAY
```

```sh
# change_to_backup.sh

#!/bin/bash
export HOME='/root/'
NIC='ens32'
VIP='192.168.5.190'
GATEWAY='192.168.5.1'
ip='192.168.5.32'
port='3306'
mycom='mysql'
log_time=`date  '+%Y%m%d'`
check_log='/etc/keepalived/keepalived_'${log_time}'.log'
check_log_error='/var/log/messages'

logging_log()
#input:$1 输入警告级别 $2 输入警告日志字符串
{
 dt_time=`date  '+%Y-%m-%d-%H:%M:%S'`
 echo "keepalive logging ${dt_time} $1:$2" >> ${check_log}
}

logging_log_error()
#input:$1 输入警告级别 $2 输入警告日志字符串
{
 dt_time=`date  '+%Y-%m-%d-%H:%M:%S'`
 echo "keepalive logging ${dt_time} $1:$2" >> ${check_log_error}
}

with_readonly()
#input:NULL
#returnvalue:NULL
{
  logging_log 'warn' "set read_only=on"
  logging_log_error 'warn' "set read_only=on"
  ${mycom} -uroot -p123456 -h ${ip} -e "set global read_only=1"
  ${mycom} -uroot -p123456 -h ${ip} -e "set global super_read_only=1"
}


#main 脚本开始
#用于检测切换后是否可以打开read only选项
echo '---backup-set---' >>${check_log}

with_readonly #设置readonly

logging_log 'info' 'start to arping on change_to_backup'
logging_log_error 'info' 'start to arping on change_to_backup'
/sbin/arping -I $NIC -c 5 -s $VIP $GATEWAY
#!/bin/bash
export HOME='/root/'
NIC='ens32'
VIP='192.168.5.190'
GATEWAY='192.168.5.1'
ip='192.168.5.32'
port='3306'
mycom='mysql'
log_time=`date  '+%Y%m%d'`
check_log='/etc/keepalived/keepalived_'${log_time}'.log'
check_log_error='/var/log/messages'

logging_log()
#input:$1 输入警告级别 $2 输入警告日志字符串
{
 dt_time=`date  '+%Y-%m-%d-%H:%M:%S'`
 echo "keepalive logging ${dt_time} $1:$2" >> ${check_log}
}

logging_log_error()
#input:$1 输入警告级别 $2 输入警告日志字符串
{
 dt_time=`date  '+%Y-%m-%d-%H:%M:%S'`
 echo "keepalive logging ${dt_time} $1:$2" >> ${check_log_error}
}

with_readonly()
#input:NULL
#returnvalue:NULL
{
  logging_log 'warn' "set read_only=on"
  logging_log_error 'warn' "set read_only=on"
  ${mycom} -uroot -p123456 -h ${ip} -e "set global read_only=1"
  ${mycom} -uroot -p123456 -h ${ip} -e "set global super_read_only=1"
}


#main 脚本开始
#用于检测切换后是否可以打开read only选项
echo '---backup-set---' >>${check_log}

with_readonly #设置readonly

logging_log 'info' 'start to arping on change_to_backup'
logging_log_error 'info' 'start to arping on change_to_backup'
/sbin/arping -I $NIC -c 5 -s $VIP $GATEWAY
```

## 具体配置-login-path方式

```sh
# mysql_check.sh

#!/bin/bash

###############################################
#修改此处配置
#用户需要权限
#grant REPLICATION CLIENT  on *.* to mon_user@'%' ;
export HOME='/root'
master_ip='170.100.132.3'
master_port='3306'
master_mycom='/mysql/app/bin/mysql'
rep_ip='170.100.132.4'
rep_port='3306'
#记录日志位置
log_time=`date  '+%Y%m%d'`
check_log='/etc/keepalived/keepalived_'${log_time}'.log'
check_log_error='/var/log/messages'
find /etc/keepalived/  -name "keepalived_*.log" -mtime +7|xargs rm -rf
#################################################

logging_log()
#input:$1 输入警告级别 $2 输入警告日志字符串
{
 dt_time=`date  '+%Y-%m-%d-%H:%M:%S'`
 echo "keepalive logging ${dt_time} $1:$2" >> ${check_log}
}

logging_log_error()
#input:$1 输入警告级别 $2 输入警告日志字符串
{
 dt_time=`date  '+%Y-%m-%d-%H:%M:%S'`
 echo "keepalive logging ${dt_time} $1:$2" >> ${check_log_error}
}


check_pid()
#input:$1检查的进程关键字比如mysqld
#return:1 检测成功，但是需要注意有多个mysqld进程(物理服务器跑了多个实例)  0 检测失败，没有mysqld进程
{
  mysqld_cnt=`ps -ef|grep -w $1|grep -v 'grep'|wc -l`

  if test ${mysqld_cnt} -ge 2 ;then
     logging_log 'warn' 'more than one mysqld process!'
     return 1  #由于mysqld检测出有多个那么，只是告警而不做任何动作
  fi

  if test ${mysqld_cnt} -lt 1 ;then
     logging_log 'err' 'mysqld process is crash!!'
     logging_log_error 'err' 'mysqld process is crash!!'
     return 0 #检测mysqld已经不存在则直接进入从库检测流程
  fi

  logging_log 'info' 'one mysqld process check sucess.'
  return 1 #检测成功，进入语句检测
}


check_sta()
#input:NULL
#return:0 失败，主库测试语句不能执行 1 成功，主库测试语句可以执行
{
  #将stderr 写入到日志文件中，比如连接失败
  #${master_mycom} -h ${master_ip} -P ${master_port}  -u ${master_user} -p${master_password} -e 'select now()'
  ${master_mycom} --login-path=local_login -e 'select now()'
  if [ $? -eq 0 ]; then
    logging_log 'info' 'master:test select now() sucess.'
    return 1 #检测成功，测试语句能够执行
  else
    logging_log 'err' master:'test select now() fail!!'
    logging_log_error 'err' master:'test select now() fail!!'
    return 0 #检测失败，测试语句不能够执行
  fi

}


check_rep_apply_all()
#input:NULL
#return value:0 失败，检测不能切换 1 成功，检测可以切换
{
  #先测试从库是否可以连接
  ${master_mycom}  --login-path=sync_login  -e 'select now()'

  if [ $? -eq 0 ]; then
    logging_log 'info' 'rep:test select now() sucess.' #检测成功，测试语句能够执行,继续执行
  else
    logging_log 'err' 'rep:test select now() fail!!'
    logging_log_error 'err' 'rep:test select now() fail!!'
    return 0 #检测失败，测试语句不能够执行
  fi

  #不放到一条语句进行检测，多次检测可以提高检测的准确性
  Rep_Master_ip=`${master_mycom}  --login-path=sync_login -e "show slave status\G" | grep -w Master_Host | awk -F": " '{print $2}'`
  Rep_Master_port=`${master_mycom}  --login-path=sync_login -e "show slave status\G" | grep -w Master_Port | awk -F": " '{print $2}'`
  Slave_IO_Running=`${master_mycom}  --login-path=sync_login -e "show slave status\G" | grep -w Slave_IO_Running | awk -F": " '{print $2}'`
  Master_Log_File=`${master_mycom}  --login-path=sync_login -e "show slave status\G" | grep -w Master_Log_File | awk -F": " '{print $2}'`
  Relay_Master_Log_File=`${master_mycom}  --login-path=sync_login -e "show slave status\G" | grep -w Relay_Master_Log_File | awk -F": " '{print $2}'`
  Read_Master_Log_Pos=`${master_mycom}  --login-path=sync_login -e "show slave status\G" | grep -w Read_Master_Log_Pos | awk -F": " '{print $2}'`
  Exec_Master_Log_Pos=`${master_mycom}  --login-path=sync_login -e "show slave status\G" | grep -w Exec_Master_Log_Pos | awk -F": " '{print $2}'`
  Seconds_Behind_Master=`${master_mycom}  --login-path=sync_login -e  "show slave status\G" | grep -w Seconds_Behind_Master | awk -F": " '{print $2}'`

  if [ ${Rep_Master_ip} != ${master_ip} ]; then
     logging_log 'err' "rep:this rep ip ${Rep_Master_ip} is not master ip:${master_ip} !!"
     logging_log_error 'err' "rep:this rep ip ${Rep_Master_ip} is not master ip:${master_ip} !!"
     return 0 #检测失败，从库连接主库的ip和填写的主库ip不一致
  fi

  if [ ${Rep_Master_port} != ${master_port} ]; then
     logging_log 'err' "rep:this rep port ${Rep_Master_port} is not master port:${master_port} !!"
     logging_log_error 'err' "rep:this rep port ${Rep_Master_port} is not master port:${master_port} !!"
     return 0 #检测失败，从库连接主库的端口和填写的主库端口不一致
  fi

  if [ ${Master_Log_File} != ${Relay_Master_Log_File} ]; then
     logging_log 'err' "rep:this rep read binlog:${Master_Log_File} is not eq execute binlog:${Relay_Master_Log_File} !!"
     logging_log_error 'err' "rep:this rep read binlog:${Master_Log_File} is not eq execute binlog:${Relay_Master_Log_File} !!"
     return 0 #检测失败，从库读取的binlog文件和执行的binlog文件不一致
  fi

  if [ ${Read_Master_Log_Pos} != ${Exec_Master_Log_Pos} ]; then
     logging_log 'err' "rep:this rep read pos:${Read_Master_Log_Pos} is not eq execute pos:${Exec_Master_Log_Pos} !!"
     logging_log_error 'err' "rep:this rep read pos:${Read_Master_Log_Pos} is not eq execute pos:${Exec_Master_Log_Pos} !!"
     return 0 #检测失败，从库执行的binlog pos和执行的binlog pos不一致
  fi
  sleep 5
  ##是否需要检测IO线程状态，只有在IO线程运行异常的情况下进行切换
  if [ ${Slave_IO_Running} == 'Yes' ]; then
     logging_log 'err' "rep:this rep IO thread is running !!"
     logging_log_error 'err' "rep:this rep IO thread is running !!"
     return 0
  fi

  if [ ${Seconds_Behind_Master} == 'NULL' -o ${Seconds_Behind_Master} == '0' ] ; then #检测成功什么都不做
     echo 'normal'
  else
     logging_log 'err' "rep:this rep Behind Master:${Seconds_Behind_Master}s blocking switch !!"
     logging_log_error 'err' "rep:this rep Behind Master:${Seconds_Behind_Master}s blocking switch !!"
     return 0 #检测失败,延迟不为0
  fi



  logging_log 'info' "rep:check sucess.master ip:${Rep_Master_ip} master port:${Rep_Master_port} io status:${Slave_IO_Running} read_pos:${Master_Log_File} ${Read_Master_Log_Pos} exe_pos:${Relay_Master_Log_File} ${Exec_Master_Log_Pos}  Seconds_Behind_Master:${Seconds_Behind_Master}."
  return 1 #返回检测从库应用状态成功

}



#main 脚本开始


#第一步检查主库状态
#1、检测主库进程是否存在，调用check_pid
#2、检测主库是否可以跑简单的语句，调用check_sta
#如果检测成功说明不需要切换，直接返回 0给 keepdalived
#通常不会进入第二步判断
#第二步检测从库是否可以切换
#如果不能切换，同样返回 0给 keepalived
#如果可以切换，则返回 1给 keepalived

ret=0


echo '---' >> ${check_log}
#第一步检查主库状态

check_pid "mysqld"
if [ $? == '1' ] ;then  #1.mysqld检测成功 2.多个mysqld实例 都返回为1
   check_sta
   if [ $? == '0' ];then #检测sql命令失败
     ret=1 #失败
   else
     ret=0 #成功
   fi
else #检测没有mysqld存在
  ret=1 #失败
fi

#检测主库状态正常，不需要进行切换，返回1给keepalived

if [ $ret == '0' ];then
 echo "主库正常"
 exit 0
fi

#第二步检测从库是否可以切换
#检测主库状态不正常，需要检查从库状态是否满足切换条件

if [ $ret == '1' ];then
   check_rep_apply_all
   if [ $? == '1' ];then #检测从库状态成功
      echo "从库可以切换"
      exit 1
   else
      echo "从库不可以切换"  #检测失败不能切换
      exit 0
   fi
fi
```

```sh
# change_to_master.sh

#!/bin/bash

###############################################
#修改此处配置
#用户需要权限
#查看从库状态需要REPLICATION CLIENT权限
#修改参数需要super权限
export HOME='/root/'
NIC='ens192'
VIP='170.100.132.5'
GATEWAY='170.100.132.254'
ip='170.100.132.3'
port='3306'
mycom='/mysql/app/bin/mysql'
#记录日志位置
log_time=`date  '+%Y%m%d'`
check_log='/etc/keepalived/keepalived_'${log_time}'.log'
check_log_error='/var/log/messages'
#循环检测次数
check_loop=10

#循环检测间隔秒数
loop_time=5
#################################################

logging_log()
#input:$1 输入警告级别 $2 输入警告日志字符串
{
 dt_time=`date  '+%Y-%m-%d-%H:%M:%S'`
 echo "keepalive loggong ${dt_time} $1:$2" >> ${check_log}
}


logging_log_error()
#input:$1 输入警告级别 $2 输入警告日志字符串
{
 dt_time=`date  '+%Y-%m-%d-%H:%M:%S'`
 echo "keepalive logging ${dt_time} $1:$2" >> ${check_log_error}
}


check_rep_apply_all()
#input:NULL
#return value:0 失败，检测不能切换 1 成功，检测可以切换
{
  #先测试从库是否可以连接
  ${mycom}  --login-path=local_login  -e 'select now()'

  if [ $? -eq 0 ]; then
    logging_log 'info' 'master check:test select now() sucess.' #检测成功，测试语句能够执行,继续执行
  else
    logging_log 'err' 'master check:test select now() fail!!'
    logging_log_error 'err' 'master check:test select now() fail!!'
    return 0 #检测失败，测试语句不能够执行
  fi

  #不放到一条语句进行检测，多次检测可以提高检测的准确性
  Slave_IO_Running=`${mycom}  --login-path=local_login -e "show slave status\G" | grep -w Slave_IO_Running | awk -F": " '{print $2}'`
  Master_Log_File=`${mycom} --login-path=local_login -e "show slave status\G" | grep -w Master_Log_File | awk -F": " '{print $2}'`
  Relay_Master_Log_File=`${mycom} --login-path=local_login -e "show slave status\G" | grep -w Relay_Master_Log_File | awk -F": " '{print $2}'`
  Read_Master_Log_Pos=`${mycom} --login-path=local_login -e "show slave status\G" | grep -w Read_Master_Log_Pos | awk -F": " '{print $2}'`
  Exec_Master_Log_Pos=`${mycom} --login-path=local_login -e "show slave status\G" | grep -w Exec_Master_Log_Pos | awk -F": " '{print $2}'`
  Seconds_Behind_Master=`${mycom} --login-path=local_login -e "show slave status\G" | grep -w Seconds_Behind_Master | awk -F": " '{print $2}'`


  if [ ${Master_Log_File} != ${Relay_Master_Log_File} ]; then
     logging_log 'err' "rep:this rep read binlog:${Master_Log_File} is not eq execute binlog:${Relay_Master_Log_File} !!"
     logging_log_error 'err' "rep:this rep read binlog:${Master_Log_File} is not eq execute binlog:${Relay_Master_Log_File} !!"
     return 0 #检测失败，从库读取的binlog文件和执行的binlog文件不一致
  fi

  if [ ${Read_Master_Log_Pos} != ${Exec_Master_Log_Pos} ]; then
     logging_log 'err' "master check:this rep read pos:${Read_Master_Log_Pos} is not eq execute pos:${Exec_Master_Log_Pos} !!"
     logging_log_error 'err' "master check:this rep read pos:${Read_Master_Log_Pos} is not eq execute pos:${Exec_Master_Log_Pos} !!"
     return 0 #检测失败，从库执行的binlog pos和执行的binlog pos不一致
  fi

  ##是否需要检测IO线程状态，只有在IO线程运行异常的情况下进行切换
  if [ ${Slave_IO_Running} == 'Yes' ]; then
     logging_log 'err' "master check:this rep IO thread is running !!"
     logging_log_error 'err' "master check:this rep IO thread is running !!"
     return 0
  fi

  if [ ${Seconds_Behind_Master} == 'NULL' -o ${Seconds_Behind_Master} == '0' ] ; then #检测成功什么都不做
     echo "1"
  else
     logging_log 'err' "rep:this rep Behind Master:${Seconds_Behind_Master}s blocking switch !!"
     logging_log_error 'err' "rep:this rep Behind Master:${Seconds_Behind_Master}s blocking switch !!"
     return 0 #检测失败,延迟不为0
  fi

  logging_log 'info' "master check:check sucess.io status:${Slave_IO_Running} read_pos:${Master_Log_File} ${Read_Master_Log_Pos} exe_pos:${Relay_Master_Log_File} ${Exec_Master_Log_Pos} Seconds_Behind_Master:${Seconds_Behind_Master}."
  return 1 #返回从库检测状态正常

}

with_out_readonly()
#input:NULL
#returnvalue:NULL
{
  logging_log 'warn' "set read_only=off"
  logging_log_error 'warn' "set read_only=off"
  ${mycom} --login-path=super_login -e "set global read_only=0"
  ${mycom} --login-path=super_login -e "set global super_read_only=0"
}


#main 脚本开始
#用于检测切换后是否可以打开read only选项
echo '---master-check---' >>${check_log}


i=1

while [ $i -le ${check_loop} ] ##循环次数
do
 check_rep_apply_all
 if [ $? -eq 1 ];then
    with_out_readonly #打开readonly选项
    break #跳出循环
 else
   echo "--check fail,loop $i times--" >>${check_log} #输出检测失败次数
   sleep ${loop_time} #睡眠时间
 fi
 let i+=1 #判断值+1
done

logging_log 'info' 'start to arping on change_to_master'
logging_log_error 'info' 'start to arping on change_to_master'
/sbin/arping -I $NIC -c 5 -s $VIP $GATEWAY
```

```sh
# change_to_backup.sh

#!/bin/bash
export HOME='/root/'
NIC='ens192'
VIP='170.100.132.5'
GATEWAY='170.100.132.254'
port='3306'
mycom='/mysql/app/bin/mysql'
log_time=`date  '+%Y%m%d'`
check_log='/etc/keepalived/keepalived_'${log_time}'.log'
check_log_error='/var/log/messages'

logging_log()
#input:$1 输入警告级别 $2 输入警告日志字符串
{
 dt_time=`date  '+%Y-%m-%d-%H:%M:%S'`
 echo "keepalive logging ${dt_time} $1:$2" >> ${check_log}
}

logging_log_error()
#input:$1 输入警告级别 $2 输入警告日志字符串
{
 dt_time=`date  '+%Y-%m-%d-%H:%M:%S'`
 echo "keepalive logging ${dt_time} $1:$2" >> ${check_log_error}
}

with_readonly()
#input:NULL
#returnvalue:NULL
{
  logging_log 'warn' "set read_only=on"
  logging_log_error 'warn' "set read_only=on"
  ${mycom}  --login-path=super_login -e "set global read_only=1"
  ${mycom} --login-path=super_login -e "set global super_read_only=1"
}


#main 脚本开始
#用于检测切换后是否可以打开read only选项
echo '---backup-set---' >>${check_log}

with_readonly #设置readonly

logging_log 'info' 'start to arping on change_to_backup'
logging_log_error 'info' 'start to arping on change_to_backup'
/sbin/arping -I $NIC -c 5 -s $VIP $GATEWAY
```

> 注意：如果两台mysql数据库先于keepalived启动，当keepalived启动后，第一次需要手动去除VIP所在节点数据库的只读属性。

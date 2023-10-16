---
title: Zabbix端口监控之自动发现
author: starifly
date: 2020-04-27T22:42:33+08:00
lastmod: 2020-04-27T22:42:33+08:00
categories: [zabbix]
tags: [zabbix,monitor]
draft: true
slug: auto-discovery-of-zabbix-port-monitoring
---

## 脚本编写

两个脚本，port_alert.sh为端口自发现脚本，port.conf为指定的监控端口号

```
[root@zabbix-agent ~]# cd /etc/zabbix/script/
[root@zabbix-agent script]# more port_alert.sh 
#/bin/bash
CONFIG_FILE=/etc/zabbix/script/port.conf
Check(){
    grep -vE '(^ *#|^$)' ${CONFIG_FILE} | grep -vE '^ *[0-9]+' &> /dev/null
    if [ $? -eq 0 ]
    then
        echo Error: ${CONFIG_FILE} Contains Invalid Port.
        exit 1
    else
        portarray=($(grep -vE '(^ *#|^$)' ${CONFIG_FILE} | grep -E '^ *[0-9]+'))
    fi
}
PortDiscovery(){
    length=${#portarray[@]}
    printf "{\n"
    printf  '\t'"\"data\":["
    for ((i=0;i<$length;i++))
      do
         printf '\n\t\t{'
         printf "\"{#TCP_PORT}\":\"${portarray[$i]}\"}"
         if [ $i -lt $[$length-1] ];then
                    printf ','
         fi
      done
    printf  "\n\t]\n"
    printf "}\n"
}
port(){
    Check
    PortDiscovery
}
port
```

```
[root@zabbix-agent script]# more port.conf
22
80
#  abc 1
#ebc
50
3306  
8080
10050
10051
 3822
21
9100
```

配置文件port.conf每个端口号一行，每行的被监控端口可以有空格，空行和注释行#会被过滤。

## 修改被监控端的zabbix_agent.conf配置文件，新增KEY值port.alert

```
[root@zabbix-agent ~]# view /etc/zabbix/zabbix_agentd.conf
UserParameter=port.alert,/etc/zabbix/script/port_alert.sh
```

重启agent端zabbix服务

## server端测试

```
[root@zabbix-server ~]# zabbix_get -s 172.27.9.65 -k port.alert
{
        "data":[
                {"{#TCP_PORT}":"22"},
                {"{#TCP_PORT}":"80"},
                {"{#TCP_PORT}":"50"},
                {"{#TCP_PORT}":"3306"},
                {"{#TCP_PORT}":"8080"},
                {"{#TCP_PORT}":"10050"},
                {"{#TCP_PORT}":"10051"},
                {"{#TCP_PORT}":"3822"},
                {"{#TCP_PORT}":"21"},
                {"{#TCP_PORT}":"9100"}
        ]
}
```

## 新建模板，创建自动发现规则

新建自动发现规则：键值为 port.alert

自动发现清单中新建监控项原型：键值为 net.tcp.listen[{#TCP_PORT}]

自动发现清单中新建触发器：last()<>1

## Reference

- [Zabbix监控指定端口](https://blog.51cto.com/3241766/2117521)

---
title: Zabbix自定义key
author: starifly
date: 2019-06-07T23:01:23+08:00
categories: [zabbix]
tags: [zabbix]
draft: true
slug: Zabbix-custom-key
---

```shell
vim /etc/zabbix/zabbix_agentd.conf    
```
```conf
AllowRoot=1  
UnsafeUserParameters=1  
Include=/etc/zabbix/zabbix_agentd.d/  
```

```shell
cd /etc/zabbix/zabbix_agentd.d/
vim zabbix_agentd.userparams.conf
```

```conf
UserParameter=check_files,sh /root/zabbix-scripts/check_files.sh  
UserParameter=vfs.files.contents[*],sh /root/zabbix-scripts/file_contents.sh $1
```

```shell
cd /root/zabbix-scripts/
cat check_files.sh
#!/bin/bash

# ls -l /root | wc -l
cd /root
find . -maxdepth 1  ! -name ".*" | wc -l

cat file_contents.sh
#!/bin/bash

cat $1
```



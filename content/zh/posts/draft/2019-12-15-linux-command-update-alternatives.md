---
title: Linux 命令 alternatives和update-alternatives
author: starifly
date: 2019-12-15T22:42:07+08:00
categories: [linux]
tags: [linux,alternatives]
draft: true
slug: linux-command-update-alternatives
---

**alternatives**

```
alternatives version 1.3.13.5.EL4 - Copyright (C) 2001 Red Hat, Inc.
This may be freely redistributed under the terms of the GNU Public License.  

usage: alternatives --install <link> <name> <path> <priority>  
                    [--initscript <service>]  
                    [--slave <link> <name> <path>]*  
       alternatives --remove <name> <path>  
       alternatives --auto <name>  
       alternatives --config <name>  
       alternatives --display <name>  
       alternatives --set <name> <path>  
  
common options: --verbose --test --help --usage --version  
                --altdir <directory> --admindir <directory>
```

**update-alternatives**

```
alternatives version 1.3.13.5.EL4 - Copyright (C) 2001 Red Hat, Inc.  
This may be freely redistributed under the terms of the GNU Public License.  
  
usage: alternatives --install <link> <name> <path> <priority>  
                    [--initscript <service>]  
                    [--slave <link> <name> <path>]*  
       alternatives --remove <name> <path>  
       alternatives --auto <name>  
       alternatives --config <name>  
       alternatives --display <name>  
       alternatives --set <name> <path>  
  
common options: --verbose --test --help --usage --version  
                --altdir <directory> --admindir <directory>
```

**实例**

举个使用例子吧，改变系统bin默认java的指向：

```
安装  
#alternatives --install /usr/bin/java java /home/server/jdk1.6.0_21/bin/java 300  
  
查询  
#alternatives --display java  
  
删除  
#alternatives --remove java  /home/server/jdk1.6.0_21/bin/java  

配置 可以在各个版本之间选择
#alternatives --config java
```

## Reference

- [Linux 命令 alternatives和update-alternatives](https://blog.csdn.net/whatday/article/details/51026642)

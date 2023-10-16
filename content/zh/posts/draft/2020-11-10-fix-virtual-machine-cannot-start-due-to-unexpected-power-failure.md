---
title: 修复意外停电导致虚拟机不能启动的问题
author: starifly
date: 2020-11-10T21:31:35+08:00
lastmod: 2020-11-10T21:31:35+08:00
categories: [linux]
tags: [linux,centos]
draft: true
slug: 2020-11-10-fix-virtual-machine-cannot-start-due-to-unexpected-power-failure
---

虚拟机因为意外断电，重新启动后报错

![](/images/xfs_1.png)

通过查询dm-0是挂载到根目录下的

![](/images/xfs_2.png)

系统是centos7，文件系统时xfs，使用LVM进行的分区，尝试使用xfs_repair /dev/dm-0命令进行修复

![](/images/xfs_3.png)

根据提示使用xfs_repair -L /dev/dm-0再次进行修复，问题得到解决，系统能够正常启动了。但是注意-L参数可能会丢失数据，再修复前一定要备份或者快照系统。

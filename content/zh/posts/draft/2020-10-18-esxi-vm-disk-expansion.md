---
title: exsi linux虚拟机磁盘扩容
author: starifly
date: 2020-10-18T21:58:48+08:00
lastmod: 2020-10-18T21:58:48+08:00
categories: [linux]
tags: [linux,esxi,disk]
draft: true
slug: esxi-vm-disk-expansion
---

创建的 Linux 系统有时随着使用的增加，磁盘使用率会逐渐升高，这时需要添加或者扩展磁盘。

以Linux(CentOS 7.3)为例

## 扩展磁盘

注意：扩容磁盘的方式分为 [添加磁盘]、[扩展磁盘] ; 扩展磁盘需要在此虚拟机停止的状态下进行，同时扩展的数字是扩展后的预期大小，比如之前是50G，希望扩展100G，那么我们应该输入150G。这里我们以扩展磁盘的方式进行。

首先在 VMware 上针对虚拟机进行磁盘扩展。

扩展后，重新启动 linux，使用 `df -kh` 命令发现磁盘目录大小没有变化，而使用 `fdisk -l` 可以确认磁盘空间是否已经扩展。

## 扩展分区

```
[root@localhost ~]# fdisk /dev/sda
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.
Command (m for help): n
Partition type:
   p   primary (2 primary, 0 extended, 2 free)
   e   extended
Select (default p): p
Partition number (3,4, default 3): 
First sector (104857600-314572799, default 104857600): 
Using default value 104857600
Last sector, +sectors or +size{K,M,G} (104857600-314572799, default 314572799): 
Using default value 314572799
Partition 3 of type Linux and of size 100 GiB is set

Command (m for help): t
Partition number (1-3, default 3): 
Hex code (type L to list all codes): L

 0  Empty           24  NEC DOS         81  Minix / old Lin bf  Solaris        
 1  FAT12           27  Hidden NTFS Win 82  Linux swap / So c1  DRDOS/sec (FAT-
 2  XENIX root      39  Plan 9          83  Linux           c4  DRDOS/sec (FAT-
 3  XENIX usr       3c  PartitionMagic  84  OS/2 hidden C:  c6  DRDOS/sec (FAT-
 4  FAT16 <32M      40  Venix 80286     85  Linux extended  c7  Syrinx         
 5  Extended        41  PPC PReP Boot   86  NTFS volume set da  Non-FS data    
 6  FAT16           42  SFS             87  NTFS volume set db  CP/M / CTOS / .
 7  HPFS/NTFS/exFAT 4d  QNX4.x          88  Linux plaintext de  Dell Utility   
 8  AIX             4e  QNX4.x 2nd part 8e  Linux LVM       df  BootIt         
 9  AIX bootable    4f  QNX4.x 3rd part 93  Amoeba          e1  DOS access     
 a  OS/2 Boot Manag 50  OnTrack DM      94  Amoeba BBT      e3  DOS R/O        
 b  W95 FAT32       51  OnTrack DM6 Aux 9f  BSD/OS          e4  SpeedStor      
 c  W95 FAT32 (LBA) 52  CP/M            a0  IBM Thinkpad hi eb  BeOS fs        
 e  W95 FAT16 (LBA) 53  OnTrack DM6 Aux a5  FreeBSD         ee  GPT            
 f  W95 Ext'd (LBA) 54  OnTrackDM6      a6  OpenBSD         ef  EFI (FAT-12/16/
10  OPUS            55  EZ-Drive        a7  NeXTSTEP        f0  Linux/PA-RISC b
11  Hidden FAT12    56  Golden Bow      a8  Darwin UFS      f1  SpeedStor      
12  Compaq diagnost 5c  Priam Edisk     a9  NetBSD          f4  SpeedStor      
14  Hidden FAT16 <3 61  SpeedStor       ab  Darwin boot     f2  DOS secondary  
16  Hidden FAT16    63  GNU HURD or Sys af  HFS / HFS+      fb  VMware VMFS    
17  Hidden HPFS/NTF 64  Novell Netware  b7  BSDI fs         fc  VMware VMKCORE 
18  AST SmartSleep  65  Novell Netware  b8  BSDI swap       fd  Linux raid auto
1b  Hidden W95 FAT3 70  DiskSecure Mult bb  Boot Wizard hid fe  LANstep        
1c  Hidden W95 FAT3 75  PC/IX           be  Solaris boot    ff  BBT            
1e  Hidden W95 FAT1 80  Old Minix      
Hex code (type L to list all codes): 8e            ## linux lvm文件系统类型
Changed type of partition 'Linux' to 'Linux LVM'

Command (m for help): w                                 ##保存退出
The partition table has been altered!
Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
Syncing disks.
```

## 加载分区表

方案一：（推荐）

```
[root@localhost ~]# partprobe
```

执行 `partprobe` 命令用于将磁盘分区表变化信息通知内核，并请求操作系统重新加载分区表，此方法可以不用重启系统；

方案二：

重启虚拟机

## 分区确认

通过 `fdisk` 可以看到已经添加了/dev/sda3

```
[root@localhost ~]# fdisk -l

Disk /dev/sda: 161.1 GB, 161061273600 bytes, 314572800 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x00026862

   Device Boot      Start         End      Blocks   Id  System
   /dev/sda1   *        2048     2099199     1048576   83  Linux
   /dev/sda2         2099200   104857599    51379200   8e  Linux LVM
   /dev/sda3       104857600   314572799   104857600   8e  Linux LVM
```

## 扩展vg

创建PV

```
[root@localhost ~]# pvcreate /dev/sda3
  Physical volume "/dev/sda3" successfully created.
```

使用 `vgs` 查看

```
[root@localhost ~]# vgs
  VG #PV #LV #SN Attr   VSize  VFree
    cl   1   2   0 wz--n- 49.00g 4.00m
```

## 把sda3加入到LVM组中

```
[root@localhost ~]# vgextend cl  /dev/sda3
  Volume group "cl" successfully extended
```

注意：cl 是vg组名称，请根据具体情况填写

```
[root@localhost ~]# vgs
  VG #PV #LV #SN Attr   VSize   VFree  
    cl   2   2   0 wz--n- 148.99g 100.00g
```

## 扩展lv

我们把新扩展的100G全部添加到 cl-root 中

```
[root@localhost ~]# lvextend /dev/mapper/cl-root /dev/sda3
 Size of logical volume cl/root changed from 46.99 GiB (12030 extents) to 146.99 GiB (37629 extents).
 Logical volume cl/root successfully resized.
```

使用 `lvs` 可以看到 cl-root 已经是 150G 了，但是…请继续往下看

```
[root@localhost ~]# lvs
  LV   VG Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  root cl -wi-ao---- 146.99g                                                    
  swap cl -wi-ao----   2.00g  
```

使用 `df -kh` 查看，空间并没有变化，look down…

```
[root@localhost ~]# df -kh
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cl-root   47G  951M   47G   2% /
devtmpfs             910M     0  910M   0% /dev
tmpfs                920M     0  920M   0% /dev/shm
tmpfs                920M  8.5M  912M   1% /run
tmpfs                920M     0  920M   0% /sys/fs/cgroup
```

## xfs在线扩容

```
[root@localhost ~]# xfs_growfs /dev/mapper/cl-root 
meta-data=/dev/mapper/cl-root    isize=512    agcount=4, agsize=3079680 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=12318720, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=6015, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 12318720 to 38532096
```

## 再次确认df状态, 添加的100G空间已经有效

```
[root@localhost ~]# df -kh
Filesystem           Size  Used Avail Use% Mounted on
/dev/mapper/cl-root  147G  951M  147G   1% /
devtmpfs             910M     0  910M   0% /dev
tmpfs                920M     0  920M   0% /dev/shm
tmpfs                920M  8.5M  912M   1% /run
tmpfs                920M     0  920M   0% /sys/fs/cgroup
/dev/sda1           1014M  139M  876M  14% /boot
tmpfs                184M     0  184M   0% /run/user/0
```

## Reference

- [exsi虚拟机磁盘扩容](https://blog.csdn.net/weixin_43946523/article/details/84850205)

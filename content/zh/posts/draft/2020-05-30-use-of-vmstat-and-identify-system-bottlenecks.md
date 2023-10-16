---
title: vmstat 的用法和判别系统瓶颈
author: starifly
date: 2020-05-30T22:26:27+08:00
lastmod: 2020-05-30T22:26:27+08:00
categories: [linux]
tags: [linux,vmstat]
draft: true
slug: use-of-vmstat-and-identify-system-bottlenecks
---

**本文为转载文章。**

vmstat 可以侦测『 CPU / 内存 / 磁碟输入输出状态 』，找出系统瓶颈  
常用的参数是两个数字，第一个参数是采样的时间间隔数，单位是秒，第二个参数是采样的次数。

**基本语法：**

    vmstat [-a] [-n] [-t] [-S unit] [delay [ count]]
    vmstat [-s] [-n] [-S unit]
    vmstat [-m] [-n] [delay [ count]]
    vmstat [-d] [-n] [delay [ count]]
    vmstat [-p disk partition] [-n] [delay [ count]]
    vmstat [-f]
    vmstat [-V]
    

选项与参数：  
\-a ：使用 inactive/active(活跃与否) 取代 buffer/cache 的内存输出资讯；  
\-f ：启动到目前为止，系统复制 (fork) 的程序数；  
\-s ：将一些事件 (启动至目前为止) 导致的内存变化情况列表说明；  
\-S ：后面可以接单位，让显示的数据有单位。例如 K/M 取代 bytes 的容量；  
\-d ：列出磁碟的读写总量统计表  
\-p ：后面列出分割槽，可显示该分割槽的读写总量统计表

例一: 2 秒采集一次，总共采集 10 次

     [root@localhost test]# vmstat 2 10
    procs -----------memory---------- ---swap-- -----io---- --system-- -----cpu-----
     r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
     0  0      0 1664836  55140 689764    0    0    25    10  210  712  9  1 88  1  0   
     0  0      0 1664348  55140 690084    0    0     0     0 1102 3035 12  2 86  0  0   

表示 2 秒采集一次，总共采集 10 次，如果没有第二个参数

例二： 2 秒采集一次，直到按下 ctr+c

    [root@localhost test]# vmstat 2
    procs -----------memory---------- ---swap-- -----io---- --system-- -----cpu-----
     r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
     0  1      0 1705448  55872 689632    0    0    23    10  225  770 10  1 88  1  0   
     2  0      0 1705140  55872 689952    0    0     0    16 1166 3422 13  2 85  0  0   
     3  0      0 1707124  55872 682612    0    0     0     0 1148 3191 13  2 85  0  0   

例三： 2 秒采集一次，结果写入文件，留待以后分析

    [root@localhost test]# vmstat 2 >> /tmp/vmstat &
    

* * *

**字段说明：**

Procs（进程）：

    r: 运行队列中进程数量，这个值也可以判断是否需要增加CPU。（长期大于1）
    b: 等待IO的进程数量
    

Memory（内存）：

    swpd: 使用虚拟内存大小
        注意：如果swpd的值不为0，但是SI，SO的值长期为0，这种情况不会影响系统性能。
    free: 空闲物理内存大小
    buff: 用作缓冲的内存大小
    cache: 用作缓存的内存大小
        注意：如果cache的值大的时候，说明cache处的文件数多，如果频繁访问到的文件都能被cache处，那么磁盘的读IO bi会非常小。
    inact: 非活跃内存大小（当使用-a选项时显示）
    active: 活跃的内存大小（当使用-a选项时显示）
    

Swap：

    si: 每秒从交换区写到内存的大小，由磁盘调入内存
    so: 每秒写入交换区的内存大小，由内存调入磁盘
    
    注意：内存够用的时候，这2个值都是0，如果这2个值长期大于0时，系统性能会受到影响，磁盘IO和CPU资源都会被消耗。有些朋友看到空闲内存（free）很少的或接近于0时，就认为内存不够用了，不能光看这一点，还要结合si和so，如果free很少，但是si和so也很少（大多时候是0），那么不用担心，系统性能这时不会受到影响的。
    

IO：（现在的Linux版本块的大小为1kb）

    bi: 每秒读取的块数
    bo: 每秒写入的块数
    
    注意：随机磁盘读写的时候，这2个值越大（如超出1024k)，能看到CPU在IO等待的值也会越大。
    

SYS：

    in: 每秒中断数，包括时钟中断。
    cs: 每秒上下文切换数。
    
    注意：上面2个值越大，会看到由内核消耗的CPU时间会越大。
    

CPU（以百分比表示）：

    us: 用户进程执行时间百分比(user time)
    
        注意： us的值比较高时，说明用户进程消耗的CPU时间多，但是如果长期超50%的使用，那么我们就该考虑优化程序算法或者进行加速。
    
    sy: 内核系统进程执行时间百分比(system time)
    
        注意：sy的值高时，说明系统内核消耗的CPU资源多，这并不是良性表现，我们应该检查原因。
    
    wa: IO等待时间百分比
    
        注意：wa的值高时，说明IO等待比较严重，这可能由于磁盘大量作随机访问造成，也有可能磁盘出现瓶颈（块操作）。
    
    id: 空闲时间百分比
    

**总结：**  
目前说来，对于服务器监控有用处的度量主要有：  
r（运行队列）  
pi（页导入）  
us（用户CPU）  
sy（系统CPU）  
id（空闲）  
注意：如果r经常大于4 ，且id经常少于40，表示cpu的负荷很重。如果bi，bo 长期不等于0，表示内存不足。

**通过VMSTAT识别CPU瓶颈：**  
r（运行队列）展示了正在执行和等待CPU资源的任务个数。当这个值超过了CPU数目，就会出现CPU瓶颈了。  
CPU 100% 并不能说明什么，Linux总是试图要CPU尽可能的繁忙，使得任务的吞吐量最大化。唯一能够确定CPU瓶颈的还是r（运行队列）的值。

Linux下查看CPU核心数的命令：

    cat /proc/cpuinfo|grep processor|wc -l

解决办法大体几种：  
1\. 最简单的就是增加CPU个数和核数  
2\. 通过调整任务执行时间，如大任务放到系统不繁忙的情况下进行执行，进尔平衡系统任务  
3\. 调整已有任务的优先级

**通过vmstat识别RAM瓶颈：**

首先用free查看RAM的数量：

    [oracle@oracle-db02 ~]$ free
    total       used       free     shared    buffers     cached
    Mem:       2074924    2071112       3812          0      40616    1598656
    -/+ buffers/cache:     431840    1643084
    Swap:      3068404     195804    2872600

当内存的需求大于RAM的数量，服务器启动了虚拟内存机制，通过虚拟内存，可以将RAM段移到SWAP DISK的特殊磁盘段上，这样会出现虚拟内存的页导出和页导入现象，页导出并不能说明RAM瓶颈，虚拟内存系统经常会对内存段进行页导出，但页导入操作就表明了服务器需要更多的内存了， 页导入需要从SWAP DISK上将内存段复制回RAM，导致服务器速度变慢。

解决的办法有几种：  
1\. 最简单的，加大RAM；  
2\. 改小SGA，使得对RAM需求减少；  
3\. 减少RAM的需求。（如：减少PGA）

**详细语法**
--------

    NAME
           vmstat - Report virtual memory statistics
    
    SYNOPSIS
           vmstat [-a] [-n] [-t] [-S unit] [delay [ count]]
           vmstat [-s] [-n] [-S unit]
           vmstat [-m] [-n] [delay [ count]]
           vmstat [-d] [-n] [delay [ count]]
           vmstat [-p disk partition] [-n] [delay [ count]]
           vmstat [-f]
           vmstat [-V]
    
    DESCRIPTION
           vmstat reports information about processes, memory, paging, block IO, traps, and cpu activity.
    
           The  first  report produced gives averages since the last reboot.  Additional reports give information on a sampling period of length
           delay.  The process and memory reports are instantaneous in either case.
    
       Options
           The -a switch displays active/inactive memory, given a 2.5.41 kernel or better.
    
           The -f switch displays the number of forks since boot.  This includes the fork, vfork, and clone system calls, and is  equivalent  to
           the  total  number  of tasks created. Each process is represented by one or more tasks, depending on thread usage.  This display does
           not repeat.
    
           The -t switch adds timestamp to the output.
    
           The -m switch displays slabinfo.
    
           The -n switch causes the header to be displayed only once rather than periodically.
    
           The -s switch displays a table of various event counters and memory statistics. This display does not repeat.
    
           delay is the delay between updates in seconds.  If no delay is specified, only one report is printed with the  average  values  since
           boot.
    
           count is the number of updates.  If no count is specified and delay is defined, count defaults to infinity.
    
           The -d reports disk statistics (2.5.70 or above required)
    
           The -w enlarges field width for big memory sizes
    
           The -p followed by some partition name for detailed statistics (2.5.70 or above required)
    
           The -S followed by k or K or m or M switches outputs between 1000, 1024, 1000000, or 1048576 bytes
           The -V switch results in displaying version information.
    
    FIELD DESCRIPTION FOR VM MODE
       Procs
           r: The number of processes waiting for run time.
           b: The number of processes in uninterruptible sleep.
    
       Memory
           swpd: the amount of virtual memory used.
           free: the amount of idle memory.
           buff: the amount of memory used as buffers.
           cache: the amount of memory used as cache.
           inact: the amount of inactive memory. (-a option)
           active: the amount of active memory. (-a option)
    
       Swap
           si: Amount of memory swapped in from disk (/s).
           so: Amount of memory swapped to disk (/s).
    
       IO
           bi: Blocks received from a block device (blocks/s).
           bo: Blocks sent to a block device (blocks/s).
    
       System
           in: The number of interrupts per second, including the clock.
           cs: The number of context switches per second.
    
       CPU
           These are percentages of total CPU time.
           us: Time spent running non-kernel code. (user time, including nice time)
           sy: Time spent running kernel code. (system time)
           id: Time spent idle. Prior to Linux 2.5.41, this includes IO-wait time.
           wa: Time spent waiting for IO. Prior to Linux 2.5.41, included in idle.
           st: Time stolen from a virtual machine. Prior to Linux 2.6.11, unknown.
    
    FIELD DESCRIPTION FOR DISK MODE
       Reads
           total: Total reads completed successfully
           merged: grouped reads (resulting in one I/O)
           sectors: Sectors read successfully
           ms: milliseconds spent reading
    
       Writes
           total: Total writes completed successfully
           merged: grouped writes (resulting in one I/O)
           sectors: Sectors written successfully
           ms: milliseconds spent writing
    IO
           cur: I/O in progress
           s: seconds spent for I/O
    
    FIELD DESCRIPTION FOR DISK PARTITION MODE
           reads: Total number of reads issued to this partition
           read sectors: Total read sectors for partition
           writes : Total number of writes issued to this partition
           requested writes: Total number of write requests made for partition
    
    FIELD DESCRIPTION FOR SLAB MODE
           cache: Cache name
           num: Number of currently active objects
           total: Total number of available objects
           size: Size of each object
           pages: Number of pages with at least one active object
           totpages: Total number of allocated pages
           pslab: Number of pages per slab
    
    NOTES
           vmstat does not require special permissions.
    
           These reports are intended to help identify system bottlenecks.  Linux vmstat does not count itself as a running process.
    
           All linux blocks are currently 1024 bytes. Old kernels may report blocks as 512 bytes, 2048 bytes, or 4096 bytes.
    
           Since procps 3.1.9, vmstat lets you choose units (k, K, m, M) default is K (1024 bytes) in the default mode
    
           vmstat uses slabinfo 1.1    FIXME

## Reference

- [vmstat 的用法和判别系统瓶颈](https://blog.csdn.net/risen16/article/details/53355417)

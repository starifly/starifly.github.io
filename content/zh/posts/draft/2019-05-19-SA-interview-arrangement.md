---
title: SA Interview Arrangement
author: starifly
date: 2019-05-19T18:34:39+08:00
categories: [interview]
tags: [sa,interview]
draft: true
slug: SA-interview-arrangement
---

### Linux：

统计Apache或nginx日志里访问次数最多的IP - https://blog.51cto.com/461205160/1749851  
chroot命令  
iptables - http://www.zsythink.net/archives/category/%E8%BF%90%E7%BB%B4%E7%9B%B8%E5%85%B3/iptables/  
Linux系统日常管理--监控系统的状态+排查是否正被攻击 - http://os.51cto.com/art/201904/594826.htm  
Linux 技巧：让进程在后台运行更可靠的几种方法 - https://mp.weixin.qq.com/s/Nm1TUCv3JMVwrQmlRqbdsA  
Centos7配置PHP + MySQL + Nginx - https://www.sufaith.com/article/155.html  
VMware安装Centos7超详细过程（图文） - https://blog.csdn.net/babyxue/article/details/80970526  
Linux服务器Cache占用过多内存导致系统内存不足最终java应用程序崩溃解决方案 - https://blog.csdn.net/u014740338/article/details/66975550

---------------------------------------------------------------------------

多进程和多线程各自的优势  
多进程数据共享复杂，同步简单，而多线程数据共享简单，同步复杂  
多进程创建销毁，切换发杂，速度慢，多线程反之  
多进程进程间不会相互影响，一个线程挂掉将导致整个进程挂掉  
进程和线程的概念,以及两者的区别   
进程是程序执行的一个实例，资源分配的最小单位  
线程是CPU调度和分派的基本单位  
进程有独立的地址空间，一个进程崩溃后，在保护模式下不会对其它进程产生影响，而线程只是一个进程中的不同执行路径。线程有自己的堆栈和局部变量，但线程没有单独的地址空间，一个线程死掉就等于整个进程死掉，所以多进程的程序要比多线程的程序健壮，但在进程切换时，耗费资源较大，效率要差一些。但对于一些要求同时进行并且又要共享某些变量的并发操作，只能用线程，不能用进程。

### 运维综合：

Linux运维入门到高级 全套系列 - 下载的书  
马哥全新第30期Linux运维教程 - 百度云收藏  
Linux就业技术指导 - https://www.cnblogs.com/chensiqiqi/category/1233544.html  
运维工程师养成实录：从确立目标到收获offer - https://www.jianshu.com/p/70ce0d942780  
总结一下：运维工程师面试的经历及面试相关问题（续2） - https://blog.51cto.com/ganbing/2059541  
运维咖啡吧 - https://ops-coffee.cn/  
总结一下：运维工程师面试的经历及面试相关问题 - https://blog.csdn.net/Ki8Qzvka6Gz4n450m/article/details/79227843  
2016运维面试问题总结 - https://blog.csdn.net/hxpjava1/article/details/79680877  
运维安全，没那么简单 - https://sq.163yun.com/blog/article/192756820601585664  
开篇：运维、职场之我见 - https://blog.csdn.net/weixin_42404929/article/details/88778117  

### 网络：

TCP连接的TIME_WAIT和CLOSE_WAIT问题 - https://starsl.cn/article/5665671839.html  
网络运维面试----考官会问到的问题？ - https://blog.51cto.com/13445059/2068904  
tcp为什么逼udp可靠

---------------------------------------------------------------------------

udp：dhcp（68和67）、tftp（69）、ntp、snmp（161和162）  
tcp：smtp（25）、pop3（110）、telnet（23）、ssh（22）、ftp（20和21）、http（80）  
udp&tcp：dns（53）、samba（udp:137和138，tcp：139和445）

---------------------------------------------------------------------------

TCP协议和UDP协议的区别是什么  
1.TCP协议是有连接的，有连接的意思是开始传输实际数据之前TCP的客户端和服务器端必须通过三次握手建立连接，会话结束之后也要结束连接。而UDP是无连接的  
2.TCP协议保证数据按序发送，按序到达，提供超时重传来保证可靠性，但是UDP不保证按序到达，甚至不保证到达，只是努力交付，即便是按序发送的序列，也不保证按序送到。  
3.TCP协议所需资源多，TCP首部需20个字节（不算可选项），UDP首部字段只需8个字节。  
4.TCP有流量控制和拥塞控制，UDP没有，网络拥堵不会影响发送端的发送速率  
5.TCP是一对一的连接，而UDP则可以支持一对一，多对多，一对多的通信。  
6.TCP面向的是字节流的服务，UDP面向的是报文的服务。

常见的应用中有哪些是应用TCP协议的，哪些又是应用UDP协议的，为什么它们被如此设计？  
1.多播的信息一定要用udp实现，因为tcp只支持一对一通信。  
2.如果要求快速响应，那么udp听起来比较合适

https://www.cnblogs.com/zmlctt/p/3690998.html

---------------------------------------------------------------------------

三次握手：  
第一次握手(SYN=1, seq=x):  
客户端发送一个 TCP 的 SYN 标志位置1的包，指明客户端打算连接的服务器的端口，以及初始序号 X,保存在包头的序列号(Sequence Number)字段里。  
发送完毕后，客户端进入 SYN_SEND 状态。  
第二次握手(SYN=1, ACK=1, seq=y, ACKnum=x+1):  
服务器发回确认包(ACK)应答。即 SYN 标志位和 ACK 标志位均为1。服务器端选择自己 ISN 序列号，放到 Seq 域里，同时将确认序号(Acknowledgement Number)设置为客户的 ISN 加1，即X+1。 发送完毕后，服务器端进入 SYN_RCVD 状态。  
第三次握手(ACK=1，ACKnum=y+1)：  
客户端再次发送确认包(ACK)，SYN 标志位为0，ACK 标志位为1，并且把服务器发来 ACK 的序号字段+1，放在确定字段中发送给对方，并且在数据段放写ISN的+1  
发送完毕后，客户端进入 ESTABLISHED 状态，当服务器端接收到这个包时，也进入 ESTABLISHED 状态，TCP 握手结束。

四次挥手：  
第一次挥手(FIN=1，seq=x)：  
假设客户端想要关闭连接，客户端发送一个 FIN 标志位置为1的包，表示自己已经没有数据可以发送了，但是仍然可以接受数据。  
发送完毕后，客户端进入 FIN_WAIT_1 状态。  
第二次挥手(ACK=1，ACKnum=x+1)：  
服务器端确认客户端的 FIN 包，发送一个确认包，表明自己接受到了客户端关闭连接的请求，但还没有准备好关闭连接。  
发送完毕后，服务器端进入 CLOSE_WAIT 状态，客户端接收到这个确认包之后，进入 FIN_WAIT_2 状态，等待服务器端关闭连接。  
第三次挥手(FIN=1，seq=y)：  
服务器端准备好关闭连接时，向客户端发送结束连接请求，FIN 置为1。  
发送完毕后，服务器端进入 LAST_ACK 状态，等待来自客户端的最后一个ACK。  
第四次挥手(ACK=1，ACKnum=y+1)：  
客户端接收到来自服务器端的关闭请求，发送一个确认包，并进入 TIME_WAIT状态，等待可能出现的要求重传的 ACK 包。  
务器端接收到这个确认包之后，关闭连接，进入 CLOSED 状态。  
客户端等待了某个固定时间（两个最大段生命周期，2MSL，2 Maximum Segment Lifetime）之后，没有收到服务器端的 ACK ，认为服务器端已经正常关闭连接，于是自己也关闭连接，进入 CLOSED 状态。

https://hit-alibaba.github.io/interview/basic/network/TCP.html  
https://juejin.im/post/5a0444d45188255ea95b66bc

### 数据库：

MySQL数据库备份之逻辑备份和物理备份概述 - https://blog.51cto.com/horizon49/1163797  
MySQL大型分布式集群-https://www.roncoo.com/view/62  
《MySQL运维内参》  
《跟老男孩学Linux运维：MySQL入门与提高实践》  
MySQL数据库运维全套视频教程 阿里巴巴DBA讲授 - https://www.lingxy.com/911.html#  
MySQL性能调优 – 你必须了解的15个重要变量 - https://www.centos.bz/2016/11/mysql-performance-tuning-15-config-item/  
【MySQL】20个经典面试题，全部答对月薪10k+ - https://blog.csdn.net/lixiaogang_theanswer/article/details/74340547  
一主多从宕机从库切换主库继续和从库同步过程 - https://blog.51cto.com/10642812/2070867  
深入学习MySQL事务：ACID特性的实现原理 - https://www.cnblogs.com/kismetv/p/10331633.html  
MySQL高级 之 事务（ACID特性 与 隔离级别） - https://blog.csdn.net/wuseyukui/article/details/73549514  
MySQL 表锁和行锁机制 - https://juejin.im/entry/5a55c7976fb9a01cba42786f  
MySQL中的锁（表锁、行锁） - https://www.cnblogs.com/chenqionghe/p/4845693.html  
MySQL - http://www.zsythink.net/archives/category/%E5%AD%98%E5%82%A8/mysql/  
使用tcpdump抓取数据包，初步分析MySQL 通信协议 - https://mp.weixin.qq.com/s/BZxS1q50I6DXZ_K9eNt1_g  
数据库 | 一次非常有趣的SQL优化经历 - http://database.51cto.com/art/201904/594568.htm  
MySQL主从复制面试之作用和原理 - https://blog.csdn.net/DarkAngel1228/article/details/80004222  
一主多从宕机从库切换主库继续和从库同步过程 - https://blog.51cto.com/10642812/2070867  
Linux运维必会的Mysql企业面试题大全 - http://www.tiejiang.org/20728.html  
校招面试中常见的 MySQL 考察难点和热点 - http://www.yunweipai.com/archives/20620.html  
19条效率至少提高3倍的MySQL技巧 - http://database.51cto.com/art/201905/596233.htm  
Mysql索引优化 - https://segmentfault.com/a/1190000009717352  
SQL高级查询——50句查询(含答案) - https://blog.csdn.net/linxianliang5201314/article/details/6902005  

---------------------------------------------------------------------------

MySQL中myisam与innodb的区别，至少5点  
innodb支持事务，myisam不支持  
innodb支持行级锁，myisam只支持表锁  
innodb支持外键，myisam不支持  
innodb支持mvcc，myisam不支持  
innodb不支持全文索引，myisam支持

---------------------------------------------------------------------------

锁  
表级锁：开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低。  
行级锁：开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。

### DevOps

DevOps面试问题 - https://blog.csdn.net/qq_40907977/article/details/85788745  
面试DevOps团队时可能会被到问哪些问题？ - https://www.itcodemonkey.com/article/4752.html  
持续集成是什么？ - https://wenku.baidu.com/view/4ace0864a58da0116d17499a?fromShare=1&fr=copy&copyfr=copylinkpop  

### 集群：

标杆徐2019中小型企业通用集群架构实践-https://edu.51cto.com/course/15477.html?utm_source=baidu&utm_medium=sem&utm_term=113944911203&utm_content=27311481798&qd=sem

### 工作内容：

个人负责局部的架构，例如：负责监控，Web，负载均衡，上线，支持开发搭建测试环境

### 职业发展规划：

目前而言，还是走技术路线，毕竟当今是互联网时代，技术更新较快，自己的技术深度与广度还不够，必须勤耕技术的深度与广度，努力提升自己的各方技术能力为主。  
如果hr追问，可以回答往云服务、docker、DevOps[、K8S、虚拟化]等方向深度发展。

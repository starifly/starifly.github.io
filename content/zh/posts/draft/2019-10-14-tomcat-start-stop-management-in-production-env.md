---
title: 生产环境tomcat启动停止管理
author: starifly
date: 2019-10-14T17:41:59+08:00
categories: [tomcat,linux]
tags: [tomcat,linux]
draft: true
slug: tomcat-start-stop-management-in-production-env
---

最近在启停 tomcat 的时候总是碰到僵尸进程或者双进程的情况出现，这个跟启动脚本有关。至于有时重启 tomcat 后台日志打印内存泄漏，那就涉及到代码层面的问题，详情参考 [记Tomcat进程stop卡住问题定位处理](https://www.cnblogs.com/Java-Script/p/11091409.html) 这篇文章。

如何避免以上问题出现，用如下 shell 脚本实现：

```shell
#!/bin/bash
# chkconfig: 2345 12 92
#         description: Saves and restores system entropy pool for \
#         higher quality random number generatio

. /etc/init.d/functions

JAVA_HOME=$(dirname $(dirname $(readlink $(readlink $(which javac)))))
PATH=$JAVA_HOME/bin:$PATH
CATALINA_HOME=/data/cctdata/webdata/tomcat
export JAVA_HOME
export CATALINA_HOME

if [ ! -f $CATALINA_HOME/bin/startup.sh ]
       then
            echo "tomcat is not exit.please install."
            exit 1
fi

#start tomcat
function start(){
        if [ `ps -ef |grep java|grep $CATALINA_HOME|grep -v grep|grep -v sh|wc -l` -eq 0 ]
              then
                    /bin/sh $CATALINA_HOME/bin/startup.sh >/dev/null 2>&1
                    [ $? -eq 0 ]&&\
                    sleep 1
                    action "tomcat start." /bin/true
              else
                    action "tomcat had been startted." /bin/true
                    exit 3
        fi
}

#stop tomcat
function stop(){
        if [ `ps -ef |grep java|grep $CATALINA_HOME|grep -v grep|grep -v sh|wc -l` -gt 0  ]
                then
                        PID=`ps -ef |grep java|grep $CATALINA_HOME|grep -v grep|awk '{print $2}'`
                        kill -9 $PID
                        [ $? -eq 0 ]&&\
                        echo "tomcat is stopping..."
                        sleep 1
                        action "tomcat  been stoped." /bin/true
                 else
                        action "tomcat had been stoped." /bin/true
                        exit 4
        fi
}

#restart tomcat
function restart(){
        if [ `ps -ef |grep java |grep $CATALINA_HOME|grep -v grep|grep -v sh|wc -l` -gt 0  ]
               then
                 PID1=`ps -ef |grep java|grep $CATALINA_HOME|grep -v grep|awk '{print $2}'`
                        kill -9 $PID1
                        [ $? -eq 0 ]&&/bin/sh $CATALINA_HOME/bin/startup.sh >/dev/null 2>&1

                        [ $? -eq 0 ]&&echo "tomcat is restarting..."
                        sleep 1
                        action "tomcat is restartted ." /bin/true
                else
                        action "tomcat is not running,please start." /bin/true
                        exit 5
        fi
}

#status tomcat
function status(){
        if [ `ps -ef |grep java |grep $CATALINA_HOME|grep -v grep|wc -l` -gt 0  ]
                then
                     action "tomcat is running."  /bin/true
                else
                      action "tomcat is stopped." /bin/false
                      exit 5
        fi
}

case $1 in
        start)
        start
;;
        stop)
        stop
;;
        restart)
        restart
;;
        status)
        status
;;

        *)
        echo "USAG:start|stop|restart|status"
esac
```

除了自己编写启动脚本外，还可以直接拷贝 bin 目录下 catalina.sh 文件至 /etc/init.d/ ,在文件开头处增加如下内容：

```conf
JAVA_HOME=$(dirname $(dirname $(readlink $(readlink $(which javac)))))
PATH=$JAVA_HOME/bin:$PATH
CATALINA_HOME=/opt/tomcat
export JAVA_HOME
export PATH
export CATALINA_HOME
```

## Reference

- [生产环境中使用脚本实现tomcat start|status|stop|restart](https://www.cnblogs.com/Steward-Xu/p/6710360.html)

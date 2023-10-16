---
title: Jenkins基于docker版本发布和回滚
author: starifly
date: 2023-01-26T16:08:50+08:00
lastmod: 2023-01-26T16:08:50+08:00
categories: [jenkins]
tags: [jenkins,docker]
draft: true
slug: jenkins-releases-and-rolls-back-on-docker
---

docker安装jenkins

`docker run -u root -d --net=host -v jenkins-data:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock jenkinsci/blueocean`

## 新建一个自由风格项目

![](/images/jenkins-01.png)

## 配置参数化构建过程

1. 添加选项参数（控制发布和回滚）

![](/images/jenkins-02.png)

2. 添加字符参数（回滚版本号）

![](/images/jenkins-03.png)

3. 添加选项参数（控制重启）

![](/images/jenkins-04.png)

## 配置构建步骤

这里是从本地发布，需要安装“Publish Over SSH”插件并进入系统配置配置SSH Servers。

1. 发布配置

构建步骤选择“Send files or execute commands over SSh”，“Exec command”处填入如下内容：

```shell
#!/bin/bash
source /etc/profile
case $Status  in
  Deploy)
    echo "Status:$Status"
    docker commit tomcat tomcat:${BUILD_NUMBER}
    echo "Completing!"
    ;;
  Rollback)
      echo "Status:$Status"
      echo "Version:$Version"
      if [ -z $(docker images -q tomcat:$Version) ];then
        exit
      else
        docker rm -f tomcat && docker rmi tomcat   #删除当前镜像
        docker tag tomcat:$Version tomcat           #还原之前的镜像
      fi
      ;;
  *)
  exit
      ;;
esac
```

2. 定期删除旧镜像脚本

构建步骤选择“Send files or execute commands over SSh”，“Exec command”处填入如下内容：

```shell
ReservedNum=5  #保留镜像数
date=$(date "+%Y%m%d-%H%M%S")

ImageNum=$(docker images | grep tomcat | awk '{print $2}' | grep  '^[1-9]*[1-9][0-9]*$' | wc -l)   #当前有几个镜像，即几个备份

while(( $ImageNum > $ReservedNum))
do
    OldImage=$(docker images | grep tomcat | awk '{print $2}' | grep  '^[1-9]*[1-9][0-9]*$' | tail -1)         #获取最旧的那个备份镜像
    echo  $date "Delete Image:"$OldImage
    docker rmi tomcat:${OldImage}
    let "ImageNum--"
done 
```

3. 更新内容发送到远程服务器

构建步骤选择“Send files or execute commands over SSh”,

![](/images/jenkins-05.png)

“Exec command”处填入如下内容：

```shell
if [ “$Status” = “Deploy” ];then
  docker cp /mnt/top/ROOT/ tomcat:/usr/local/tomcat/webapps
  rm -rf /mnt/top/ROOT/
fi
case $Restart  in
  Yes)
    echo "Restart:$Restart"
    case $Status in
      Deploy)
        docker restart tomcat
        echo "Completing!"
        ;;
      Rollback)
        cd /root/package/docker/ && docker-compose up -d tomcat
        ;;
      *)
        exit
        ;;
    esac
    ;;
  No)
      echo "Restart:$Restart"
      ;;
  *)
  exit
      ;;
esac
```

4. 删除旧的发布内容

构建步骤选择“Send files or execute commands over SSh”，“Exec command”处填入如下内容：

```shell
if [ -d /var/lib/docker/volumes/jenkins-data/_data/workspace/test-job/target ];then
  rm -rf /var/lib/docker/volumes/jenkins-data/_data/workspace/test-job/target/*
fi
```

5. 提交新镜像

构建步骤选择“Send files or execute commands over SSh”，“Exec command”处填入如下内容：

```shell
case $Status  in
  Deploy)
    [ -z $(docker images -q tomcat:latest) ] || docker rmi tomcat
    docker commit tomcat tomcat
    ;;
  *)
  exit
      ;;
esac
```

## 发布

- 回到项目主界面，点击Build with Parameters，发布选择Deploy--->开始构建，即可开始发布。
- 回滚选择Rollback--->输入回滚版本---->开始构建，版本号从构建历史中选择一个输入。

## Reference

- [Jenkins版本回滚](https://www.jianshu.com/p/af5fecaa8357)

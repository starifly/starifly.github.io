---
title: spug发布教程
author: starifly
date: 2023-01-26T18:27:18+08:00
lastmod: 2023-01-26T18:27:18+08:00
categories: [spug]
tags: [spug]
draft: true
slug: spug-release-tutorial
---

## 环境管理

首先新建各个环境，比如开发环境、测试环境、预发布环境、生产环境，新建环境时需要填写环境名称和唯一标识符（都可以自定义）。

![](/images/spug-01.png)

## 应用管理

新建应用，有多少个应用就建多少个应用

![](/images/spug-02.png)

## 新建发布

![](/images/spug-03.png)

以常规发布为例，下面分别说明设计平台前端、设计平台后端、管理平台前端和管理平台后端4个应用的发布配置。

**设计平台前端**

其中代码检出后执行的配置：

```
set -e
cd vue-xj-designer
#mkdir node_modules
#cp -a $SPUG_REPOS_DIR/$SPUG_DEPLOY_ID/node_modules/nodejieba ./node_modules
#cp -a $SPUG_REPOS_DIR/$SPUG_DEPLOY_ID/node_modules/node-sass ./node_modules

#cp -a $SPUG_REPOS_DIR/$SPUG_DEPLOY_ID/node_modules ./
#ln -s $SPUG_REPOS_DIR/$SPUG_DEPLOY_ID/node_modules .

#npm cache clean --force
cnpm install --unsafe-perm=true
cnpm run build:prod --unsafe-perm=true

#yarn
#yarn build:stage

rm -rf node_modules

mv dist /tmp
cd ..
rm -rf *
mv /tmp/dist ./design
```

![](/images/spug-04.png)
![](/images/spug-05.png)
![](/images/spug-06.png)

**设计平台后端**

其中应用发布前执行的配置：

```
set -e
if [[ -n $(docker ps -q -f "name=xj-design") ]];then
  docker tag xj-design xj-design:temp
  docker rmi xj-design
  docker build -t xj-design --build-arg JAR_FILE=xj-design.jar .

  docker stop xj-design
  docker rm xj-design
  docker rmi xj-design:temp
  docker run -di --name xj-design --restart=always -p 8088:8088 -v /data/xjlog/design:/home/xjlog -v /etc/localtime:/etc/localtime xj-design --spring.profiles.active=prod
else
  docker build -t xj-design --build-arg JAR_FILE=xj-design.jar .
  docker run -di --name xj-design --restart=always -p 8088:8088 -v /data/xjlog/design:/home/xjlog -v /etc/localtime:/etc/localtime xj-design --spring.profiles.active=prod
fi
```

![](/images/spug-07.png)
![](/images/spug-08.png)
![](/images/spug-09.png)

**管理平台前端**

其中代码检出后执行的配置：

```
set -e
#mkdir node_modules
#cp -a $SPUG_REPOS_DIR/$SPUG_DEPLOY_ID/node_modules/raphael ./node_modules
#cp -a $SPUG_REPOS_DIR/$SPUG_DEPLOY_ID/node_modules/tslib ./node_modules
#cp -a $SPUG_REPOS_DIR/$SPUG_DEPLOY_ID/node_modules/eve ./node_modules
cp -a ../$SPUG_DEPLOY_ID/node_modules ./

#cp -a $SPUG_REPOS_DIR/$SPUG_DEPLOY_ID/node_modules ./
#ln -s $SPUG_REPOS_DIR/$SPUG_DEPLOY_ID/node_modules .

#npm cache clean --force
npm install --unsafe-perm=true
npm run build:dev --unsafe-perm=true
rm -rf node_modules

#yarn
#yarn build

mv dist /tmp
rm -rf *
mv /tmp/dist ./
```

![](/images/spug-10.png)
![](/images/spug-11.png)
![](/images/spug-12.png)

**管理平台后端**

其中应用发布前执行的配置：

```
set -e
if [[ -n $(docker ps -q -f "name=xj-admin") ]];then
  docker tag xj-admin xj-admin:temp
  docker rmi xj-admin
  docker build -t xj-admin --build-arg JAR_FILE=target/xj-admin.jar .

  docker stop xj-admin
  docker rm xj-admin
  docker rmi xj-admin:temp
  docker run -di --name xj-admin --restart=always -p 8070:8070 -v /data/xjlog/admin:/home/xjlog -v /etc/localtime:/etc/localtime xj-admin --spring.profiles.active=prod
else
  docker build -t xj-admin --build-arg JAR_FILE=target/xj-admin.jar .
  docker run -di --name xj-admin --restart=always -p 8070:8070 -v /data/xjlog/admin:/home/xjlog -v /etc/localtime:/etc/localtime xj-admin --spring.profiles.active=prod
fi
```

![](/images/spug-13.png)
![](/images/spug-14.png)
![](/images/spug-15.png)

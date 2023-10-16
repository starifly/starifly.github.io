---
title: mongodb副本集部署
author: starifly
date: 2023-01-26T16:56:17+08:00
lastmod: 2023-01-26T16:56:17+08:00
categories: [mongodb]
tags: [mongodb]
draft: true
slug: deploy-mongodb-replica-set
---

## 下载&解压

```
cd /usr/local/mongodb/
# 下载 
curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-4.0.0.tgz
# 复制至其他服务器
scp -P ${ssh端口} /usr/local/mongodb-linux-x86_64-4.0.0.tgz  ${用户名}@${主机}:/usr/local
# 解压 
tar -zxvf mongodb-linux-x86_64-4.0.0.tgz

# 将解压包拷贝到指定目录 
mv mongodb-linux-x86_64-4.0.0/ /usr/local/mongodb

# 构建软连接
ln -s /usr/local/mongodb/bin/mongod /usr/bin/
ln -s /usr/local/mongodb/bin/mongo /usr/bin/
```

## 修改配置

```
# 创建相关存储目录 
cd /usr/local/mongodb 
# 创建db存放目录
mkdir -p ./data/db 
# 创建日志存放目录
mkdir -p ./logs
# 利用touch命令创建空白日志文件
touch ./logs/mongodb.log

vim mongodb.conf
########### 编辑mongodb配置文件 ###########
#端口号
port=27017
#db目录
dbpath=/usr/local/mongodb/data/db
#日志目录
logpath=/usr/local/mongodb/logs/mongodb.log
#kfile地址
keyFile=/data/mongodb/mongodb-keyfile
#日志增加而不是覆盖
logappend=true
#进程ID文件路径
pidfilepath=/var/run/mongod.pid
#允许远程IP连接
bind_ip=0.0.0.0
#操作日志大小
oplogSize=10000
#是否后台运行 进程在后台运行的守护进程模式
fork=true
#开启用户认证
#auth=true

# 设置副本集名称，在各个配置文件中，其值必须相同
replSet=rs0
```

```
# 创建目录
mkdir -p /data/mongodb
# 创建key文件
openssl rand -base64 741 > /data/mongodb/mongodb-keyfile
# 复制key文件至其他服务器
scp -P ${ssh端口} /data/mongodb/mongodb-keyfile  ${用户名}@${主机}:/home/weihu/
scp -P ${ssh端口} /data/mongodb/mongodb-keyfile  ${用户名}@${主机}:/home/weihu/
# 移动至指定目录
mv /home/weihu/mongodb-keyfile /data/mongodb/mongodb-keyfile

# 修改成 600 的文件属性
chmod 600 /data/mongodb/mongodb-keyfile
```

## 启动服务

```
# 启动服务
mongod -f /usr/local/mongodb/mongodb.conf
```

## 组建副本集

```
mongo
# 选择admin数据库
use admin
# 创建配置项
config = {_id: 'rs0', members: [{_id: 0, host: '172.17.0.3:27017'},{_id: 1, host: '172.17.0.4:27017'},{_id: 2, host:'172.17.0.5:27017'}]}
# 输出如下：
{
    "_id" : "replSet",
    "members" : [
        {
            "_id" : 0,
            "host" : "${主机1}:27017"
        },
        {
            "_id" : 1,
            "host" : "${主机2}:27017"
        },
        {
            "_id" : 2,
            "host" : "${主机3}:27017"
        }
    ]
}
# 初始化副本集：返回{ "ok" : 1 }即成功
rs.initiate(config)
# 查看副本集状态，找到primary节点的IP
rs.status()
```

## 创建帐户密码

```
# 连接主节点
mongo --port 27017
# 切换数据库
use admin
#创建分配用户权限的帐户：admin
db.createUser(
      {
        user: "admin",
        pwd: "123456",
        roles: [ { role: "root", db: "admin" } ]
      }
    )
# 验证
db.auth('admin', '123456')
# 查询用户
show users
db.system.users.find()
#没有则创建mytest数据库
use mytest
# 创建数据库用户
db.createUser({user:"mytest",pwd:"mytest",roles:[{role:"dbOwner",db:"mytest"}]})  
```

## 测试验证

```
# 连接服务
/usr/local/mongodb/bin/mongo 
# 测试
show dbs
```

## Reference

- [MongoDB集群部署](https://www.jianshu.com/p/bdbc7e72d313)
- [MongoDB之副本集配置](https://blog.csdn.net/pengjunlee/article/details/84101732)

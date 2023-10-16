---
title: centos安装OpenJDK 8
author: starifly
date: 2019-10-09T22:11:24+08:00
categories: [jdk]
tags: [jdk,openjdk]
draft: true
slug: install-OpenJDK-8
---

### 安装

```shell
yum -y install java-1.8.0-openjdk java-1.8.0-openjdk-devel
```

### 配置环境变量

```shell
# cat > /etc/profile.d/java8.sh <<EOF
export JAVA_HOME=$(dirname $(dirname $(readlink $(readlink $(which javac)))))
export PATH=\$PATH:\$JAVA_HOME/bin
export CLASSPATH=.:\$JAVA_HOME/jre/lib:\$JAVA_HOME/lib:\$JAVA_HOME/lib/tools.jar
EOF

# source /etc/profile.d/java8.sh
```

> 如果有另外版本的 JDK，通过如下命令配置默认版本  
> `# alternatives --config java`

## Reference

- [CentOS 7 : Install OpenJDK 8](https://www.server-world.info/en/note?os=CentOS_7&p=jdk8&f=2)

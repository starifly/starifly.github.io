---
title: Centos7安装Python3.7以上版本（兼容Python2.7）
author: starifly
date: 2023-01-26T15:10:34+08:00
lastmod: 2023-01-26T15:10:34+08:00
categories: [centos]
tags: [centos,python]
draft: true
slug: centos7-setup-python3
---

## 安装依赖

```
yum install libffi-devel
```

## 下载欲安装Python3安装包

```
wget https://www.python.org/ftp/python/3.7.0/Python-3.7.0.tar.xz
```

## 编译安装

```
tar xvf Python-3.7.0.tar.xz 
cd Python-3.7.0/

./configure \
  prefix=/usr/local/python3 \
	--with-ssl \
	--enable-shared

make && make install
```

测试安装情况

```
/usr/local/python3/bin/python3 -V
```

> 如果出现找不到 libpython3.7m.so.1.0 链接库的问题的话可以尝试搜索下并接入环境变量中

```
# 尝试查找本地，若没有则需要安装
# sudo find / -name libpython3.7m.so.1.0
/data/Python-3.7.10/libpython3.7m.so.1.0
/usr/local/python3/lib/libpython3.7m.so.1.0
# 设置环境变量
# export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/python3/lib
```

## 修改python命令软链

```
mv /usr/bin/python /usr/bin/python.bak
ln -s /usr/local/python3/bin/python3 /usr/bin/python
#Python3自带pip，只需要增加一个软链接即可
ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3
ln -s /usr/local/python3/bin/pyvenv /usr/bin/pyvenv3
```

## Reference

- [Centos7安装Python3.7以上版本（兼容Python2.7](https://blog.51cto.com/leyex/2163465)

---
title: docker部署freeswitch及镜像精简
author: starifly
date: 2020-05-03T16:38:12+08:00
lastmod: 2020-05-03T16:38:12+08:00
categories: [freeswitch]
tags: [freeswitch,docker]
draft: true
slug: docker-deployment-freeswitch-and-reduce-image
---

docker 部署 freeswitch 主要是 Dockerfile 的编写，对于镜像，我们可以利用 docker-slim 工具 或者谷歌的 distroless 镜像对其进行精简。

## docker-slim

### 创建Dcokefile

```
FROM centos:7
MAINTAINER  Fly

ADD fs1.4.26.tar.gz /data/cctdata/cctadmin

RUN yum install -y libedit iproute net-tools hostname inotify-tools yum-utils which \
&& yum install -y git alsa-lib-devel autoconf automake bison epel-release broadvoice-devel \
    bzip2 curl-devel db-devel e2fsprogs-devel flite-devel g722_1-devel gcc-c++ gdbm-devel \
    gnutls-devel ilbc2-devel ldns-devel libcodec2-devel libcurl-devel libedit-devel \
    libidn-devel libjpeg-devel libmemcached-devel libogg-devel libsilk-devel libsndfile-devel \
    libtheora-devel libtiff-devel libtool libuuid-devel libvorbis-devel libxml2-devel lua-devel \
    lzo-devel mongo-c-driver-devel ncurses-devel net-snmp-devel openssl-devel opus-devel \
    pcre-devel perl perl-ExtUtils-Embed pkgconfig portaudio-devel postgresql-devel python26-devel \
    python-devel soundtouch-devel speex-devel sqlite-devel unbound-devel unixODBC-devel wget which yasm zlib-devel \
&& ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
&& echo "/usr/local/lib/vad-check/lib" >> /etc/ld.so.conf \
&& echo "/usr/local/lib/speech-recognize/lib" >> /etc/ld.so.conf \
&& ldconfig \
&& yum clean all

WORKDIR /data/cctdata/cctadmin/fs1.4.26/bin/
#CMD ["/data/cctdata/cctadmin/fs1.4.26/bin/freeswitch"]
CMD ["./freeswitch"]
```

这一步主要是安装 freeswitch 的一些依赖，执行 `docker build -t freeswitch:temp .` 创建临时镜像。

另外一种方式是通过 `ldd freeswitch` 先找出 freeswitch 依赖的库并打包为 lib64.tar.gz ，然后再创建以下 Dockerfile 文件：

```
FROM centos:7
MAINTAINER  Fly

ADD fs1.4.26.tar.gz /data/cctdata/cctadmin

RUN cp -a /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
&& rm -rf /usr/lib64
ADD lib64.tar.gz /usr

WORKDIR /data/cctdata/cctadmin/fs1.4.26/bin/
#CMD ["/data/cctdata/cctadmin/fs1.4.26/bin/freeswitch"]
CMD ["./freeswitch"]
```

### docker-slim精简临时镜像

```
docker-slim build --http-probe=false --include-path='/usr/lib64:755' freeswitch:temp
```

将自动生成 freeswitch.slim 的镜像，镜像能够缩小好几倍（从几百兆缩减至一百多兆大小），此外我们可以执行 `docker tag freeswitch.slim freeswitch` 对其进行重命名。

### 运行容器

```
docker run -dit -v /opt/docker/freeswitch/fs1.4.26:/data/cctdata/cctadmin/fs1.4.26 freeswitch
``` 

注意 `/opt/docker/freeswitch/fs1.4.26` 目录是宿主机上的 freeswitch 目录，因为容器内实际上没有安装 freeswitch ，所以要进行映射。

## distroless

### 创建Dcokefile

```
FROM centos:7 as base
MAINTAINER  Fly

#ADD fs1.4.26.tar.gz /data/cctdata/cctadmin

ARG Asia/Shanghai

ADD lib64.tar.gz /opt
RUN mkdir -p /opt/etc && \
    cp -a /usr/share/zoneinfo/Asia/Shanghai /opt/etc/localtime

FROM wl86129/distroless_base

COPY --from=base /opt /

WORKDIR /data/cctdata/cctadmin/fs1.4.26/bin/
#CMD ["/data/cctdata/cctadmin/fs1.4.26/bin/freeswitch"]
CMD ["./freeswitch"]
```

执行 `docker build -t freeswitch .`

### 运行容器

```
docker run -dit -v /opt/docker/freeswitch/fs1.4.26:/data/cctdata/cctadmin/fs1.4.26 freeswitch
``` 

## Reference

- [优化docker镜像的几种方法](https://mp.weixin.qq.com/s/AzX-r3k0qfRdb6mQofhMvw)
- [两个奇技淫巧，将 Docker 镜像体积减小 99%](https://mp.weixin.qq.com/s/6bgtD0Aer6-3u4u9jWBBhw)
- [docker-slim](https://github.com/docker-slim/docker-slim)
- [关于docker的scratch镜像与helloworld](https://www.cnblogs.com/uscWIFI/p/11917662.html)

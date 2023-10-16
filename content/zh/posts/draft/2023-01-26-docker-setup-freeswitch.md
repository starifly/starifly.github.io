---
title: docker安装freeswitch1.10.6
author: starifly
date: 2023-01-26T15:59:01+08:00
lastmod: 2023-01-26T15:59:01+08:00
categories: [docker,freeswitch]
tags: [docker,freeswitch]
draft: true
slug: docker-setup-freeswitch
---

Docker 部署 FreeSWITCH 1.10.6，基于debian:buster（第一阶段）和debian:buster-slim（第二阶段）,采用多阶段构建，以减少镜像体积。

Dockerfile文件内容如下：

```
#MAINTAINER "Fly fly@cn"
FROM debian:buster as build

ENV FS_VERSION=v1.10
ENV FS_TAG=v1.10.6

# This prevents us from get errors during apt-get installs as it notifies the
# environment that it is a non-interactive one.
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update &&\
    apt-get install -y wget git-core autoconf libtool-bin build-essential lsb-release libtool libtiff-dev libjpeg-dev cmake build-essential uuid-dev &&\
    wget -O - https://files.freeswitch.org/repo/deb/debian-release/fsstretch-archive-keyring.asc | apt-key add - &&\
    echo "deb http://files.freeswitch.org/repo/deb/debian-release/ `lsb_release -sc` main" > /etc/apt/sources.list.d/freeswitch.list &&\
    echo "deb-src http://files.freeswitch.org/repo/deb/debian-release/ `lsb_release -sc` main" >> /etc/apt/sources.list.d/freeswitch.list &&\
    apt update &&\
    apt-get build-dep -y freeswitch

RUN cd /tmp  &&\
    git clone https://mirror.ghproxy.com/https://github.com/freeswitch/spandsp spandsp &&\
    cd spandsp && ./bootstrap.sh
RUN    cd /tmp/spandsp && ./configure
RUN    cd /tmp/spandsp &&  make && make install && ldconfig

RUN cd /tmp  &&\
    git clone https://mirror.ghproxy.com/https://github.com/signalwire/libks
RUN    cd /tmp/libks && cmake .
RUN    cd /tmp/libks &&  make && make install && ldconfig

RUN cd /tmp  &&\
    git clone https://mirror.ghproxy.com/https://github.com/signalwire/signalwire-c
RUN    cd /tmp/signalwire-c && cmake .
RUN    cd /tmp/signalwire-c &&  make && make install && ldconfig

RUN cd /tmp  &&\
    git clone https://mirror.ghproxy.com/https://github.com/freeswitch/sofia-sip sofia-sip &&\
    cd sofia-sip && ./bootstrap.sh
RUN    cd /tmp/sofia-sip && ./configure
RUN    cd /tmp/sofia-sip &&  make && make install && ldconfig

#ADD build/modules.conf /usr/src/freeswitch/modules.conf

RUN cd /tmp  &&\
    git clone -b ${FS_VERSION} https://mirror.ghproxy.com/https://github.com/signalwire/freeswitch.git freeswitch &&\
    cd freeswitch &&\
    git checkout ${FS_TAG} &&\
    git config pull.rebase true &&\
#    git apply /tmp/signalwire-disabled.patch &&\
    ./bootstrap.sh -j
RUN    cd /tmp/freeswitch && ./configure --enable-portable-binary \
         --prefix=/usr --localstatedir=/var --sysconfdir=/etc \
         --with-gnu-ld --with-openssl \
         --enable-core-odbc-support --enable-zrtp \
         --enable-core-pgsql-support
RUN    cd /tmp/freeswitch &&  make && make install

FROM debian:buster-slim

ARG TZ=Asia/Shanghai

RUN rm -f /etc/timezone && \
      echo "${TZ}" > /etc/timezone && \
      ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && \
      dpkg-reconfigure -f noninteractive tzdata

ENV DEBIAN_FRONTEND noninteractive
RUN apt update && apt-get install -y unixodbc libssl-dev sqlite libfreetype6 libcurl4-openssl-dev \
    libspeex1 libspeexdsp1 libedit2 libtpl0 libldns-dev libopencore-amrnb-dev libopus-dev libavformat-dev libsndfile1-dev libsndfile-dev liblua5.2-dev libswscale-dev
#RUN apk update && apk add libpq libuuid sqlite-dev curl-dev pcre-dev speex-dev speexdsp-dev libedit-dev unixodbc-dev \
#    jpeg-dev opus-dev tiff-dev libsndfile-dev lua5.2-dev ldns-dev

COPY --from=build /etc/freeswitch /etc/freeswitch
COPY --from=build /run/freeswitch /run/freeswitch

COPY --from=build ["/usr/bin/freeswitch", "/usr/bin/fs_cli", "/usr/bin/fs_encode", "/usr/bin/fs_ivrd", \
    "/usr/bin/fsxs", "/usr/bin/gentls_cert", "/usr/bin/fs_tts", "/usr/bin/tone2wav", "/usr/bin/"]

COPY --from=build /usr/include/freeswitch /usr/include/freeswitch
COPY --from=build /usr/lib/freeswitch /usr/lib/freeswitch
COPY --from=build /usr/lib/libfreeswitch.* /usr/lib/
COPY --from=build /usr/lib/pkgconfig/freeswitch.pc /usr/lib/pkgconfig/
COPY --from=build /usr/share/freeswitch /usr/share/freeswitch
#COPY --from=build /var/lib/freeswitch /var/lib/freeswitch
COPY --from=build /var/log/freeswitch /var/log/freeswitch

COPY --from=build /usr/local/include/spandsp /usr/local/include/spandsp
COPY --from=build /usr/local/lib/libspandsp.* /usr/local/lib/
COPY --from=build /usr/local/lib/pkgconfig/spandsp.pc /usr/local/lib/pkgconfig/

COPY --from=build /usr/include/libks /usr/include/libks
COPY --from=build /usr/lib/libks.* /usr/lib/
COPY --from=build /usr/share/doc/libks /usr/share/doc/libks
COPY --from=build /usr/lib/pkgconfig/libks.pc /usr/lib/pkgconfig/

COPY --from=build /usr/local/include/signalwire-client-c /usr/local/include/signalwire-client-c
COPY --from=build /usr/local/lib/libsignalwire_client.* /usr/local/lib/
COPY --from=build /usr/local/share/doc/signalwire-client-c /usr/local/share/doc/signalwire-client-c
COPY --from=build /usr/local/lib/pkgconfig/signalwire_client.pc /usr/local/lib/pkgconfig/

COPY --from=build /usr/local/bin/addrinfo /usr/local/bin/
COPY --from=build /usr/local/bin/localinfo /usr/local/bin/
COPY --from=build /usr/local/include/sofia-sip-1.13 /usr/local/include/sofia-sip-1.13/
COPY --from=build /usr/local/lib/libsofia-sip-ua.* /usr/local/lib/
COPY --from=build /usr/local/share/sofia-sip /usr/local/share/sofia-sip
COPY --from=build /usr/local/lib/pkgconfig/sofia-sip-ua.pc /usr/local/lib/pkgconfig/
COPY --from=build /usr/local/bin/sip-options /usr/local/bin/
COPY --from=build /usr/local/bin/sip-date /usr/local/bin/
COPY --from=build /usr/local/bin/sip-dig /usr/local/bin/

RUN ldconfig

ENTRYPOINT ["/usr/bin/freeswitch"]
```

## Reference

- [](https://github.com/Vetal-ca/docker-freeswitch)
- [](https://hub.fastgit.org/Vetal-ca/docker-freeswitch)
- [](https://github.com/sip-rtc/freeswitch-docker)
- [Docker 部署 FreeSWITCH](http://www.manongjc.com/detail/16-eyysxqhnfqdgzun.html)
- [使用docker部署freeswitch](https://zhuanlan.zhihu.com/p/379561988)
- [ubuntu20.04 freeswitch1.10 的安装部署](https://blog.csdn.net/fengyou200/article/details/115538166)

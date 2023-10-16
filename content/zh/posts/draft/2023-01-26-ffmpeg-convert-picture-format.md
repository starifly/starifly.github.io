---
title: ffmpeg转换图片格式为webp
author: starifly
date: 2023-01-26T16:50:54+08:00
lastmod: 2023-01-26T16:50:54+08:00
categories: [ffmpeg]
tags: [ffmpeg,webp]
draft: true
slug: ffmpeg-convert-picture-format
---

ffmepg版本：N-105884-g74117ab
libwebp版本：1.2.2

## 安装libwebp

手动编译安装libwebp

```
yum install autoconf automake libtool libsysfs
git clone https://github.com/webmproject/libwebp.git
cd libwebp
./autogen.sh
./configure --prefix=/usr/local/libwebp
make
make install
export PKG_CONFIG_PATH="/usr/local/libwebp/lib/pkgconfig/"
```

## 安装ffmpeg

```
git clone https://git.ffmpeg.org/ffmpeg.git ffmpeg
cd ffmpeg
# 如果提示yasm报错，yum install yasm
./configure --prefix=/usr/local/ffmpeg --enable-libwebp --extra-cflags="-I/usr/local/libwebp/include/webp" --extra-ldflags="-L/usr/local/libwebp/lib"
make
make install
```

添加ffmpeg到环境变量

```
export FFMEPG=/usr/local/ffmpeg
export PATH=${FFMEPG}/bin:${PATH}
source /etc/profile
```

## 使用

安装好后，我们可以使用ffmpeg来对图片进行转换，比如从某种格式转换为webp格式，可以缩减图片大小同时保留一定的图片质量，可以根据实际情况对参数进行配置“compression_level”和“quality”，比较均衡的配置是“compression_level”设为6，“quality”设为75。

```
#!/bin/bash

num=5
fifofile="/tmp/$$.fifo"

#创建管道文件，以8作为管道符，删除不影响句柄使用
mkfifo $fifofile
exec 8<> $fifofile
rm $fifofile

#创建for循环使得管道中初始化已存在5行空行
for i in `seq $num`
do
        echo "" >&8
done

#for string in $(find /data/oss -name "*.jpg" -o -name "*.jpeg" -o -name "*.png" -size +1M -type f)
find /data/oss \( -name "*.jpg" -o -name "*.jpeg" \) -a -size +2097152c -a -type f | while read string
do
  read -u 8
  {
 # echo "$string"
  tmpname=${string}
  if [[ ${tmpname##*.} = "jpeg" ]]; then
    #fileName=$(basename "$string" jpeg)
    dirName=$(echo $string | sed 's/\.jpeg$//')
    #echo "jpeg $dirName $fileName"
    #mv "$tmpname" "$dirName".webp
    ffmpeg -i "$tmpname" -codec libwebp -compression_level 6 -quality 75 "$dirName".webp
  elif [[ ${tmpname##*.} = "jpg" ]]; then
    #fileName=$(basename "$string" .jpg)
    dirName=$(echo $string | sed 's/\.jpg$//')
    #echo "jpg $dirName $fileName"
    #mv "$tmpname" "$dirName".webp
    ffmpeg -i "$tmpname" -codec libwebp -compression_level 6 -quality 75 "$dirName".webp
  fi
  #distname=${string//\//--}
  #cp -a "$tmpname" /data/dist/"$distname"
  echo >&8
  }&
done
wait
echo "ffmpeg run finish..."
```

## Reference

- [Centos7安装FFmpeg](https://blog.csdn.net/weixin_43404791/article/details/106163871)
- [使用ffmpeg进行webp图片压缩，ffmpeg的帮助信息查看方法](https://www.jianshu.com/p/1b370c2f05f3)
- [视频转 webp 动图](https://www.jianshu.com/p/f06eb4685fd7)

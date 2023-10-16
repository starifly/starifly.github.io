---
title: Zabbix图形中文字体显示方块处理
author: starifly
date: 2019-06-07T22:18:05+08:00
categories: [zabbix]
tags: [zabbix]
draft: true
slug: Zabbix-graphic-font
---

原因很简单，图形显示用的字体是dejavu，不支持中文。

怎么办？先理清逻辑。

zabbix 配置文件(/usr/share/zabbix/include/defines.inc.php)里，定义的字体叫做 graphfont.ttf，然后一路软链接到 DejaVuSans.ttf，如下：

```shell
/usr/share/zabbix/graphfont.ttf -> /etc/alternatives/zabbix-web-font -> /usr/share/fonts/dejavu/DejaVuSans.ttf
```

那么，修改掉最后一层软连接的目标字体就可以了。执行类似下面的命令就可以了。

```shell
yum install google-noto-sans-simplified-chinese-fonts.noarch -y
mv /etc/alternatives/zabbix-web-font /etc/alternatives/zabbix-web-font_bak 
ln -s /usr/share/fonts/google-noto/NotoSansSC-Regular.otf /etc/alternatives/zabbix-web-font
```

## Reference

- <https://www.centos.bz/2017/10/zabbix%E5%9B%BE%E5%BD%A2%E4%B8%AD%E6%96%87%E5%AD%97%E4%BD%93%E6%98%BE%E7%A4%BA%E6%96%B9%E5%9D%97%E5%A4%84%E7%90%86/>

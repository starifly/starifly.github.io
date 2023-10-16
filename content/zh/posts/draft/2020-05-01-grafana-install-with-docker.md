---
title: docker安装grafana
author: starifly
date: 2020-05-01T22:52:16+08:00
lastmod: 2020-05-01T22:52:16+08:00
categories: [grafana]
tags: [grafana,docker]
draft: true
slug: grafana-install-with-docker
---

## docker安装grafana

```
mkdir -p /data/grafana
chmod 777 /data/grafana
docker run -dit --net=host --name=grafana -v /data/grafana:/var/lib/grafana --restart=always grafana/grafana
# 安装grafana-zabbix插件
docker exec grafana  grafana-cli plugins install alexanderzobnin-zabbix-app
# 如果上面的命令下载速度比较慢，也可以上grafana官网（https://grafana.com/grafana/plugins/alexanderzobnin-zabbix-app/installation）下载zabbix插件，解压后传递到宿主机的/data/grafana/plugins目录下
# 重启grafana
docker restart  grafana
# 再进grafana插件配置页面，启用zabbix,并配置数据源，不再赘述
```

## Reference

- [Docker方式安装ZabbixServer/Agent及Grafana](https://www.jianshu.com/p/055a3cf63233)

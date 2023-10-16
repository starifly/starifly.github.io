---
title: prometheus 监控 nginx 使用 nginx-vts-exporter 采集数据
author: starifly
date: 2023-01-26T17:35:38+08:00
lastmod: 2023-01-26T17:35:38+08:00
categories: [prometheus]
tags: [prometheus]
draft: true
slug: prometheus-nginx-vts-exporter
---

prometheus 监控 nginx 使用 nginx-vts-exporter 采集数据。同时，需要 nginx 支持 nginx-module-vts 模块获取 nginx 自身的一些数据。

## nginx的模块支持

git clone git://github.com/vozlt/nginx-module-vts.git

编译安装nginx，只需要在之前的编译参数中加上 --add-module=/path/to/nginx-module-vts 即可

```
./configure --user=nginx --group=nginx --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --conf-path=/etc/nginx/nginx.conf --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --with-http_gzip_static_module --with-http_stub_status_module --with-http_ssl_module --with-pcre --with-http_v2_module --add-module=/path/to/nginx-module-vts
```

修改nginx配置

```
server {
    #如果需要https需要如此配置，不需要的话listen 只需要加80端口号和server_name
    listen 8099;
    location /nginx_status  {
        stub_status on;
        access_log off;
        #allow 127.0.0.1;允许哪个ip可以访问
    }
}
vhost_traffic_status_zone;
#vhost_traffic_status_zone shared:vhost_traffic_status:64m; #如果提示nginx提示内存不足可以开启此项配置
#vhost_traffic_status_filter_by_set_key $uri uris::$server_name; #监控uri情况
#vhost_traffic_status_filter_by_set_key $remote_port client::ports::$server_name; #监控端口情况
#vhost_traffic_status_filter_by_set_key $geoip_country_code country::*;
vhost_traffic_status_filter_by_set_key $geoip_country_code country::$server_name; #监控访问国家
vhost_traffic_status_filter_by_host on; #开启此功能，会根据不同的server_name进行流量的统计，否则默认会把流量计算到第一个上
geoip_country /etc/nginx/geoip/GeoIP.dat; #监控访问国家需要用到的数据库
server {
    listen 8090;
	allow 127.0.0.1；
    allow prometheus_server_ip;  #替换为你的prometheus ip；
    location /status {
	    access_log off;
        vhost_traffic_status_display;
        vhost_traffic_status_display_format html;
    }

	#测试，返回国家城市信息
    location /myip {
        default_type text/plain;
        return 200 "$remote_addr $geoip_country_name $geoip_country_code $geoip_city";
    }
}
```

在不想统计流量的server 区域(未规范配置server_name或者无需进行监控的server上)可以禁用 vhost_traffic_status：

```
server {
vhost_traffic_status off；
...
}
```

## 数据展示

访问 http://127.0.0.1:8090/status/format/prometheus 可直接获取prometheus格式的监控数据。
访问 http://127.0.0.1:8090/status/format/json 可直接获取json格式的监控数据。

## 接入prometheus

nginx-vts-exporter 抓取vts数据传向prometheus，docker安装：

```
docker run  -dti --restart=always --net=host --env NGINX_STATUS="http://localhost:8090/status/format/json" sophos/nginx-vts-exporter
```

端口为9913，查看数据：

```
curl http://127.0.0.1:9913/metrics
```

在Prometheus中添加此target 便可接入。

## 接入grafana

QPS 以5分钟为时间维度进行统计展示

![](/images/prometheus-01.png)

访问国家统计

![](/images/prometheus-02.png)

## Reference

- [https://www.cnblogs.com/huandada/p/10472031.html]( prometheus — nginx-vts-exporter )
- [https://blog.csdn.net/specter11235/article/details/86755857](prometheus配置nginx监控)
- [https://www.cnblogs.com/tobrainto/p/14328566.html](Nginx虚拟主机流量状态模块（nginx-module-vts）使用说明文档（一） )
- [https://www.cnblogs.com/reblue520/p/7885027.html](通过python操作GeoLite2-City.mmdb库将nginx日志访问IP转换为城市写入数据库)
- [https://blog.csdn.net/b_boyyang/article/details/81939452](Nginx服务器中配置GeoIP模块来拦截指定国家IP)`

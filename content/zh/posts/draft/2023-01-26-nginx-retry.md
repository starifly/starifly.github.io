---
title: nginx重试机制
author: starifly
date: 2023-01-26T18:44:31+08:00
lastmod: 2023-01-26T18:44:31+08:00
categories: [nginx]
tags: [nginx]
draft: true
slug: nginx-retry
---

对外服务的网站，很少只使用一个服务节点，大多都是部署多台服务器，通过一定机制保证容错和负载均衡。

nginx的重试机制就是常用一种HTTP反向代理服务器的支持容错和负载均衡。

```
	upstream tomcatserver1 {  
		server 192.168.1.9:8081 weight=2;  
		server 192.168.1.29:8081 weight=2;  
    }
	server {
        listen       8082;
        listen       192.168.1.29:8082;
        server_name  192.168.1.29;
 
 
		location / {
            proxy_pass http://tomcatserver1;
			index  index.html index.htm;
			proxy_next_upstream error timeout http_404 http_502 http_503 non_idempotent;
			proxy_connect_timeout 20s;
			proxy_read_timeout 20s;
			proxy_send_timeout 20s;
			
			#开启gzip压缩，降低传输流量
			gzip  on;
			gzip_min_length    1k;
			gzip_buffers    4 16k;
			gzip_http_version  1.1;
			gzip_comp_level  2;
			gzip_types  text/plain application/x-javascript text/css  application/xml  application/json  image/jpeg image/gif image/png;
			gzip_vary on;
			
			
        }
		
	}
```

配置说明：

当其中一个服务节点各种原因宕机、或不可访问、或超时则去访问另一个其他服务。

实例中遇到503错误则去请求另外服务，但是有可能服务请求需要25s返回，则有可能造成重复提交情况。

## Reference

- [(三）nginx反向代理html，nginx的重试机制 proxy_next_upstream](https://blog.csdn.net/goodlook0123/article/details/83344821)
- [【nginx】- 通过ip_hash实现不停机发布 ](https://www.cnblogs.com/codetalking/p/13176203.html)

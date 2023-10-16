---
title: nginx ssl优化
author: starifly
date: 2023-01-26T18:36:54+08:00
lastmod: 2023-01-26T18:36:54+08:00
categories: [nginx]
tags: [nginx]
draft: true
slug: nginx-ssl-optimization
---
## 调整 Cipher 优先级

尽量挑选更新更快的 Cipher，有助于减少延迟：

```
# 启用ssl_prefer_server_ciphers，用来告诉Nginx在TLS握手时启用服务器算法优先，由服务器选择适配算法而不是客户端
ssl_prefer_server_ciphers on;
ssl_ciphers 'EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH';
```

## 启用 OCSP Stapling

在国内这可能是对使用 Let’s Encrypt 证书的服务或网站影响最大的延迟优化了。如果不启用 OCSP Stapling 的话，在用户连接你的服务器的时候，有时候需要去验证证书。而因为一些不可知的原因（这个就不说穿了）Let’s Encrypt 的验证服务器并不是非常通畅[6]，因此可以造成有时候数秒甚至十几秒延迟的问题

解决这个问题的方法有两个：

1. 不使用 Let’s Encrypt，可以尝试替换为阿里云提供的免费 DV 证书
2. 开启 OCSP Stapling,开启了 OCSP Stapling 的话，跑到证书验证这一步可以省略掉。省掉一个 roundtrip，特别是网络状况不可控的 roundtrip，可能可以将你的延迟大大减少。

在 Nginx 中启用 OCSP Stapling 也非常简单，只需要设置：

```
ssl_stapling on;
ssl_stapling_verify on;
ssl_trusted_certificate /path/to/full_chain.pem;
```
如何检测 OCSP Stapling 是否已经开启？
可以通过以下命令来测试

```
openssl s_client -connect test.kalasearch.cn:443 -servername kalasearch.cn -status -tlsextdebug < /dev/null 2>&1 | grep -i "OCSP response"
```

如果结果为如下则表明已经开启。

```
OCSP response:
OCSP Response Data:
    OCSP Response Status: successful (0x0)
    Response Type: Basic OCSP Response
```

## 启用 SSL Session 缓存

缓存 SSL 连接凭据可以避免频繁握手带来的速度降低和性能损耗。

TLS 协议有两类会话缓存机制：会话标识 session ID 与会话记录 session ticket。session ID 由服务器端支持，协议中的标准字段，因此基本所有服务器都支持，服务器端保存会话ID以及协商的通信信息，Nginx 中1M 内存约可以保存4000个 session ID 机器相关信息，占用服务器资源较多；而 session ticket 属于一个 TLS 扩展字段，需要服务器和客户端都支持。

二者对比，主要是保存协商信息的位置与方式不同，类似与 HTTP 中的 session 与 cookie。都存在的情况下，Nginx 优先使用 session_ticket。

启用 SSL Session 缓存可以大大减少 TLS 的反复验证，减少 TLS 握手的 roundtrip。虽然 session 缓存会占用一定内存，但是用 1M 的内存就可以缓存 4000 个连接，可以说是非常非常划算的。同时，对于绝大多数网站和服务，要达到 4000 个同时连接本身就需要非常非常大的用户基数，因此可以放心开启。

这里 **ssl_session_cache** 设置为使用 20M 内存，以及 2 小时的连接超时关闭时间 ssl_session_timeout

```
# Enable SSL cache to speed up for return visitors
ssl_session_cache   shared:SSL:20m; # speed up first time. 1m ~= 4000 connections
ssl_session_timeout 2h;
```

## Reference

- [高性能 Nginx HTTPS 调优 - 如何为 HTTPS 提速 30%](https://www.cnblogs.com/blxt/p/14501181.html)
- [记一个开启 ssl_session_tickets 导致的 SSL 证书冲突的问题](https://cong5.net/post/nginx-ssl-session-tickets-error)
- [SSL 配置优化的若干建议](https://blog.csdn.net/vencent7/article/details/79190249)
- [SSL 配置优化的若干建议](https://blog.csdn.net/vencent7/article/details/79190249)

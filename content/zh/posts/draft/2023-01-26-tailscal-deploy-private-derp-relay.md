---
title: Tailscale部署私有derp中继教程
author: starifly
date: 2023-01-26T16:37:57+08:00
lastmod: 2023-01-26T16:37:57+08:00
categories: [tailscale,网络]
tags: [tailscale,网络,derp]
draft: true
slug: tailscal-deploy-private-derp-relay
---

## 中继是什么

对于 Hard NAT 来说，STUN 就不好使了，即使 STUN 拿到了客户端的公网 ip:port 告诉通信对端也于事无补，因为防火墙是和 STUN 通信才打开的缺口，这个缺口只允许 STUN 的入向包进入，其他通信对端知道了这个缺口也进不来。通常企业级 NAT 都属于 Hard NAT。

这种情况下打洞是不可能了，但也不能就此放弃，可以选择一种折衷的方式：创建一个中继服务器（relay server），客户端与中继服务器进行通信，中继服务器再将包中继（relay）给通信对端。

至于中继的性能，那要看具体情况了：

- 如果能直连，那显然没必要用中继方式；  
- 但如果无法直连，而中继路径又非常接近双方直连的真实路径，并且带宽足够大，那中继方式并不会明显降低通信质量。延迟肯定会增加一点，带宽会占用一些，但相比完全连接不上，还是可以接受的。

事实上对于大部分网络而言，Tailscale 都可以通过各种黑科技打洞成功，只有极少数情况下才会选择中继，中继只是一种 fallback 机制。

## 中继协议简介

中继协议有多种实现方式，如TURN、DERP等。

DERP 即 Detoured Encrypted Routing Protocol，这是 Tailscale 自研的一个协议：

- 它是一个通用目的包中继协议，运行在 HTTP 之上，而大部分网络都是允许 HTTP 通信的。  
- 它根据目的公钥（destination’s public key）来中继加密的流量（encrypted payloads）。

Tailscale 使用的算法很有趣，所有客户端之间的连接都是先选择 DERP 模式（中继模式），这意味着连接立即就能建立（优先级最低但 100% 能成功的模式），用户不用任何等待。然后开始并行地进行路径发现，通常几秒钟之后，我们就能发现一条更优路径，然后将现有连接透明升级（upgrade）过去，变成点对点连接（直连）。

因此，DERP 既是 Tailscale 在 NAT 穿透失败时的保底通信方式（此时的角色与 TURN 类似），也是在其他一些场景下帮助我们完成 NAT 穿透的旁路信道。 换句话说，它既是我们的保底方式，也是有更好的穿透链路时，帮助我们进行连接升级（upgrade to a peer-to-peer connection）的基础设施。

## 自建私有 DERP server

Tailscale 官方内置了很多 DERP 服务器，分步在全球各地，惟独不包含中国大陆，原因你懂得。这就导致了一旦流量通过 DERP 服务器进行中继，延时就会非常高。而且官方提供的 DERP 服务器是万人骑，存在安全隐患。

为了实现低延迟、高安全性，我们可以参考 [Tailscale 官方文档](https://tailscale.com/kb/1118/custom-derp-servers/)自建私有的 DERP 服务器。有两种部署模式，一种是基于域名，另外一种不需要域名，可以直接使用 IP，不过需要一点黑科技。

### 使用纯 IP

用docker启动derper：

```
docker run --restart always --net host --name derper -d ghcr.io/yangchuansheng/ip_derper
```

运行后查看容器日志，放行相应得端口。

除了 derper 之外，Tailscale 客户端还需要跳过域名验证，这个需要在 DERP 的配置中设置。而 Headscale 的本地 YAML 文件目前还不支持这个配置项，所以没办法，咱只能使用在线 URL 了。JSON 配置内容如下：

```
{
  "Regions": {
    "901": {
      "RegionID": 901,
      "RegionCode": "ali-sh",
      "RegionName": "Aliyun Shanghai",
      "Nodes": [
        {
          "Name": "901a",
          "RegionID": 901,
          "DERPPort": 443,
          "HostName": "xxxx",
          "IPv4": "xxxx",
          "InsecureForTests": true
        }
      ]
    }
  }
}
```

配置解析：

- HostName 直接填 derper 的公网 IP，即和 IPv4 的值相同。
- InsecureForTests 一定要设置为 true，以跳过域名验证。

然后你需要把这个 JSON 文件变成 Headscale 服务器可以访问的 URL，比如在 Headscale 主机上搭个 Nginx，或者上传到对象存储（比如阿里云 OSS）。

接下来还需要修改 Headscale 的配置文件，引用上面的自定义 DERP 的 URL。需要修改的配置项如下：

```
# /etc/headscale/config.yaml
derp:
  # List of externally available DERP maps encoded in JSON
  urls:
  #  - https://controlplane.tailscale.com/derpmap/default
    - https://xxxxx/derp.json

  # Locally available DERP map files encoded in YAML
  #
  # This option is mostly interesting for people hosting
  # their own DERP servers:
  # https://tailscale.com/kb/1118/custom-derp-servers/
  #
  # paths:
  #   - /etc/headscale/derp-example.yaml
  paths:
    - /etc/headscale/derp.yaml

  # If enabled, a worker will be set up to periodically
  # refresh the given sources and update the derpmap
  # will be set up.
  auto_update_enabled: true

  # How often should we check for DERP updates?
  update_frequency: 24h
```

修改完配置后，重启 headscale 服务：

```
systemctl restart headscale
```

在 Tailscale 客户端上使用以下命令查看目前可以使用的 DERP 服务器：

```
tailscale netcheck
```

再次查看与通信对端的连接方式：

```
tailscale status
```

### 使用域名

参考 <https://fuckcloudnative.io/posts/custom-derp-servers/#使用域名>

## 防止 DERP 被白嫖

只有使用域名的方式才可以通过认证防止被白嫖，使用纯 IP 的方式无法防白嫖，你只能小心翼翼地隐藏好你的 IP 和端口，不能让别人知道，因为别人一旦知道了你的 DERP 服务器的地址和端口，就可以为他所用。

那怎么防止呢？只需要做两件事情：

1、在 DERP 服务器上安装 Tailscale并启动 tailscaled 进程。

2、derper 启动时加上参数 --verify-clients。

默认情况下 --verify-clients 参数设置的是 false。我们不需要对 Dockerfile 内容做任何改动，只需在容器启动时加上环境变量即可，将启动命令修改一下，加入“-e DERP_VERIFY_CLIENTS=true”参数。

## Reference

- [Tailscale 基础教程：部署私有 DERP 中继服务器](https://fuckcloudnative.io/posts/custom-derp-servers/)
- [Custom DERP Servers](https://tailscale.com/kb/1118/custom-derp-servers/)
- [Encrypted TCP relays (DERP)](https://tailscale.com/blog/how-tailscale-works/#encrypted-tcp-relays-derp)



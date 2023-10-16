---
title: SIP路由机制解析
author: starifly
date: 2020-05-24T22:05:56+08:00
lastmod: 2020-05-24T22:05:56+08:00
categories: [sip]
tags: [sip]
draft: true
slug: 2020-05-24-sip-From-Contact-Via-Record-Route
---

在前面已经陆续介绍了SIP重要头域、注册流程、会话流程等SIP相关知识，现在再来介绍一下SIP中的路由机制。

SIP中存在两种路由场景：

*   1，请求消息的路由
*   2，响应消息的路由

其中，响应消息的路由非常简单，就是完全依靠Via来完成的，具体请见我关于RFC3261中会话流程的分析。  
下面我们只谈SIP请求消息的路由。

首先我们要搞清楚什么是`严格路由`和`松散路由`。

**严格路由（Strict Routing）：**

> 可以理解为比较“死板”的理由机制，这种路由机制在SIP协议的前身RFC 2534中定义，其机制非常简单。  
> 要求接收到的消息的request-URI必须是自己的URI，然后它会把第一个Route头域“弹”出来，并把其中的URI作为新的request-RUI，然后把该消息路由给该URI。

**松散路由（Louse Routing，lr）：**

> 该路由机制较为灵活，也是SIP路由机制的灵魂所在，在SIP根本大典RFC 3261中定义。  
> 下面介绍一下一个松散路由的Proxy的路由决策过程：
> 
> 1，Proxy首先会检查消息的request-URI是不是自己属于自己所负责的域。如果是，它就会通过定位服务将该地址“翻译”成具体的联系地址并以此替换掉原来的request-URI；否则，它不会动request-URI。
> 
> 2，Proxy检查第一个Route头域中的URI是不是自己的，如果是，则移除之。
> 
> 3，前面两项都是准备工作，下面该进行真正的路由了。如果还有Route头域，则Proxy会把消息路由给该头域中的URI，否则就路由给request-URI。至于如何从下一跳URI确定出IP地址，端口以及传输协议那是另外一回事了。

对于前面的3条规则，我们可以简单总结为一句话：`Route的优先级高于request-URI的`

好，了解了两种路由机制，我们再来了解一下`Route`和`Record-Route`。  
如果说Via是为了给一个请求消息的响应消息留后路，那么`Record-Route`就是为了给该请求消息之后的请求消息留后路。  
【说明】一个SIP消息每经过一个Proxy（包括主叫），都会被加上一个Via头域，当消息到达被叫后，Via头域就记录了请求消息经过的完整路径。被 叫将这些Via头域原样copy到响应消息中（包括各Via的参数，以及各Via的顺序），然后下发给第一个Via中的URI，每个Proxy转发响应消 息前都会把第一个Via（也就是它自己添加的Via）删除，然后将消息转发给新的第一个Via中的URI，直到消息到达主叫。

而在一个请求消息的传输过程中，Proxy也可能（纯粹自愿，如果它希望还能接收到本次会话的后续请求消息的话）会添加一个`Record-Route`头 域，这样当消息到达被叫后里面就有会有0个或若干个`Record-Route`头域。被叫会将这些`Record-Route`头域并入路由集，并并入自己的路 由集，随后被叫在发送请求消息时就会使用该路由集构造一系列`Route`头域，以便对消息进行路由。  
然后，被叫会像上面对待Via头域一样，将`Record-Route`头域全部原样copy到响应消息中返回给主叫。  
主叫收到响应消息后也会将这些`Record-Route`头域并入路由集，只是它会将其反序。该会话中的后续请求消息的`Route`头域就会通过路由集构造。

【注意】`Record-Route`头域不用来路由，而只是起到传递信息的作用。  
`Record-Route`头域不是路由集的唯一来源，路由集还可以通过手工配置等方式得到。

只是描述还是比较抽象，下面就以RFC 3261中的两个实例来解释一下。

* * *

**路由示例1：**
----------

**场景：**  
两个UE间有两个Proxy，U1 -> P1 -> P2 -> U2，并且两个Proxy都乐意添加Record-Route头域。

**消息流：**  
【说明】由于我们在此只关心SIP路由机制，因此下面消息中跟路由机制无关的头域都省略了。

U1发出一个INVITE请求给P1（P1是U1的外拨代理服务器）：

> INVITE [sip:callee@domain.com](mailto:sip:callee@domain.com) SIP/2.0 Contact:  
> [sip:caller@u1.example.com](mailto:sip:caller@u1.example.com)

[P1不负责域domain.com](http://xn--P1domain-e49l442fl27knba.com)，消息中也没有Route头域，因此通过DNS查询得到负责该域的Proxy的地址并且把消息转发过去。这里P1在转发 前就添加了一个Record-Route头域，里面有一个lr参数，说明P1是一个松散路由器，遵循RFC3261中的路由机制。

> INVITE [sip:callee@domain.com](mailto:sip:callee@domain.com) SIP/2.0 Contact:  
> [sip:caller@u1.example.com](mailto:sip:caller@u1.example.com)  
> Record-Route: sip:p1.example.com;lr

[P2负责域domain.com](http://xn--P2domain-ul8n5373bjba.com)，因此它通过定位服务得到callee@domain.com 对应的设备地址是callee@u2.domain.com，因此用新的URI重写request-URI。消息中没有Route头域，因此它就把该消息转发给request-URI中的URI，转发前它也增加了 一个Record-Route头域，并且也有lr参数。

> INVITE [sip:callee@u2.domain.com](mailto:sip:callee@u2.domain.com) SIP/2.0  
> [Contact:sip:caller@u1.example.com](mailto:Contact:sip:caller@u1.example.com)  
> Record-Route: sip:p2.domain.com;lr  
> Record-Route: sip:p1.example.com;lr

位于u2.domain.com的被叫收到了该INVITE消息，并且返回一个200 OK响应。其中就包括了INVITE中的Record-Route头域。

> SIP/2.0 200 OK  
> Contact: [sip:callee@u2.domain.com](mailto:sip:callee@u2.domain.com)  
> Record-Route: sip:p2.domain.com;lr  
> Record-Route: sip:p1.example.com;lr

被叫此时也就有了自己的路由集：

> (sip:p2.domain.com;lr,sip:p1.example.com;lr)

并且它本次会话的远端目的地址设置为INVITE中Contact中的URI： [caller@u1.example.com](mailto:caller@u1.example.com)，此后被叫在该会话中的请求消息就发到这个URI。同样，被叫在200 OK响应中也携带了自己的联系地址，主叫收到该响应消息后也会把本次会话的远端目的地址设置为： [callee@u2.domain.com](mailto:callee@u2.domain.com)，此后主机在该会话中的请求消息就发到这个URI。  
同样，主叫也有了自己的路由集，只是跟被叫的是反序的：

> (sip:p1.example.com;lr,sip:p2.domain.com;lr)

通话完毕后，我们架设主叫先挂机，则主叫发出BYE请求：

> BYE [sip:callee@u2.domain.com](mailto:sip:callee@u2.domain.com) SIP/2.0  
> Route: sip:p1.example.com;lr,sip:p2.domain.com;lr

可以看到，BYE的Route头域正是主机的路由集构造来的。  
由于p1在第一个Route中，因此BYE首先发给P1。

P1收到该消息后，发现request-URI中的URI不属于自己负责的域，而消息有Route头域，并且第一个Route头域中的URI正是自己，因此删除之，并且把消息转发给新的第一个Route头域中的URI，也就是P2：

> BYE [sip:callee@u2.domain.com](mailto:sip:callee@u2.domain.com) SIP/2.0  
> Route: sip:p2.domain.com;lr

P2收到该消息后，发现request-URI中的URI不属于自己负责的域（[P2负责的是domain.com](http://xn--P2domain-0i5qr02k073cnba.com)，[而不是u2.domain.com](http://xn--u2-yv2cm46g2v2a.domain.com)）， 第一个Route头域中的URI正是自己，因此删除之，此时已经没有Route头域了，因此就转发给了request-URI中的URI。

被叫就会收到BYE消息：

> BYE [sip:callee@u2.domain.com](mailto:sip:callee@u2.domain.com) SIP/2.0

* * *

**路由示例2：**
----------

如果说上面的示例主要关注的是路由流程，那么本示例关注的则是严格路由与松散路由的区别。

**场景：**  
U1->P1->P2->P3->P4->U2  
其中，P3是严格路由的，其余Proxy都是松散路由的，并且4个Proxy都很乐意增加Record-Route头域。

**消息流：**  
我们直接给出了到达被叫的INVITE消息：

> INVITE [sip:callee@u2.domain.com](mailto:sip:callee@u2.domain.com) SIP/2.0  
> Contact: [sip:caller@u1.example.com](mailto:sip:caller@u1.example.com)  
> Record-Route: sip:p4.domain.com;lr  
> Record-Route: sip:p3.middle.com  
> Record-Route: sip:p2.example.com;lr  
> Record-Route: sip:p1.example.com;lr

这中间的其他消息我们就不过问了，直接看一下被叫最后发出的BYE消息大概是什么样子：

> BYE [sip:caller@u1.example.com](mailto:sip:caller@u1.example.com) SIP/2.0  
> Route: sip:p4.domain.com;lr  
> Route: sip:p3.middle.com  
> Route: sip:p2.example.com;lr  
> Route: sip:p1.example.com;lr

因为P4在第一个Route里，因此被叫将BYE消息发给了P4。

P4收到该消息后，[发现自己不负责域u1.example.com](http://xn--u1-yv2c10ulsfmqknv3bq1x2cvfb.example.com)，但是第一个Route头域中的URI正是自己，因此删除之。P4还发现新的第一个 Route头域中的URI是一个严格路由器，因此它把request-URI中的URI添加到最后一个Route的位置，并且将第一个Route“弹出” 并且覆盖原来的request-URI。然后将消息转发给当前的request-URI，也就是P3。

> BYE sip:p3.middle.com SIP/2.0  
> Route: sip:p2.example.com;lr  
> Route: sip:p1.example.com;lr  
> Route: sip:caller@u1.example.com

P3收到该消息后，直接把消息作出如下变换并且发给P2：

> BYE sip:p2.example.com;lr SIP/2.0  
> Route: sip:p1.example.com;lr  
> Route: sip:caller@u1.example.com

P2收到该消息后，发现消息中的request-URI是自己的，因此在进一步处理先首先对消息做如下变换：

> BYE [sip:caller@u1.example.com](mailto:sip:caller@u1.example.com) SIP/2.0  
> Route: sip:p1.example.com;lr

然后，[P2发现自己不负责域u1.example.com](http://xn--P2u1-hb5fk91awlhf5mrv1cuz3a00znba.example.com)，第一个Route中的URI也不是自己的，因此将消息转发给该URI，也就是P1。

P1收到该消息后，[发现自己不负责域u1.example.com](http://xn--u1-yv2c10ulsfmqknv3bq1x2cvfb.example.com)，但是第一个Route头域中的URI正是自己，因此删除之。消息变成下面的样子：

> BYE [sip:caller@u1.example.com](mailto:sip:caller@u1.example.com) SIP/2.0

既然Route头域已经是空，[因此P1把消息发给u1.example.com](http://xn--P1u1-cc1g05nq7tg4dv4yx2ekz1d.example.com)。

- [SIP route与 record_route /SIP路由机制解析](https://blog.csdn.net/jiajiren11/article/details/96337782)
- [SIP中From ,Contact, Via 和 Record-Route/Route](https://www.cnblogs.com/luoyinjie/p/7219380.html)

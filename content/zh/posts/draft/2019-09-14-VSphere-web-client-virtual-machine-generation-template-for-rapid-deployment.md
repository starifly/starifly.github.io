---
title: vSphere web client 虚拟机生成模板快速部署
author: starifly
date: 2019-09-14T12:11:17+08:00
categories: [vsphere]
tags: [vsphere,虚拟机]
draft: true
slug: VSphere-web-client-virtual-machine-generation-template-for-rapid-deployment
---

 <p>一般来说，在 vSphere web client 中如果需要部署虚拟机，则要上传 Images 到存储器，然后通过挂载镜像的方式进行安装。这篇文章主要写通过虚拟机来克隆模板，并且通过模板进而重新生成虚拟机，进而快速部署，下面开始：</p>

<p>一、生成模板</p>

<p>1、打开 vSphere web client <br>
右击已经存在的虚拟机，选择克隆—&gt;克隆为模板</p>

<p><img src="https://img-blog.csdn.net/20160302143532634" alt="这里写图片描述" title=""></p>

<p>2、输入模板名称和选择模板存放位置</p>

<p><img src="https://img-blog.csdn.net/20160302143646541" alt="这里写图片描述" title=""></p>

<p>3、选择存放模板的计算机及检查兼容性</p>

<p><img src="https://img-blog.csdn.net/20160302143758761" alt="这里写图片描述" title=""></p>

<p>4、选择存储器，并且选择虚拟磁盘格式（精简置备）</p>

<p><img src="https://img-blog.csdn.net/20160302143856901" alt="这里写图片描述" title=""></p>

<p>5、生成模板</p>

<p><img src="https://img-blog.csdn.net/20160302144034108" alt="这里写图片描述" title=""></p>

<p>6、查看模板生成状态，大约等待几分钟即可完成</p>

<p><img src="https://img-blog.csdn.net/20160302144142887" alt="这里写图片描述" title=""></p>

<hr>

<p>二、通过模板生成新虚拟机</p>

<p>看到刚才通过虚拟机生成的模板 Centos-6.5-Model</p>

<p><img src="https://img-blog.csdn.net/20160302145405314" alt="这里写图片描述" title=""></p>

<p>1、打开 vSphere web client 主界面，点击新建虚拟机</p>

<p><img src="https://img-blog.csdn.net/20160302145837128" alt="这里写图片描述" title=""></p>

<p>2、创建类型选择为—&gt;从模板部署</p>

<p><img src="https://img-blog.csdn.net/20160302150347857" alt="这里写图片描述" title=""></p>

<p>3、选择部署所需的模板，并勾选创建后打开虚拟机电源</p>

<p><img src="https://img-blog.csdn.net/20160302150450421" alt="这里写图片描述" title=""></p>

<p>4、输入该虚拟机名称及存储位置</p>

<p><img src="https://img-blog.csdn.net/20160302150524203" alt="这里写图片描述" title=""></p>

<p>5、选择存储该虚拟机的主机</p>

<p><img src="https://img-blog.csdn.net/20160302150609724" alt="这里写图片描述" title=""></p>

<p>6、选择存储器</p>

<p><img src="https://img-blog.csdn.net/20160302150654002" alt="这里写图片描述" title=""></p>

<p>7、点击完成即可生成新虚拟机</p>

<p><img src="https://img-blog.csdn.net/20160302150731643" alt="这里写图片描述" title=""></p>

<p>8、查看生成进度</p>

<p><img src="https://img-blog.csdn.net/20160302150813034" alt="这里写图片描述" title=""></p>

<hr>

<p>三、修改所生成的虚拟机主机名、网卡信息，不然该主机有mac和ip地址会冲突，无法进行网络通讯</p>

<p>修改主机名</p>

<pre class="prettyprint"><code class=" hljs makefile">vim /etc/sysconfig/network
<span class="hljs-constant">NETWORKING</span>=yes
<span class="hljs-constant">HOSTNAME</span>=Tomcat-GGL-12
<span class="hljs-constant">GATEWAY</span>=192.168.0.1</code></pre>

<p>修改IP地址</p>



<pre class="prettyprint"><code class=" hljs lasso">vim /etc/sysconfig/network<span class="hljs-attribute">-scripts</span>/ifcfg<span class="hljs-attribute">-eth0</span>
IPADDR<span class="hljs-subst">=</span><span class="hljs-number">192.168</span><span class="hljs-number">.0</span><span class="hljs-number">.12</span>
PREFIX<span class="hljs-subst">=</span><span class="hljs-number">24</span></code></pre>

<p>修改网卡信息 <br>
删除 70-persistent-net.rules，并且reboot，之后将系统重新生成的mac信息进行拷贝，并且进行修改</p>



<pre class="prettyprint"><code class=" hljs lasso">rm /etc/udev/rules<span class="hljs-built_in">.</span>d/<span class="hljs-number">70</span><span class="hljs-attribute">-persistent</span><span class="hljs-attribute">-net</span><span class="hljs-built_in">.</span>rules
reboot

vim /etc/sysconfig/network<span class="hljs-attribute">-scripts</span>/ifcfg<span class="hljs-attribute">-eth0</span>
HWADDR<span class="hljs-subst">=</span><span class="hljs-number">00</span>:<span class="hljs-number">50</span>:<span class="hljs-number">56</span>:<span class="hljs-number">82</span>:<span class="hljs-number">9</span>e:b9

reboot</code></pre>

<p>重启之后网卡即可正常显示</p>

## Reference

- [vSphere web client 虚拟机生成模板快速部署](https://blog.csdn.net/wanglei_storage/article/details/50780328)

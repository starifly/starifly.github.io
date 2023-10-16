---
title: python爬取加密货币交易所公告
author: starifly
date: 2023-01-26T16:29:15+08:00
lastmod: 2023-01-26T16:29:15+08:00
categories: [python]
tags: [python]
draft: true
slug: python-get-crypto-exchange-announcement
---

## 爬取芝麻开门公告

```python
import requests
import re
if __name__=='__main__':
    target = 'https://www.gateio.ch/cn/articlelist/ann'
    req = requests.get(url=target)
    html = req.content
    html_doc = str(html, 'utf-8')
    content = re.findall(r'<div class="entry">(.*?)<div style="clear:both"></div>', html_doc,re.S)
    a = len(content)
    n = 0
    while (n < a):
        href = re.findall(r'<a href="(.*?)"',content[n], re.S)[0]
        href = 'https://www.gateio.ch%s' % href
        title = re.findall(r'<h3>(.*?)</h3>',content[n], re.S)[0]
        time = re.findall(r'<span id="updated">(.*?)</span>', content[n],re.S)[0]
        print(time,title,href)
        #print("{:\u3000<18} {:\u3000<55} {:<20}".format(time, title, href)) 
        n=n+1
```

## 爬取币安币币交易公告

```python
import requests
import re
import time
from datetime import datetime
    
if __name__=='__main__':
    while 1:
        target = 'https://www.binancezh.co/zh-CN/support/announcement/c-48?navId=48'
        req = requests.get(url=target)
        html = req.content
        html_doc = str(html, 'utf-8')
        content = re.findall(r' class="css-1ej4hfo">(.*?)</a>', html_doc,re.S)
        href = re.findall(r'<a data-bn-type="link" href="/zh-CN/support/announcement/(.*?)"', html_doc,re.S)
        a = len(content)
        n = 0

        print(time.strftime('%Y-%m-%d %H:%M:%S',time.localtime(time.time()))) 
        while (n < a):
            print(content[n], '        ', 'https://www.binancezh.co/zh-CN/support/announcement/' + href[n])
            n=n+1
        time.sleep(60)
        print("\n")
```

## Reference

- [Python爬虫之爬取静态网站——爬取各大币交易网站公告（一）](https://blog.csdn.net/qq_41813030/article/details/82916119)

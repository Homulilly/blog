---
title: Python3 抓取 HTML 网页
date: 2016-08-20 23:51:43
categories:
  - Learn
tags:
  - Python
---

Python 还在学习中，突然想拖一个网站上的图片，因为有点多，所以纯手动太麻烦，就试试能不能用一下。毕竟现在虽然看不懂模块的代码，但是会用就可以了。于是把中间的过程做一下笔记。  

一开始知道，可以通过 urllib.request 模块获取一个链接中的内容。  
```python
import urllib.request
data = urllib.request.openurl('$url').read()
```
此时 data 是二进制数据，如果是图片内容的话，直接写入文件就可以了，网页的话可以通过 `html_code = data.decode('utf-8')` 转换为 string 。  
<!--more-->
是这样不错，不过我想下图的那个网站的网址不知道为什么通过 `urllib.request.openurl` 的方式总是提示 404 错误。随后想通过直接下载 html 文件的方式，可能是我读取时没处理好，在解析 html 时还是出现错误。  

## Pycurl
随后搜索了一下 Python 的 curl 模块，果然是有的，虽然用法没怎么看懂，但是官网给的 [示例](http://pycurl.io/docs/latest/quickstart.html) 正好就拿来用了。  
```python  
# -*- coding: utf-8 -*-
"""
GET HTML FROM URL USE Pycurl
"""

import pycurl
from io import BytesIO

def Get_html(url):
    buffer = BytesIO()
    c = pycurl.Curl()
    c.setopt(c.URL, url)
    c.setopt(c.WRITEDATA, buffer)
    c.perform()
    c.close()

    body = buffer.getvalue()
    html_code = body.decode('utf-8')

    return html_code
```

## Http.cilent
后来搜索 `urllib.request.openurl` 为啥会提示 404 ，发现使用 [http.request](https://docs.python.org/3/library/http.client.html#examples) 也可以。

```python  
# -*- coding: utf-8 -*-
"""
GET HTML FROM URL USE http.client
"""

import http.client

def Get_html(url):
    host = url.split('/')[2]
    url_content = url[url.index(host)+len(host):]

    conn = http.client.HTTPConnection(host)
    conn.request("GET", url_content)
    r = conn.getresponse()
    """
    print(r.status,r.reason)
    """
    data = r.read()
    html_code = data.decode('utf-8')

    return html_code
```
嘛 不过感觉，都差不多的样子（

## Requests
最后的时候，终于发现了最简单的方法，导入 `requests` 模块。
```python
# -*- coding: utf-8 -*-
import requests
response = requests.get(url)
html_code = response.text
```

## 解析 HTML
由于需要分析的网页并不复杂，所以搜索到了一个很简单的 [例子](http://blog.csdn.net/hxsstar/article/details/17241709) 就可以使用了。  
下面这段就是返回 html_code 中所有 img 标签的 'scr = ' 的链接，然后根据情况稍加过滤就可以了。    
```python
"""
GET LINK FROM HTML IMG TAG
"""
from html.parser import HTMLParser  
class MyHTMLParser(HTMLParser):   
    def __init__(self):   
        HTMLParser.__init__(self)   
        self.links = []   
    def handle_starttag(self, tag, attrs):   
        #print "Encountered the beginning of a %s tag" % tag   
        if tag == "img":   
            if len(attrs) == 0:   
                pass   
            else:   
                for (variable, value) in attrs:   
                    if variable == "src":   
                        self.links.append(value)   
```


## BeautifulSoup
最后发现了 BeautifulSoup 这个，解析 HTML 时使用起来更加的方便。 <s>（也更好看懂，而且还有中文）</s> 。  

[BeautifulSoup 文档](https://www.crummy.com/software/BeautifulSoup/bs4/doc.zh/)  

使用 Python的内置标准库解析 HTML ：
```python
import bs4
soup = bs4.BeautifulSoup(html_code, "html.parser")
```

---

## 其他

- Python 输出 RSS2.0 标准的 xml 文件 ： [PyRSS2Gen](http://www.dalkescientific.com/Python/PyRSS2Gen.html)

- 记录 Crontab 的日志  
#修改rsyslog  
`sudo vim /etc/rsyslog.d/50-default.conf`  
cron.*              /var/log/cron.log #将cron前面的注释符去掉
#重启rsyslog  
`sudo  service rsyslog  restart`  
#查看crontab日志  
`less  /var/log/cron.log`  

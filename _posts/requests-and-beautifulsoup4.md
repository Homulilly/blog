---
title: Requests and Beautifulsoup4
date: 2016-08-30 20:38:54
categories:
  - Learn
tags:
  - Python
  - Requests
  - BeautifulSoup
---
虽然还在学习中，不过最近有些需求使用了 Requests 和 BS4 这两个模块，只是一些简单的功能，记录一下。  
Requests 应该是安装了 pip 后自动就安装了， bs4 还需要手动安装一下。  

### 安装
```
pip install requests, bs4
```
### 基本的用法  
```python
import bs4
import requests

url = 'http://example.com'
resp = requests.get(url)
soup = bs4.BeautifulSoup(resp.text, 'html.parser')
```
<!--more-->
### 一些扩展的用法  
#### Requests
- 设置 Cookie：`cookies = {'name':'value'}`
- 设置请求 header： `headers = {'user-agent': 'value'}`
- 设置代理：[Socks proxy](http://docs.python-requests.org/zh_CN/latest/user/advanced.html#socks)    
```python
proxies = {
    "http": "http://127.0.0.1:8080",
    "https": "http://127.0.0.1:8080",
}
```

请求内容：
```python
resp = requests.get(url, cookies = cookies, headers = headers, proxies = proxies)
```

resp.text -> 文本内容  
resp.content -> 二进制内容，可以用获取文件、图片  
resp.status_code -> 服务器响应代码， `resp.raise_for_status()` 可以用于异常代码时抛出异常  
resp.cookies -> Cookies  
resp.encoding -> 网页编码，默认自动检测，可以手动指定：`r.encoding = 'utf-8'`  

#### BeautifulSoup4  
HTML 解析器 ： `html.parser` 使用 Python 内置 HTML解析器，无需安装，速度适中， `lxml` 使用 lxml HTML 解析器 ，速度快，需要额外安装 lxml 模块，可以解析 XML 。具体可以参考 [文档](https://www.crummy.com/software/BeautifulSoup/bs4/doc.zh/#id9)。  

`soup.find('tag',attr = 'value')` 返回第一个匹配内容，匹配 class 时要写 `class_ = ''`   
`soup.find_all('tag',attr = 'value')` 返回列表，包含所有匹配的内容  
`soup.select('.class_value')` 根据 css 选择器返回列表，包含所有匹配内容  

对象的一些操作：  
`.get('attr')`  
`.string`  

(´・ω・\`)，到这突然发现 BS4 的用法，都在文档里面，并没有什么我能去精炼总结的地方（  

### 参考文档  
两个模块都有提供中文文档：  
- Requests: http://docs.python-requests.org/zh_CN/latest/
- BeautifulSoup4: https://www.crummy.com/software/BeautifulSoup/bs4/doc.zh/

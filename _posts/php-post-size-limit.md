---
title: PHP 单个POST上传大文件
date: 2019-01-19 20:11:20
tags:
    - 日常掉坑
    - cURL
    - PHP
---

在写新的博客系统，文件上传的时候出了点问题，大文件直接无法上传。

<!-- more -->

## 曲折的审计代码

博客系统采用的是自定义的 `XRPC` 协议，使用 `JSON` 和 `POST` 做API调用，我采用CURL做为实现，因此审计的路线：

- CURL 是否上传了文件
- CURL 是否完成了想要的功能

以上两个步骤，涉及点：

1. CURL 上传文件，之前是通过 `@文件名` 标识文件，后面更新了，采用 `CURLFile` 类来包装
2. CURL 检查包发送状态，我这里采用 `Fiddler` 来做抓包，给 `cURL` 设置代理解决，发现包内容正常

> cURL 设置代理

```php
curl_setopt($curl, CURLOPT_PROXY, '127.0.0.1'); //代理服务器地址   
curl_setopt($curl, CURLOPT_PROXYPORT, 8888 ); //代理服务器端口
```

以上是代码的坑，到了这里，发现`POST`包很正常，但是 `PHP` 端没法解析，于是检擦PHP配置。

- `upload_max_filesize` = 2M 文件上传最大大小2M
- `post_max_size` = 8M  HTTP的POST包最大8M

以上， `9.8M` 的文件，总算上传了。
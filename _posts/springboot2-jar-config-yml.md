---
title: SpringBoot 2 中的配置文件运行时指定
date: 2018-08-09 14:37:30
tags:
    - Java
    - SpringBoot 2.0
categories:
    - 技术笔记
---

因为一个配置文件，搞了我几天，记录一下

<!-- more -->


## Springboot2 配置文件配置顺序

在 Springboot2中，配置文件配置文件的加载和覆盖顺序如下

![](springboot2-jar-config-yml/1.png)

也就是说，`application.yml` 配置优先于 `application-xx.yml` 加载，并且 `application-xxx.yml` 的配置项可以覆盖 `application.yml` 的配置项

**就是这个坑！！！！**

## 运行指定配置

在 Springboot2 的jar包直接运行，用命令参数 `--spring.profiles.active` 指定激活的配置项

> 指定使用 application-dev.yml 配置

```
java -jar xxx.jar --spring.profiles.active=dev
```

这样加载的配置会覆盖 `application.yml` 的配置文件。


## 运行指定配置文件

在 Springboot2 的jar包直接运行， 用命令参数 `--spring.config.location` 指定使用外部配置文件，**不会覆盖jar包中的`application.jar`配置项，或者说就没加载！** 昂，然后我就被坑了，只在外部配置文件中指定部分配置，然后凉了，mybatis-plus 没指定 mapper文件，GG

---
title: Suda - 13 自动加载类规则
date: 2018-08-22 21:58:14
tags:
    - Suda
    - PHP
    - PSR4
categories:
    - 基础教程
    - Suda框架技能使用
---

在PHP中，我们使用 `require`、`require_once`、`include`、`include_once` 来引用其他的PHP文件，但是在框架中，我们很少看到这样的代码，那么，框架是如何找到我们想要的类的？

<!-- more -->

## PSR4 加载规范

在PSR4中，有几个概念

### 命名空间(namespace)

起初实在C++中兴起的，在Java中体现为包名(package)，在C#中好像也是命名空间，PHP中也是，是为了解决命名冲突的问题而产生的比如A写了一个类叫 MyClass。B也写了一个MyClass类，我们在用的时候怎么区分？加个前缀啊，A写的类就叫 A\MyClass，B写的类就叫B\MyClass，完美的解决了。

### 根命名空间(root namespace)

继续上一个例子，如果A再写一个类，叫Student，一般按照习惯，我们自己写的东西都会放到自己的地方，那么，通常来说A的类Student就是A\Student，那么对于 MyClass 和 Student 来说，A就是相当于他们的父节点也就是树的父节点，即为根

### 查找目录

一般来说，我们A写的代码肯定放到A的区域里，体现在源文件中的话，一般是用文件夹来表示，那么文件夹结构应该是这样的：

```
源码文件夹X
    |___A 
    |    |_ MyClass.php
    |    |_ Student.php
    |___B
        |_ MyClass.php
```

在PSR4规则中，我们注册了一个文件夹，比如 `src` 文件夹，作为查找目录，那么，我们找类 `A\Student` 的时候会去各种源码文件夹找
`A\Student.php` 文件，在源码文件夹X中找到了就自动include。也就避免我们自己来include文件了，毕竟很难找。


### 命名空间前缀 （namespace prefix）

如果说我们的命名空间很长，比如：`very\long\name\space\A\MyClass` 我们一般会选择简化一下，因为我是A嘛，我写的东西肯定放在了A文件夹的下面，那么类名的话肯定是这样的 `very\long\name\space\A\Student` 但是我又不想创建一串文件夹 very啥的啥的，那么，我们假设，源码文件夹X本省表示一个命名空间：`very\long\name\space\` 那么，用这个做前缀。则

```
源码文件夹X = very\long\name\space
    |___A  = very\long\name\space\A
    |    |_ MyClass.php = very\long\name\space\A\MyClass
    |    |_ Student.php = very\long\name\space\A\Student
    |___B
        |_ MyClass.php = very\long\name\space\B\MyClass
```

以上就基本上是PSR4规则的一个简要介绍。


### Suda 的源码自动加载

在Suda框架中，只支持PSR4规则，一般配置在文件 `manifast.json` 或者 `module.json` 文件中，如：

```json
{
    "import": {
        "share": {
            "dxkite\\usercenter\\client": "share"
        },
        "src": {
            "dxkite\\usercenter\\client\\response": "src"
        }
    }
}
```

这里就懒得创建一堆文件夹了，就把源码下面的 src 作为 `dxkite\usercenter\client\response` 的一个根命名空间了，如果需要找类 `dxkite\usercenter\client\response\IndexResponse` 就去 src 目录下找 `IndexResponse.php` 文件，**注意哦，命名空间提倡小写。**


### share 与 src  

在 `import` 规则中我们可以看到 `share` 规则和 `src` 规则，具体定义如下：

当模块被使用的时候（就是运行了模块的某一个响应类，比如 `dxkite\usercenter\client\response\IndexResponse`）那么`src` 规则被激活，自动查找类的时候会包含 `src` 下面的文件，**但是！** 在非激活状态，比如运行其他模块的时候，只会把`share`规则注册进查找目录，用来查找类文件，也就是说，`share` 是其他模块可以访问的类，`src` 是只能自己用的类。






---
title: 'PHP函数的默认行为'
date: 2018-07-19 22:14:18
tags:
    - 'code audit'
    - PHP
    - CTF
    - '网络安全'
categories:
    - '技术分享'
---

函数的默认行为有时候会被忽略掉，但是造成的安全问题也是有点东西的；本次讲三个函数默认行为：`class_exists`、`in_array`、`preg_quote`。

<!-- more -->

## class_exists

> bool class_exists ( string $class_name [, bool $autoload = true ] )

函数功能：判断类是否存在
默认行为：调用自动加载函数 `__autoload` 或者调用 `spl_autoload_register` 注册过的自动加载函数。

*函数代码*

```php
function my_autoload(string $name){
    echo 'call --> '.__FUNCTION__.'('.$name.')'."\r\n";
}
spl_autoload_register('my_autoload');
echo 'with default = true'."\r\n";
var_dump(class_exists('Some')) ;
echo 'with default = false'."\r\n";
var_dump(class_exists('Some',false));
```

*运行结果*


## in_array

> bool in_array ( mixed $needle , array $haystack [, bool $strict = FALSE ] )

函数功能：判断值是否在数组中
默认行为：非严格判断是否相同（自带类型装换）

*函数代码*

```php
# 数字绕过
var_dump(in_array('5_val',range(0,10)));
# 黑名单绕过
var_dump(in_array('5.php',['php','png','jpg']));
var_dump(in_array('5_val',range(0,10),true));
```

## preg_quote

> string preg_quote ( string $str [, string $delimiter = NULL ] )

函数功能：转义每个正则表达式语法中的字符
默认行为：只转义 `. \ + * ? [ ^ ] $ ( ) { } = ! < > | : -`

*函数代码*

```php
preg_match('/'.preg_quote('x./.').'/i','sox./text');
preg_match('/'.preg_quote('x./.','/').'/i','sox./text');
```

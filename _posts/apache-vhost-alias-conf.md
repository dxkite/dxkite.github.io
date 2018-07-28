---
title: Apache 实现一域一文件夹的虚拟主机配置
date: 2018-07-27 07:01:21
tags:
    - Apache
categories:
    - 技术分享
---

在开发网站的过程中，使用Apache做为主要服务器的时候一般都是 `localhost` 访问网站，访问的时候只能一次访问一个网站域，在本文中将讲解如何配合 `hosts` 文件来控制访问的网站程序

<!-- more -->

## Apache 实现虚拟主机根目录

**前置条件**：虚拟主机目录必须在主机目录下
**依赖模块**：`mod_vhost_alias`

格式说明 （`VirtualDocumentRoot`） 语法格式：`%-N+.-M+`

| 参数  | 说明  |
|-------|-------|
| %% | 百分号转义 |
| %N | N为任意数字，负号表示从后面开始 |
| %N.M | M表示截取N的子字符 |
| + | 表示包含当前位置以及以后的位置 |

具体实现要求： 

1. 访问 `www.dxkite.org` 时加载 `${DocumentRoot}/dxkite.org/www/public` 下的网站程序
2. 访问 `dxkite.org` 时加载 `${DocumentRoot}/dxkite.org/www/public` 下的网站程序
3. 访问 `$1.org` 时加载 `${DocumentRoot}/$1/www/public` 下的网站程序
4. 访问 `$1.$2.org` 时加载 `${DocumentRoot}/$2/$1/public` 下的网站程序
5. 访问 `localhost` 或者 ip 定位到 `${DocumentRoot}/localhost/public` 下的网络程序

不难看出，要求 1 是要求 3的特殊情况

> dxkite.org --> 导向本地开发网站

## 域名分割例子 

| 域名 | 参数 | 结果 |
|---|--------|-------|
| test.dxkite.cn | %0 |  test.dxkite.cn |
| test.dxkite.cn | %1 |  test |
| test.dxkite.cn | %2 |  dxkite |
| test.dxkite.cn | %2+ |  dxkite.cn |
| test.dxkite.cn | %1.2 |  e |


## 具体规则 

### 规则 5

```xml
<VirtualHost _default_:80>
    DocumentRoot "D:\Server\localhost\public"
    ErrorLog "D:\Server\logs\localhost.err"
    CustomLog "D:\Server\logs\localhost.log" common
    <Directory "D:\Server\localhost\public">
        Options -Indexes +FollowSymLinks +ExecCGI
        AllowOverride All
        Order allow,deny
        Allow from all
        Require all granted
    </Directory>
</VirtualHost>
```

### 规则2

规则3无法目前通过其他方式实现（我暂未发现），暂实现特例规则2
参考实现方案：

- 使用代理
- 使用URL重写

```xml
<VirtualHost *:80>
    ServerName dxkite.org
    VirtualDocumentRoot "D:\Server\vhost\dxkite.org\www\public" 
    ErrorLog "D:\Server\logs\vhost_access.err"
    CustomLog "D:\Server\logs\vhost_access.log" vhost_common
    <Directory "D:\Server\vhost\%2+\%1\public">
        Options +Indexes +FollowSymLinks
        AllowOverride All
        Order allow,deny
        Allow from all
        Require all granted
    </Directory>
</VirtualHost>
```
### 规则 3

如果要添加其他域名的话，就在后面加就可以了

```xml
<VirtualHost *:80>
    ServerAlias dxkite.org atd3.org
    VirtualDocumentRoot "D:\Server\vhost\%0\www\public" 
    ErrorLog "D:\Server\logs\vhost_access.err"
    CustomLog "D:\Server\logs\vhost_access.log" vhost_common
    <Directory "D:\Server\vhost\%0\www\public">
        Options +Indexes +FollowSymLinks
        AllowOverride All
        Order allow,deny
        Allow from all
        Require all granted
    </Directory>
</VirtualHost>
```

### 规则 1，4

**注意**: * 表示匹配所有、 ? 匹配单个字符 (官网没找到解释，看源码)

```xml
<VirtualHost *:80>
    ServerAlias *
    VirtualDocumentRoot "D:\Server\vhost\%2+\%1\public" 
    ErrorLog "D:\Server\logs\vhost_access.err"
    CustomLog "D:\Server\logs\vhost_access.log" vhost_common
    <Directory "D:\Server\vhost\%2+\%1\public">
        Options +Indexes +FollowSymLinks
        AllowOverride All
        Order allow,deny
        Allow from all
        Require all granted
    </Directory>
</VirtualHost>
```



> **注意** 虚拟主机配置规则有顺序要求
>> **参考文献**
>> http://httpd.apache.org/docs/2.0/en/vhosts/
>> http://httpd.apache.org/docs/2.2/mod/mod_rewrite.html
>> http://httpd.apache.org/docs/current/mod/mod_proxy.html

**虚拟主机参考源码**
> httpd-2.4.34/server/vhost.c:894~925
```c
/* return 1 if host matches ServerName or ServerAliases */
static int matches_aliases(server_rec *s, const char *host)
{
    int i;
    apr_array_header_t *names;

    /* match ServerName */
    if (!strcasecmp(host, s->server_hostname)) {
        return 1;
    }

    /* search all the aliases from ServerAlias directive */
    names = s->names;
    if (names) {
        char **name = (char **) names->elts;
        for (i = 0; i < names->nelts; ++i) {
            if (!name[i]) continue;
            if (!strcasecmp(host, name[i]))
                return 1;
        }
    }
    names = s->wild_names;
    if (names) {
        char **name = (char **) names->elts;
        for (i = 0; i < names->nelts; ++i) {
            if (!name[i]) continue;
            if (!ap_strcasecmp_match(host, name[i]))
                return 1;
        }
    }
    return 0;
}
```

>httpd-2.4.34/server/util.c:211~234
```c

AP_DECLARE(int) ap_strcasecmp_match(const char *str, const char *expected)
{
    int x, y;

    for (x = 0, y = 0; expected[y]; ++y, ++x) {
        if (!str[x] && expected[y] != '*')
            return -1;
        if (expected[y] == '*') {
            while (expected[++y] == '*');
            if (!expected[y])
                return 0;
            while (str[x]) {
                int ret;
                if ((ret = ap_strcasecmp_match(&str[x++], &expected[y])) != 1)
                    return ret;
            }
            return -1;
        }
        else if (expected[y] != '?'
                 && apr_tolower(str[x]) != apr_tolower(expected[y]))
            return 1;
    }
    return (str[x] != '\0');
}
```
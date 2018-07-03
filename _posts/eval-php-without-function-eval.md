---
title: '特殊方式运行PHP代码'
date: 2018-07-01 16:08:47
tags:
    - '网络安全'
    - PHP
categories:
    - '技术分享'
---

之前一直在想一个问题，怎么在不使用`eval`函数的前提下执行php代码，因为不管是一句话还是其他的骚操作好像都要通过`eval`函数来执行PHP代码（如果有其他方式，请指出吧，我表示就知道一个），现在的一句话代码也是如此，那么如何绕过这个东西来执行代码？用PHP也有几年了，今天突然灵光一闪，我们来看下面的代码：

```php
include 'http://www.test.com/xx.php'
```

<!--more-->

如上代码可以通过包含远程HTTP文件来执行PHP文件，也就是执行远程代码，我们在玩一句话的时候基本上都不使用这些东西来操纵，如果我们设置了环境变了``不过也提供了一些思路，现在们来了解下这串代码所包含的含义，先看下我如今的PHP配置项

```ini
;;;;;;;;;;;;;;;;;;
; Fopen wrappers ;
;;;;;;;;;;;;;;;;;;

; Whether to allow the treatment of URLs (like http:// or ftp://) as files.
; http://php.net/allow-url-fopen
allow_url_fopen =Off

; Whether to allow include/require to open URLs (like http:// or ftp://) as files.
; http://php.net/allow-url-include
allow_url_include = Off
```

在默认的情况下，好像可以通过`include`来包含远程文件，现在我们把这两个都关了，远程文件打开和远程文件包含都禁止使用，也就是说，上面第一串代码也就算是失效了

## PHP的包装器

在上面的配置文件中，我们看到了一个概念 `Fopen Wrapper` , 在官方的参考文档中的位置：http://php.net/manual/en/class.streamwrapper.php 在这里中我们可以看到一个标准的包装器的介绍，那么，我们实现一个包装器来执行代码怎么样？？

> **开发任务**
实现一个包装器，使其出现如下代码的时候输出 `'hello, dxkite'`

```php
include 'hello://dxkite'
```

## HelloStream 实现

对于一个流来说，按照官方的文档，必须实现如下的类方法：

```php
streamWrapper {
    /* Properties */
    public resource $context ;
    /* Methods */
    __construct ( void )
    __destruct ( void )
    public bool dir_closedir ( void )
    public bool dir_opendir ( string $path , int $options )
    public string dir_readdir ( void )
    public bool dir_rewinddir ( void )
    public bool mkdir ( string $path , int $mode , int $options )
    public bool rename ( string $path_from , string $path_to )
    public bool rmdir ( string $path , int $options )
    public resource stream_cast ( int $cast_as )
    public void stream_close ( void )
    public bool stream_eof ( void )
    public bool stream_flush ( void )
    public bool stream_lock ( int $operation )
    public bool stream_metadata ( string $path , int $option , mixed $value )
    public bool stream_open ( string $path , string $mode , int $options , string &$opened_path )
    public string stream_read ( int $count )
    public bool stream_seek ( int $offset , int $whence = SEEK_SET )
    public bool stream_set_option ( int $option , int $arg1 , int $arg2 )
    public array stream_stat ( void )
    public int stream_tell ( void )
    public bool stream_truncate ( int $new_size )
    public int stream_write ( string $data )
    public bool unlink ( string $path )
    public array url_stat ( string $path , int $flags )
}
```

在刚开始测试的时候，发现只需要实现用到的方法即可，不需要实现所有的功能，实现如下接口的功能即可：

```php
interface  IncludeStreamWrapper
{
    function stream_open($path, $mode, $options, &$opened_path);
    function stream_read($count);
    function stream_tell();
    function stream_eof();
    function stream_seek($offset, $whence);
    function stream_stat();
}
```

按照这个写法，我们实现一个我们自己的 `HelloStream` 类：

```php
class HelloStream
{
    protected $position;
    protected $code;

    public function stream_open($path, $mode, $options, &$opened_path)
    {
        $url = parse_url($path);
        $name = $url["host"];
        $this->code = "<?php echo 'hello, {$name}';";
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
}
```

## 注册 HelloStream

通过函数 `stream_wrapper_register` 注册包装器

```php
require_once __DIR__ .'/HelloStream.php';

stream_wrapper_register('hello', HelloStream::class);
include 'hello://dxkite';
```

运行这串代码我们可以看到如下的结果

```
PS D:\WorkSpace\PHP\Stream> php test.php
hello, dxkite
```

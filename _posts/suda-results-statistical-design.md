---
title: Suda - 07 成绩统计系统 - 项目设计
date: 2018-08-05 07:15:56
tags:
    - Suda
    - PHP
categories:
    - 基础教程
    - Suda框架技能使用
---

本次开始将用一个例子来集合前面我们学习过的东西 - 成绩统计，在开始之前，我们创建一个新项目，之前的老东西就不用了。

<!-- more -->

准备操作：

创建一个新项目，访问后，把 app 目录下的 `/data`  目录删了， `modules` 下面的 app 模块页删了，创建一个 `statistical` 模块，目录结构暂时如下：

![](suda-results-statistical-design/1.png)

**配置文件**：

```json
{
    "name": "dxkite/statistical",
    "namespace": "dxkite/statistical",
    "authors": [{
        "name": "dxkite",
        "email": "dxkite@qq.com"
    }],
    "import": {
        "share": {
            "dxkite\\statistical": "share"
        },
        "src": {
            "dxkite\\statistical\\response": "src"
        }
    },
    "discription": "DXkite的成绩统计模块",
    "require": {},
    "version": "1.0.0"
}
```
配置好模块名，这里我的模块名是 `dxkite/statistical` 模块名可以任意命名，但是格式必须是 `空间/模块` 

命名空间这里我是用 `dxkite\statistical` 而不是之前的 `cn\atd3` 了，这些东西都可以自定义

路由文件：清空路由文件

```json
{}
```

**启用模块**:

模块一般都是添加上去就可以用，不过我默认指定了可用模块

```json
{
    "name": "suda-test-app",
    "namespace": "cn\\atd3",
    "version": "1.0-dev",
    "application": "suda\\core\\Application",
    "modules": ["suda", "app"],
    "reachable": ["app"],
    "language": "zh-CN",
    "url": {
        "mode": 0,
        "beautify": true,
        "rewrite": true
    }
}
```

源配置文件表示：启用 suda 和 app 模块，其中 app  模块可以被访问到，但是现在我们把 app 模块删了，要改成：启用 suda 和 statistical 模块，其中 statistical  模块可以被访问到，顺便把app的名字改下：

```json
{
    "name": "dxkite-statistical-app",
    "namespace": "dxkite\\statistical",
    "version": "1.0-dev",
    "application": "suda\\core\\Application",
    "modules": ["suda", "statistical"],
    "reachable": ["statistical"],
    "language": "zh-CN",
    "url": {
        "mode": 0,
        "beautify": true,
        "rewrite": true
    }
}
```

## 案例分析

在成绩统计的时候，我们会用到那些东西？学号、姓名、成绩。其中对于我们来说，学号是唯一的。在表设计阶段也就可以把学号作为主键。本次在统计的时候我还想添加一个功能，同时创建多个统计单，支持设置统计类型。我本次选择将统计类型作为一个配置存储在配置文件 `config/statistic.json` 中。


### 统计类型设计

一般来说，统计类型一般为 `number` 、 `string` 、 `text` 分别对应 `input ="number"` 、`input="text"`、`textarea`。

### 表单标题

一般一个统计页面会有一个标题，所以在配置文件中也要体现出来

### 整体配置文件格式

看配置和逻辑，不难看出表id和学号两个才能构成一个主键

```json
{
    "table_id_1": {
        "name": "学生XXX统计表",
        "fields": {
            "math": {
                "name": "数学成绩",
                "type": "number"
            },
            "english": {
                "name": "英语成绩",
                "type": "number"
            }
        }
    },
    "table_id_2": {
        "name": "学生XXX统计表-2",
        "fields": {
            "math": {
                "name": "数学成绩",
                "type": "number"
            },
            "english": {
                "name": "英语成绩",
                "type": "number"
            }
        }
    }
}
```

## 数据表设计

根据现有的分析，数据表设计大概长这样

| 表段 | 类型 | 备注 | 说明 |
|-----|------|------|------|
| student_id | varchar | PK | 学号，字符串类型，长度暂时不定 |
| table_id | varchar | PK | 表id，字符串类型，长度暂时不定 |
| name | varchar | | 学生姓名|
| data | text | | 数据域，为了存储不定长的数据，这里我们要处理 |

根据如上的表，我们的表文件PHP具体描述应该大致如下

```php
<?php
namespace cn\atd3\table;
class StatisticTable extends \suda\archive\Table {

    public function  __construct(){
        // 数据表名
        parent::__construct('statistic');
    }

    protected function onBuildCreator($table){
        $table->fields(
            $table->field('student_id','varchar',80)->primary(),
            $table->field('table_id','varchar',80)->primary(),
            $table->field('name','varchar',80),
            $table->field('data','text')
        );
        return $table;
    }
}
```

### 字段处理

Suda 框架提供对数据的预处理，在数据输入前和获取前都可以处理一下，处理对应到字段：

```php
<?php
namespace cn\atd3\table;
class StatisticTable extends \suda\archive\Table {

    public function  __construct(){
        // 数据表名
        parent::__construct('statistic');
    }

    protected function onBuildCreator($table){
        $table->fields(
            $table->field('student_id','varchar',80)->primary(),
            $table->field('table_id','varchar',80)->primary(),
            $table->field('name','varchar',80),
            $table->field('data','text')
        );
        return $table;
    }

    protected function _inputDataField($data){
        return serialize($data);
    }

    protected function _outputDataField($data){
        return unserialize($data);
    }
}
```

其中 `_input字段名Field` 表示处理输入的字段，`_ouput字段名Field`表示处理输出的字段，字段名首字母大写，
这里，在输入的时候使用 `serialize` 将任意类型的数据转换成字符串存储在数据库中，`unserialize` 在我们读取的时候转换回来。

## 路由设计

我们需要几个页面：

- index 页面，显示可以输入的表格数量，提供一个下载链接
- table 页面，显示输入的表格，以及输出结果
- download 页面，提供下载表格的功能


## 作业

实现所以路由和表格

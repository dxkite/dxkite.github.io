---
title: '在Hexo博客系统中引用图片与资源'
date: 2018-07-04 20:16:17
tags:
    - Hexo
categories:
    - '杂记分享'
---

因为 `Hexo` 的模块 `hexo-asset-image` 只支持图片装换，而且图片装换不太友好，根据原先的代码，我修改以后PR提交给原作者，原作者也不理我，所以新开一个坑用来处理这个问题，使用非常简单，按照常规`Markdown`方式引用即可，因为 `Hexo` 系统的原因，所有的资源都会归档到 `_posts` 下的同名文件夹下，使用的时候开启 `post_asset_folder` 配置，然后直接引用资源即可。

**实例Markdown**

```markdown
# Test Assets

[archive](markdown-post/archive.zip)

![image](markdown-post/image.png)

```

文件夹结构

```shell
markdown-post
├── archive.zip
└── image.png
markdown-post.md
```

这样不管在编辑时还是处理时，都会保证资源引用的正确性，写起来也方便，不需要预览或者开启服务

## 模块安装

使用如下命令安装即可

```shell
npm install hexo-assets --save
```
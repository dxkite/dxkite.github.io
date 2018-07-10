---
title: '02 贪吃蛇 - 加载图片与显示：显示图片'
date: 2018-07-06 10:04:52
tags:
    - Python
    - pygame
categories:
    - '基础教程'
    - 'Python贪吃蛇教程'
---

上一篇文章我们显示了一个文字：hello world，这篇文章我们来显示一波图片
<!-- more -->

首先准备一张图片，这张图是贪吃蛇游戏中的食物

![](snake-display-image/0.png)

*贪吃蛇用：食物*

## 基础：加载并显示图片

```python
import pygame
# 1. 先初始化游戏框架
pygame.init()
# 2. 创建一个窗口
screen = pygame.display.set_mode((100 , 100 ),0,32)
# 2.1 屏幕填充白色
screen.fill((0xff,0xff,0xff))
# 2.2 加载图片
image = pygame.image.load('0.png')
# 2.3 贴图图片
screen.blit(image,(0,0))
# 3. 刷新显示窗口
pygame.display.flip()  
# 假装进行了什么操作 （延时2秒）
pygame.time.wait(2000)
# 4. 退出框架
pygame.quit()
```

从代码可以看到，使用函数 `pygame.image.load` 用于从文件中加载一个图片，返回一个Surface参数，加载完成后，使用blit方法将图片贴在了`screen`表面上并渲染

![2-1](snake-display-image/2-1.png)

## 控制图片位置

`Surface.blit` 方法用于贴图

> blit(source, dest, area=None, special_flags = 0) -> Rect

| 参数 | 说明 |
|-----|-------|
| source | 显示的Surface |
| dest | 显示的目标区域 (x,y) |
| area | 被显示的区域（雪碧图） (x,y,w,h) |
| special_flags | pygame 1.8的东西。处理像素点 |

```python
import pygame

# 1. 先初始化游戏框架
pygame.init()
# 2. 创建一个窗口
screen = pygame.display.set_mode((100 , 100 ),0,32)
# 2.1 屏幕填充白色
screen.fill((0xff,0xff,0xff))
# 2.2 加载图片
image = pygame.image.load('0.png')
# 2.3 贴图图片
screen.blit(image,(10,10))
# 3. 刷新显示窗口
pygame.display.flip()  
# 假装进行了什么操作 （延时2秒）
pygame.time.wait(2000)
# 4. 退出框架
pygame.quit()
```

显示效果

![2-2](snake-display-image/2-2.png)

## 使用参数 area 裁剪图片

```python
import pygame

# 1. 先初始化游戏框架
pygame.init()
# 2. 创建一个窗口
screen = pygame.display.set_mode((100 , 100 ),0,32)
# 2.1 屏幕填充白色
screen.fill((0xff,0xff,0xff))
# 2.2 加载图片
image = pygame.image.load('0.png')
# 2.3 贴图图片
screen.blit(image,(10,10),(0,0,10,10))
# 3. 刷新显示窗口
pygame.display.flip()  
# 假装进行了什么操作 （延时2秒）
pygame.time.wait(2000)
# 4. 退出框架
pygame.quit()

```


显示效果

![2-3](snake-display-image/2-3.png)

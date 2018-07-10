---
title: '05 贪吃蛇 - 音频加载与播放：音频处理'
date: 2018-07-09 16:52:23
tags:
    - Python
    - pygame
categories:
    - '基础教程'
    - 'Python贪吃蛇教程'
---

看了下pygame的介绍，发现居然是！！SDL！！2014年一直玩的东西，打算水完Python系列以后，再水一下用SDL2来做一个贪吃蛇，很久没弄了。

<!-- more -->

要播放音频，和显示文字差不多，基本的步骤如下：

1. 初始化音频
2. 加载音频
3. 播放音频

具体代码如下：

## 播放音效代码

```python
import pygame 
from pygame import QUIT,KEYDOWN
from pygame.mixer import Sound
# 1. 先初始化游戏框架
pygame.init()
# 2. 创建一个窗口
screen = pygame.display.set_mode((300,150),0,32)
# 加载字体 (使用系统字体)
font = pygame.font.SysFont('arial', 16)
# 运行标志
running = True
text = 'Play music with any keydown'

clock = pygame.time.Clock()

'''
初始化音频播放器
'''
# 初始化
pygame.mixer.init()
# 加载音效
sound=Sound('eat.wav')

# 1. 游戏死循环
while running:
    # 2. 获取事件流
    for event in pygame.event.get():
        # 3. 处理事件流
        if event.type == QUIT:
            # 结束运行
            running = False
        elif event.type == KEYDOWN:
            # 播放音频
            sound.play()

    # 2.1 屏幕填充白色 (Red,Green,Blue) 颜色取值范围：0~0xff
    screen.fill((0xff,0xff,0xff))
    # 2.3 渲染文字
    text_surface = font.render(text  ,True,(0xff,0,0))
    # 2.4 贴图文字
    screen.blit(text_surface,(0,0))
    # 3. 刷新显示窗口
    pygame.display.flip() 
    clock.tick(60) 

# 4. 退出框架
pygame.quit()
```

### 方法说明

| 方法 | 说明 |
|------|-----|
|play(loops=0, maxtime=0, fade_ms=0) -> Channel | 开始音频播放 |
|stop() -> None| 结束音频播放 |
|fadeout(time) -> None | 播放淡出 |
|set_volume(value) -> None| 设置播放音量，0.0 ~ 1.0 |
|get_volume() -> value | 获取播放音量 |
|get_num_channels() -> count | 计算声音播放次数 |
|get_length() -> seconds | 计算音频长度 |
|get_raw() -> bytes | 获取声音二进制原始数据 |


---
title: '07 贪吃蛇 - 界面切换与控制'
date: 2018-07-11 18:29:23
tags:
    - Python
    - pygame
categories:
    - '基础教程'
    - 'Python贪吃蛇教程'
---

上一节我们创建了一个按钮，现在我们要把按钮利用起来，现在我们需要三个界面：

1. 游戏主界面：提供游戏的开始和退出
2. 游戏界面：显示贪吃蛇以及游戏
3. 游戏失败界面：贪吃蛇死亡后显示的界面
<!-- more -->

回想一下界面的整个流程，现在我们需要开始正式写了，我们需要一个主程序，来控制各种界面的切换，以及需要三个界面，控制界面的显示等事务：

```python
class View:
    screen = None

    """
    界面初始化
    """
    def __init__(self, screen):
        self.screen = screen

    """
    界面绘制处理
    """
    def on_draw(self):
        pass

    """
    界面事件分发
    """
    def on_event(self, event):
        pass

```

一个界面至少要包含两个方法，用来处理绘制和事件，现在我们来写第一个界面，这个界面为主界面，用来显示开始游戏和退出按键，以及显示一个游戏大图, 实现代码：

```python
class MainView(View):

    logo = None
    btn_start = None
    btn_exit = None
    context = None

    def __init__(self, context):
        self.btn_start = Button('Start Game', self._on_start)
        self.btn_exit = Button('Exit Game', self._on_exit)
        self.context = context
        self.logo = pygame.image.load('img/logo.png')
        w, h = self.logo.get_size()
        # 调整界面适应 Logo 大小以及显示按钮
        self.context.screen = pygame.display.set_mode(
            (int(w * 1.5), int(h * 2)), 0, 32)
        # 设置界面标题
        pygame.display.set_caption('Python Snake Game v1.0')
        View.__init__(self, context.screen)

    def on_draw(self):
        # 绘制界面
        self._on_draw()
        # 绘制按钮
        self.btn_start.on_draw(self.context.screen)
        self.btn_exit.on_draw(self.context.screen)

    def on_event(self, event):
        # 监控各种事件
        self.btn_start.on_event(event)
        self.btn_exit.on_event(event)

    def _on_exit(self):
        # 控制退出状态
        print('exit game')
        self.context.running = False

    def _on_start(self):
        # 切换界面到游戏界面
        print('start game')
        self.context.view = GameView(self.context)

    def _on_draw(self):
        '''
        计算各个按钮的位置以及Logo的位置
        '''
        s_w, s_h = self.context.screen.get_size()
        l_w, l_h = self.logo.get_size()
        x, y, bs_w, bs_h = self.btn_start.get_box()
        x, y, be_w, be_h = self.btn_exit.get_box()
        y_start = l_h + 50
        '''
        界面元素排版
        '''
        self.btn_start.pos = (s_w/2 - bs_w / 2, y_start)
        self.btn_exit.pos = (s_w/2 - be_w / 2, y_start + bs_h + 4)
        self.context.screen.blit(self.logo, (s_w/2-l_w/2, s_h/2-l_h/2))

```

为了使界面能够正常显示，我们添加一个 `Game` 类来控制游戏以及界面的显示，控制界面的显示与操作

```python 
import pygame
from pygame import QUIT
from pygameui import Button
from views import MainView


class Game:
    # 运行状态
    running = True
    GAME_START = 0
    GAME_OVER = 1
    # 屏幕状态
    screen = None
    # 显示界面
    view = None
    # 运行时钟
    clock = None

    def __init__(self):
        pygame.init()
        # 初始化基础界面
        self.clock = pygame.time.Clock()
        self.screen = pygame.display.set_mode((300, 150), 0, 32)
        self.view = MainView(self)

    def run(self):
        while self.running:
            for event in pygame.event.get():
                # 分发界面事件
                self.view.on_event(event)
                if event.type == QUIT:
                    self.running = False
            self.screen.fill((0xff, 0xff, 0xff))
            # 调用界面显示
            self.view.on_draw()
            pygame.display.flip()
            self.clock.tick(60)


if __name__ == '__main__':
    Game().run()
```


全部代码： https://github.com/DXkite/python-snake-game-demo/tree/master/07-snake-frame

*运行界面*

![](snake-frame/0.png)
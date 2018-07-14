---
title: '08 贪吃蛇 - 绘制与移动蛇'
date: 2018-07-12 21:23:49
tags:
    - Python
    - pygame
categories:
    - '基础教程'
    - 'Python贪吃蛇教程'
---

主界面做好以后，我们开始做贪吃蛇界面，首先定义好一个贪吃蛇游戏界面的类，创建好地图大小，我们这里创建一个 12*12的地图块


<!-- more -->

```python
class GameView(View):
    block = None
    food = None

    def __init__(self, context):
        View.__init__(self, context.screen)
        # 加载基础资源
        self.block = pygame.image.load('img/block.jpg')
        self.food = pygame.image.load('img/food.png')
        # 获取地图块大小
        self.BLOCK_SIZE = self.block.get_size()
        # 设置屏幕大小
        context.screen = pygame.display.set_mode(
            (12*self.BLOCK_SIZE[0], 12 * self.BLOCK_SIZE[1]), 0, 32)

    """""""""""""""""""""
    在某个位置绘制一个块
    """""""""""""""""""""

    def draw_surface(self, pos, surface):
        self.screen.blit(
            surface, (pos[0]*self.BLOCK_SIZE[0], pos[1]*self.BLOCK_SIZE[1]))

```

创建完成后，我们来实现一个贪吃蛇类，控制贪吃蛇的移动与绘制，首先我们生成一个数组存储贪吃蛇并对其进行一系列的处理

```python
class Snake:
    # 方向说明
    HEAD_UP = 0
    HEAD_DOWN = 1
    HEAD_LEFT = 2
    HEAD_RIGHT = 3
    # 移动方向
    direction = HEAD_LEFT
    # 身体数据
    body = []
    # 初始化
    def __init__(self, x, y):
        self.body = [[x, y], [x - 1, y], [x-2, y]]

    # 监听按键
    def on_event(self, event):
        pass

    # 绘制贪吃蛇
    def on_draw(self):
        pass

    # 判断是否吃到食物
    def has_eat(self, food):
        pass

    # 判断是否撞到物品
    def has_hit(self, blocks):
        pass

```

初始化的时候，我们在地图上创建了个 12*12 的地图，蛇从地图的任意地方生成蛇头，我们将蛇身初始为3格,初始向右爬行，则初始化身体：

```python
def __init__(self, x, y):
    self.body = [[x, y], [x - 1, y], [x-2, y]]
```

在初始化的时候我们需要将资源也加载上来

```python
def __init__(self, x, y):
    self.body = [[x, y], [x - 1, y], [x-2, y]]
    self.direction = self.HEAD_RIGHT
    self.img_head = [pygame.image.load(
        'img/head_' + str(i+1) + '.png') for i in range(4)]
    self.img_body = pygame.image.load('img/body.png')
```

现在我们监控按键，对于蛇，按下后会改变方向，且只能向正向改变方向（如：当蛇向上移动时不能一下向下移动）

```python
def on_event(self, event):
    if event.key == K_UP and self.direction != self.HEAD_DOWN:
        self.direction = self.HEAD_UP
    elif event.key == K_DOWN and self.direction != self.HEAD_UP:
        self.direction = self.HEAD_DOWN
    elif event.key == K_LEFT and self.direction != self.HEAD_RIGHT:
        self.direction = self.HEAD_LEFT
    elif event.key == K_RIGHT and self.direction != self.HEAD_LEFT:
        self.direction = self.HEAD_RIGHT
```

现在我们需要绘制蛇身，遍历身体的每一个位置，每次刷新绘图的时候都会调用 `on_draw` 我们通过监控 `on_draw` 调用的次数来计算速度，当蛇爬到边界时从对侧出现，实现代码：

```python
# 绘制贪吃蛇
def on_draw(self, view):
    # 绘制已有的身体
    for i in range(len(self.body)):
        if i == 0:
            view.draw_surface(self.body[i], self.img_head[self.direction])
        else:
            view.draw_surface(self.body[i], self.img_body)
    # 计算移动
    if self.frame_count < self.speed:
        self.frame_count = self.frame_count + 1
    else:
        self.frame_count = 0
        # 调整头部方向
        move = copy.deepcopy(self.body[0])
        # 计算移动
        if self.direction == self.HEAD_UP:
            move[1] = move[1] - 1
            if move[1] < 0:
                move[1] = view.h - 1
            self.body.insert(0, move)
        elif self.direction == self.HEAD_DOWN:
            move[1] = move[1] + 1
            if move[1] >= view.h:
                move[1] = 0
            self.body.insert(0, move)
        elif self.direction == self.HEAD_LEFT:
            move[0] = move[0] - 1
            if move[0] < 0:
                move[0] = view.w - 1
            self.body.insert(0, move)
        else:
            move[0] = move[0] + 1
            if move[0] >= view.w:
                move[0] = 0
            self.body.insert(0, move)
        # 剔除最末尾
        self.body.pop(len(self.body) - 1)
```

现在我们将蛇加入地图 GameView：

```python
class GameView(View):
    block = None
    food = None
    snake = None

    def __init__(self, context):
        View.__init__(self, context.screen)
        # 加载基础资源
        self.block = pygame.image.load('img/block.jpg')
        self.food = pygame.image.load('img/food.png')
        self.w = 12
        self.h = 12
        self.BLOCK_SIZE = self.block.get_size()
        context.screen = pygame.display.set_mode(
            (self.w*self.BLOCK_SIZE[0], self.h * self.BLOCK_SIZE[1]), 0, 32)
        # 初始化蛇
        self.snake = Snake(0, 0)

    """""""""""""""""""""
    在某个位置绘制一个块
    """""""""""""""""""""

    def draw_surface(self, pos, surface):
        self.screen.blit(
            surface, (pos[0]*self.BLOCK_SIZE[0], pos[1]*self.BLOCK_SIZE[1]))

    def on_draw(self):
        self.snake.on_draw(self)

    def on_event(self, event):
        if event.type == KEYDOWN:
            self.snake.on_event(event)
```

运行后可以发现蛇在移动，通过上下左右控制蛇身以及方向

![](snake-draw-and-move/0.png)

程序代码：https://github.com/DXkite/python-snake-game-demo/tree/master/08-snake-draw-and-move
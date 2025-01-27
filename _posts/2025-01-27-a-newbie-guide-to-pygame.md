---
layout: post
title:  "pygame新手指南"
date:   2025-01-27 10:00:00 +0800
categories: "software_development"
---

Pygame is a python wrapper for SDL, written by Pete Shinners. What this means is that, using pygame, you can write games or other multimedia applications in Python that will run unaltered on any of SDL's supported platforms (Windows, Linux, Mac, and others).

> Pygame 是 SDL 的 Python 包装器，由 Pete Shinners 开发。这意味着，使用 pygame，您可以在 Python 中编写游戏或其他多媒体应用程序，这些应用程序可以在 SDL 支持的任何平台（Windows、Linux、Mac 等）上原封不动地运行。

---

## Recognize which parts of pygame you really need.
> ## 识别到您真正需要的 pygame 的哪些部分。

### Know what a surface is.
> ### 知道什么是 surface。

The most important part of pygame is the surface. Just think of a surface as a blank piece of paper. You can do a lot of things with a surface -- you can draw lines on it, fill parts of it with color, copy images to and from it, and set or read individual pixel colors on it. A surface can be any size (within reason) and you can have as many of them as you like (again, within reason). One surface is special -- the one you create with pygame.display.set_mode(). This 'display surface' represents the screen; whatever you do to it will appear on the user's screen.

> pygame 最重要的部分是 surface。只需将表面视为一张白纸（补充：更像是绘画中“图层”的概念）。你可以用 surface 做很多事情 -- 你可以在其上绘制线条，用颜色填充其中一部分，将图像复制到该对象或从中复制图像，以及设置或读取其上的单个像素颜色。一个 surface 可以是任何大小（在合理范围内），并且你可以拥有任意数量（在合理范围内）的 surface。其中有一个 surface 很特殊 -- 您用 pygame.display.set_mode() 创建的 surface。 这个 “显示表面” 代表屏幕，你对它所做的任何事情都会出现在用户的屏幕上。

So how do you create surfaces? As mentioned above, you create the special 'display surface' with pygame.display.set_mode(). You can create a surface that contains an image by using pygame.image.load(), or you can make a surface that contains text with pygame.font.Font.render(). You can even create a surface that contains nothing at all with pygame.Surface().

> 那么，如何创建 surface 呢？ 如上所述，您用 pygame.display.set_mode() 创建了特殊的 'display surface'。您可以通过 pygame.image.load() 创建一个包含图像的 surface，或者你可以通过 pygame.font.Font.render() 创建一个包含文本的 surface。 您甚至可以通过 pygame.Surface() 创建一个 根本不包含任何内容的 surface。

Most of the surface functions are not critical. Just learn Surface.blit(), Surface.fill(), Surface.set_at() and Surface.get_at(), and you'll be fine.

> 大多数 surface 的功能并不重要。只需学习 Surface.blit()、Surface.fill()、Surface.set_at() 和 Surface.get_at() 等。

- Surface.blit():
  - 用于将一个Surface对象绘制到另一个Surface上。通常用于将图像、精灵等绘制到屏幕上。
  - 例如：screen.blit(image, (x, y))将image这个Surface绘制到screen这个Surface上的(x, y)位置。

- Surface.fill():
  - 用于用指定颜色填充整个Surface或指定的矩形区域。
  - 例如：screen.fill((255, 255, 255))将Surface填充为白色。

- Surface.set_at()/Surface.get_at()
  - 用于设置/获取 Surface 上指定位置的像素颜色
  - 例如：screen.set_at((x, y), color)将screen这个Surface上的(x, y)位置设置为color表示的颜色，color可以是一个 RGB 三元组 (R, G, B)，也可以是一个 RGBA 四元组 (R, G, B, A)，其中 A 表示透明度

## Use Surface.convert(). 
> ## 使用 Surface.convert()

Surface.convert() 的主要作用是将 Surface 对象转换为当前显示模式所使用的像素格式。当你加载一个图像或者创建一个新的 Surface 对象时，它的像素格式可能与当前显示模式不匹配，这会导致在绘制该 Surface 时需要进行额外的转换操作，从而降低性能。通过调用 convert() 方法，可以将 Surface 转换为与显示模式相同的像素格式，避免这些额外的转换操作，提高绘制效率。

示例：
- 使用 surface = pygame.image.load('foo.png').convert()
- 而不是 surface = pygame.image.load('foo.png')

注意事项：
- 透明处理：如果 Surface 包含透明度信息（例如 PNG 图像的透明通道），应该使用 convert_alpha() 方法而不是 convert()。因为 convert() 方法会丢失透明度信息，而 convert_alpha() 可以正确处理透明度。

像素格式：
- Pygame 会根据系统和硬件的不同，自动选择合适的默认像素格式。一般来说，现代系统支持 24 位 RGB 或 32 位 RGBA 格式
- 获取像素格式信息：
  - get_bitsize()：返回 Surface 的每个像素使用的位数（32或者24）。
  - get_masks()：返回一个元组，包含红、绿、蓝、透明度四个通道的掩码，用于确定每个通道在像素值中的位置（R/G/B/透明的排列顺序，是RGBA还是ARGB等等）。

## Rects are your friends. 
> ## Rect 是你的朋友。

Rect 只是一个矩形 —— 仅由其左上角的位置、宽度和高度定义。许多 pygame 函数将 rects 作为参数。定义 rect 后可以访问 Rect 的实用程序函数，这些函数包括移动、缩小和膨胀 rect、查找两个 rect 的并集以及各种碰撞检测函数。

```Python
rect = pygame.Rect(10, 20, 30, 30)
rect = pygame.Rect((10, 20, 30, 30))
rect = pygame.Rect((10, 20), (30, 30))

# 很多 Surface 对象的方法会返回一个 Rect 对象，代表该 Surface 的边界矩形。
image = pygame.Surface((100, 50))  # <Surface(100x50x32 SW)>
image_rect = image.get_rect()  # <rect(0, 0, 100, 50)>
```

```
import pygame

pygame.init()
rect1 = pygame.Rect(100, 100, 200, 150)
rect2 = pygame.Rect(150, 120, 100, 80)

# 判断两个 Rect 对象是否相交，常用于检测游戏中的碰撞
is_colliding = rect1.colliderect(rect2)  # 判断 rect1 和 rect2 是否相交（碰撞）
print(is_colliding)
```

## Don't bother with pixel-perfect collision detection.
> ## 不要为像素完美的碰撞检测而烦恼。

For most games, it's probably better just to do 'sub-rect collision' -- create a rect for each sprite that's a little smaller than the actual image, and use that for collisions instead. It will be much faster, and in most cases the player won't notice the imprecision.

> 对于大多数游戏，最好只执行“sub-rect collision” —— 为每个 sprite 创建一个比实际图像小一点的 rect，然后用它来进行碰撞。它会快得多，并且在大多数情况下，玩家不会注意到不精确性。

## Managing the event subsystem.
> ## 管理事件子系统。

Pygame's event system is kind of tricky. There are actually two different ways to find out what an input device (keyboard, mouse or joystick) is doing.

> Pygame 的事件系统有点棘手。实际上，有两种不同的方法可以找出输入设备（键盘、鼠标或操纵杆）正在做什么。

The first is by directly checking the state of the device. You do this by calling, say, pygame.mouse.get_pos() or pygame.key.get_pressed(). This will tell you the state of that device at the moment you call the function.

> 第一种是直接检查设备的状态。您可以通过以下方式执行此操作：
例如，调用 pygame.mouse.get_pos() 获取鼠标光标位置或 pygame.key.get_pressed() 获取所有键盘按钮的状态。

The second method uses the SDL event queue. This queue is a list of events -- events are added to the list as they're detected, and they're deleted from the queue as they're read off.

优点：
1. 获取的是实时状态 -- 如果 mouse.get_pressed([0]) 为 1，则表示此时此刻鼠标左键处于按下状态。
2. 它很容易检测到 “chording”，也就是说，同时存在多个状态。如果你想知道 t 和 f 键是否同时关闭，只需检查：
```Python
if key.get_pressed[K_t] and key.get_pressed[K_f]:
    print("Yup!")
```

缺点：
1. 它只报告设备在被调用时的状态：如果用户点击鼠标按钮，然后在调用 mouse.get_pressed() 之前松开它，则鼠标按钮将返回 0 -- get_pressed() 完全错过了鼠标按钮按下。

> 第二种方法使用 SDL 事件队列。此队列是一个事件列表 -- 检测到事件时将事件添加到列表中，并在读取事件时从队列中删除事件。

缺点：
- 无法实时获取状态：事件处理是基于事件的发生，只有当事件发生时才能获取到相关信息。对于一些需要实时了解设备当前状态的场景，如持续监测鼠标的位置或者按键的长按状态，仅依靠事件处理可能无法满足需求。

The lesson is: choose the system that meets your requirements. If you don't have much going on in your loop -- say you're just sitting in a while True loop, waiting for input, use get_pressed() or another state function; the latency will be lower. On the other hand, if every keypress is crucial, but latency isn't as important -- say your user is typing something in an editbox, use the event queue. Some key presses may be slightly late, but at least you'll get them all.

> 经验教训是：选择满足您要求的方法。如果你的循环中没有太多事情发生 -- 假设你只是坐了一会儿 True 循环，等待输入，使用 get_pressed() 或其他状态函数延迟会更低。另一方面，如果每次按键都很重要，但延迟并不那么重要 - 假设你的用户正在编辑框中键入内容，请使用事件队列。有些按键可能会稍微晚一些，但至少你会得到它们。

Another note about the event queue -- even if you don't want to use it, you must still clear it periodically because it's still going to be filling up with events in the background as the user presses keys and mouses over the window. On Windows, if your game goes too long without clearing the queue, the operating system will think it has frozen and show a "The application is not responding" message. Iterating over event.get() or simply calling event.clear() once per frame will avoid this.

> 关于事件队列的另一个注意事项 —— 即使您不想使用它，您仍然必须定期清除它，因为当用户在窗口上按下键和鼠标时，它仍然会在后台填充事件。在 Windows 上，如果游戏运行时间过长而未清除队列，操作系统将认为它已冻结并显示“应用程序未响应”消息。迭代 event.get() 或每帧简单地调用 event.clear() 可以避免这种情况。

## Software architecture, design patterns, and games.
> ## 软件架构、设计模式和游戏。

```Python
import pygame

pygame.init()

screen = pygame.display.set_mode((1280,720))

clock = pygame.time.Clock()

while True:
    # Process player inputs.
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            raise SystemExit

    # Do logical updates here.
    # ...

    screen.fill("purple")  # Fill the display with a solid color

    # Render the graphics here.
    # ...

    pygame.display.flip()  # Refresh on-screen display
    clock.tick(60)         # wait until next frame (at 60 FPS)
```

It does some initial setup, starts a loop, and then proceeds to repeatedly collect input, handle the game's logic, and draw the current frame forever until the program ends. The update, render, wait loop shown here is actually a design pattern that serves as the skeleton of most games -- it's prolific because it's clean, it's organized, and it works. (There's also an important but easy-to-miss design feature here in the form of a strict division between the game's logic and rendering routines. This decision alone prevents a whole category of potential bugs related to objects updating and rendering concurrently, which is nice).

> 它进行一些初始设置，启动一个循环，然后继续重复收集输入，处理游戏的逻辑，并绘制当前帧，直到程序结束。这里显示的 update，render，wait 循环实际上是一种设计模式，它是大多数游戏的框架 —— 它使用广泛，因为它清晰、有序且有效。（这里还有一个重要但容易被忽视的设计功能，即游戏的逻辑和渲染例程之间的严格划分。仅此决定就可以防止与对象同时更新和渲染相关的一整类潜在错误，这很好）。

It turns out that there are many design patterns like this that are used frequently in games and in software development at large. For a great resource on this specifically for games, I highly recommend [Game Programming Patterns](https://gameprogrammingpatterns.com/contents.html), a short free, e-book on the topic. It covers a bunch of useful patterns and concrete situations where you might want to employ them. It won't instantly make you a better coder, but learning some theory about software architecture can go a long way towards helping you escape plateaus and tackle larger projects more confidently.

> 事实证明，有许多这样的设计模式在游戏和整个软件开发中经常使用。对于专门针对游戏的大量资源，我强烈推荐 [Game Programming Patterns](https://gameprogrammingpatterns.com/contents.html)，这是一本关于该主题的简短免费电子书。它涵盖了许多你可能想要使用它们的有用模式和具体情况。它不会立即让您成为更好的编码员，但学习一些有关软件架构的理论可以大大帮助您摆脱停滞期并更自信地处理更大的项目。

---

## References

[A Newbie Guide to pygame](https://scuba.cs.uchicago.edu/pygame/tut/newbieguide.html)

[Game Programming Patterns](https://gameprogrammingpatterns.com/contents.html)

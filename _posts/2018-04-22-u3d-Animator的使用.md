---
layout: post
title:  "美妙的Unity-Unity2d入门"
date:   2018-04-18
excerpt: "记述了使用u2d最基础的一些知识"
tag:
- blog
- unity
comments: true
---
#### 打开你的2d模式

    你也许已经惊喜的发现，你的Unity打开之后默认是3d的视图。点击Scene窗口上方的"2D"按钮，即可切换到2d模式。
    Unity引擎的2d，实际上是伪2d。它的2d只是在合适的位置放上了一只平面摄像机，摄像机视角的方向平行于z轴。此时，Y轴决定了GameObject上下的
位置，X轴决定了GameObject左右的位置。你可能已经发现，你完全可以在没有2d模式的情况下，用3d视图制作一个2d或2.5d的游戏。但2d模式可以让你更
方便的制作出你的游戏。

#### 使用你的精灵

    2d模式下的GameObject只有一种，那就是精灵（Sprite）。精灵没有z轴厚度，而且它有着专门的编辑器Sprite Editor。这并不是说你不能在2d模
式下


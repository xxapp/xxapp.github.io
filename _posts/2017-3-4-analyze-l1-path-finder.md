---
layout: post
title:  "分析工具库 l1-path-finder"
date:   2017-03-04 10:53:55
categories: frontend library analysis
---

# 

## 简介

> A fast path planner for grids.

这是一个针对二维迷宫的快速路线规划算法，JS 的实现版本，下文简称 l1。在大概了解这个库之后，想到了三点用途：

1. 在作为嵌入式脚本时，指导机器进行简单的路线规划
2. 在基于 BS 的游戏中作为玩家的辅助，比如自动寻路
3. 用于地图应用，寻找最优路线

l1 支持在 nodejs 和浏览器环境运行。l1 的作者说这个库的算法性能不错，甚至可以和 C++ 的实现比肩，并给出了[性能指标对比]，有兴趣的小伙伴可以比较一下。

## 在线示例

项目主页有一个非常有趣的 [DEMO]

![DEMO](https://raw.githubusercontent.com/xxapp/xxapp.github.io/master/assests/l1-path-finder-demo.png)

若干蓝色小球有各自的起点和终点，它们被 l1 规划好了路线，不断地穿越迷宫徘徊于起点终点。这里有一个不易发现的功能：可以点住某一个点不放，小球们就会改变命运，向这个点全速前进。

## 源码分析

可以说 l1 的技术含量中，算法占主要部分。

核心算法是一种叫做`A*`的算法，从起点开始每次移动前都需要进入如下步骤。

对于当前位置的上下左右四个方向：

1. 计算已经移动的步数，记为 G
2. 计算从当前位置到目的地的距离（不考虑障碍物），记为 H
3. 判断上下左右四个方向的






[性能指标对比]:   http://mikolalysenko.github.io/l1-path-finder/benchmark.html
[DEMO]:          http://mikolalysenko.github.io/l1-path-finder/www/
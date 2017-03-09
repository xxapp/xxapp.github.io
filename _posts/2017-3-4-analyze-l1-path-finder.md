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

### A* 算法

核心算法是一种叫做`A*`的搜索算法，用来计算从起点到终点的最短路径。这里从非专业角度简述一下过程，从起点开始每次移动前都需要进入如下步骤。

计算当前位置的**下一个可移动的点**的的权值 `weight = distence + heuristic`，其中 distence 表示起点到这个点的距离，heuristic 表示这个点到终点不考虑障碍物的距离，叫做启发性估值。

``` js
// 源码片段
v.heuristic   = heuristic(this.landmarkDist, this.dstX, this.dstY, v)
v.weight      = Math.abs(this.srcX - v.x) + Math.abs(this.srcY - v.y) + v.heuristic
```

然后取 weight 最小的那个点进行移动。

重复上面的步骤，最终会到达终点并得出最短的那条路径。

### 缩小搜索范围

假设二维迷宫是按照网格划分的，A* 算法对于路径的探索是一个格子一个格子计算的，l1 做了一些工作把迷宫抽象成一个 `visibility graph`。

l1 依赖了另外一个 JS 库 `contour-2d` 获得迷宫墙壁的轮廓多边形，筛选出“凸点”，并两两相连成为一个 `visibility graph`，如下图所示：

![visibility graph](https://raw.githubusercontent.com/xxapp/xxapp.github.io/master/assests/l1-path-finder-graph.png)

其中黄色矩形表示墙壁，紫色线条表示 visibility graph，这样 A* 算法就可以基于这个 graph 来进行搜索了。

这个过程中用到了一个判断两点之间没有任何障碍物的算法。

``` js
proto.stabBox = function(ax, ay, bx, by) {
  var lox = Math.min(ax, bx)
  var loy = Math.min(ay, by)
  var hix = Math.max(ax, bx)
  var hiy = Math.max(ay, by)

  var s = this.integrate(lox-1,loy-1)
        - this.integrate(lox-1,hiy)
        - this.integrate(hix,loy-1)
        + this.integrate(hix,hiy)

  return s > 0
}
```

用到了积分的数学知识，非常优美非常有趣，这里就不深入分析了，有兴趣的小伙伴可以研究体会一下。

上图只给出了一个简单的例子，如果迷宫很复杂，有很多“凸点”，graph 的连线数量会成指数级增长，然而我们并不需要这么多连线，因此 l1 使用最小生成树（[Steiner tree]）算法进行优化，降低搜索过程中的开销。

## 总结

平时不太研究这种级别的算法，在分析 l1 的时候，很多算法看了代码也不太懂它为什么这么写，分析过程如果有错误请大神多多容忍和指导。自己也找到了当初开始学习编程时的感觉，那时候看 HTML 都是一堆乱码，不过这也激发了学习的热情。





[性能指标对比]:   http://mikolalysenko.github.io/l1-path-finder/benchmark.html
[DEMO]:         http://mikolalysenko.github.io/l1-path-finder/www/
[Steiner tree]: https://en.wikipedia.org/wiki/Steiner_tree_problem
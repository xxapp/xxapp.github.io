---
layout: post
title:  "分析工具库 premonish"
date:   2017-03-20 19:20:44
categories: frontend library analysis
---

![west-world](https://raw.githubusercontent.com/xxapp/xxapp.github.io/master/assests/west-world-dev.png)


我记得彩票站经常有些人盯着墙上的历史开奖记录，不时地用笔计算着什么，试图用这些神秘的算法预言下一次的开奖号码。相对于随机分布的事件来说，软件用户的动作更容易预测一些，[premonish](https://github.com/mathisonian/premonish) 就是这样一个运行于浏览器的 JS 库。

## DEMO

https://mathisonian.github.io/premonish/

需要用 PC 端的浏览器打开。点击 `SHOW DEBUG VIEW` 按钮可以显示鼠标未来的移动轨迹。

## 使用方法

``` js
import Premonish from 'premonish';
const premonish = new Premonish({
  selectors: ['a', '.list-of' '.selectors', '.to', '#watch'],
  elements: [] // 也可以观察一个 DOM 元素列表。
});

premonish.onIntent(({el, confidence}) => {
  console.log(el); // 我们猜测用户将与之交互的 DOM 节点。
  console.log(confidence); // 我们对用户意图的预测有多自信，用 0 到 1 之间的数字表示。
});
```

## 源码分析

这里有两个重要的步骤：

1. 预测鼠标即将到达的点
2. 找出 DOM 元素列表中离这个点最近的一个

### 预测鼠标运动

这里用到了 premonish 作者写的另外的一个库 curve-store。有点类似 redux，只是 curve-store 存储的是不同时间（t1~t2）的连续的状态值，然后可以获取 t1~t2 之间任一时间点的状态值。而 redux 存储的状态值是离散的且不借助开发工具是没有时间概念的。

``` js
// 创建 store
...
document.body.onmousemove = (e) => {
    // 向 store 输入值
    ...
    // 获取当前位置和瞬时速度
    const { velocity, position } = store.sample(time);
    // 计算鼠标即将移动到的点
    let x = position.x + 100 * Math.pow(1 + velocity.x, 2) * sign(velocity.x);
    let y = position.y + 100 * Math.pow(1 + velocity.y, 2) * sign(velocity.y);
    ...
}
```

### 寻找距离最近的元素

这里将需要观测的 DOM 元素看作是矩形，并通过 `getBoundingClientRect()` 方法计算矩形四个点的坐标。只要找出与预测的点最近的点，就能找到用户可能与哪个元素发生交互。普通的遍历方法可以实现，但这里做了优化，用泰森多边形来计算最近的点。使用的是一个外部库 d3-voronoi。

### 提高准度

如果连续被选为最有可能与用户发生交互的元素，预测的自信度 confidence 就会增加，并暴露给用户，用户可以根据需要来调节精确度。

## 总结

作者结合了物理学和地理学的知识解决用户界面上的问题，令人大开眼界，相信以后会有更多的对现实世界的研究应用到虚拟世界的应用。premonish 可以让使用者有非常大的想象空间，更好地优化人机交互过程。

## 疑问

再分析源码的过程中，有一段代码没有看懂，计算鼠标即将移动的距离：`100 * Math.pow(1 + velocity.x, 2) * Math.sign(velocity.x)`，有看得懂的小伙伴吗？
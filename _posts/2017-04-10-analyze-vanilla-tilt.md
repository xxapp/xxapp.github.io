---
layout: post
title:  "分析动画插件 vanilla-tilt.js"
date:   2017-04-10 15:42:49
categories: frontend plugin analysis
---

## 丝滑柔顺的 3D 倾斜效果

这是来自 Șandor Sergiu 的动画插件，项目地址是 https://github.com/micku7zu/vanilla-tilt.js

先来纵享一下丝滑的效果 [DEMO](https://micku7zu.github.io/vanilla-tilt.js/)

特点是丝滑，小巧，易用。

vanilla 指的是基于原生 JS 实现的意思（作为被 vanilla.js 深深欺骗过的小菜鸟对这个单词印象非常深刻），不过基于原生实现的插件可能是现在一些开发者的首选，比如说 jQuery 插件，本来项目中没有引用 jQuery，为了一个插件就引入 jQuery 不是很划算。

## 使用说明

给你想让其应用动画效果的元素加上 `data-tilt` 属性，并引入插件就好了。

也可以手动应用，在引入插件后，执行 `VanillaTilt.init(document.querySelector('.your-element'));`。

插件同时提供了一些参数，用来调整动画效果，比如反向倾斜、最大倾斜角度、透视程度和是否放大等。详细请参见项目主页。

## 源码分析

总体来说，源码做的事情是结合 CSS3 的 3D 变换和 JS 的计算能力实现的可交互的动画效果。

### 3D 效果是怎么实现的

利用 CSS3 的 transform 属性实现 3D 效果：

`transform: perspective(1000px) rotateX(0deg) rotateY(0deg) scale3d(1, 1, 1)`

通过 JS 动态调整 rotateX 和 rotateY 的值实现动画效果：

`this.element.style.transform = "..."`

### 效果为何如此丝滑柔顺

CSS3 本身就使带有一些优化，再加上作者是在 requestAnimateFrame 中动态调整 transform 属性的值，由浏览器内部合理安排动画执行顺序，进一步提高了动画性能。如果浏览器支持 GPU 加速，在鼠标进入元素的时候，将元素的 `will-change` 改为 transform 的方法可以提前启动 GPU 计算过程，最终造就了顺畅的动画效果。

### 元素的倾斜角度是根据什么算出来的

下图中 `event` 为 mousemove 事件触发时的事件对象；`rect` 为 `element.getBoundingClientRect()`。

![compute](https://raw.githubusercontent.com/xxapp/xxapp.github.io/master/assests/vanilla-tilt.png)

首先是通过鼠标位置和元素位置计算鼠标在元素内的坐标 `(x, y)`。然后计算当前鼠标位置距元素中心点的向量，`x - width / 2` 和 `height / 2 - y`。最后根据 `x向量 / width * 最大角度` 和 `y向量 / height * 最大角度`，计算出两个方向的倾斜角度，分别对应 rotateY 和 rotateX 的值。

## 结语

vanilla-tilt 是 JS 与 CSS3 结合使用的非常好的例子，值得学习一下。并且提供了 init、destroy 和 reset 等方法和 change 事件，很容易和支持组件的框架结合使用，这样的 API 设计也值得学习。
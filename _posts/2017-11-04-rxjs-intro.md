---
layout: post
title:  "RxJS 简介"
date:   2017-11-04 15:18:43
categories: language paradigm
---


## 响应式编程

RxJS 属于响应式编程，响应式编程在维基百科上的定义是“一种面向数据流与变化传播的异步编程范式”，想象在下雨的时候，雨滴落在湖面上的场景，雨滴带着数据，雨滴激起的涟漪就是数据的传播，无数雨滴落下让湖面呈现出优美而复杂的图案是软件的一种表现方式。在软件中，这些雨滴可能是用户事件，可能是来自于其它系统的消息，也有可能来自于定时器，如果是一场“大雨”，原来针对静态数据的编程方式可能不太适用了，我们需要一种能处理异步动态数据的编程方式，并且我们还可以用对异步静态数据一样的操作方式去操作动态数据，响应式编程正式我们需要的。

## RxJS

很多常用的语言不是为操作异步动态数据而设计的，为了能使用上这种特性，微软同学提出了一种响应式扩展的方式来实现，这种扩展方式很受欢迎，在其它一些编程语言中也有人实现了这种扩展，RxJS (Reactive Extension for Javascript) 就是这样诞生的。

### 表示方法

在 RxJS 中一个很重要的对象是 Observable，也就是我们前面提到的表示动态数据的数据结构，它可以将各种同步或异步的数据源转化为同一种结构。

``` js
// 静态数据
Rx.Observable.from([1, 2, 3]);
// 事件
Rx.Observable.fromEvent(document, 'keyup');
// Promise
Rx.Observable.fromPromise($.ajax(url));
// 还有 定时器、websocket 等等异步方法，都可以转化为 Observable 对象
// 不一一列举了
```

我们用这样的图来表示 Observable 对象：

```
--a---b-c--d-----e--|-->
```

可以理解为一个时间轴上分布着 abcde 五个数据，也可以理解为一个数据流。

### 使用方法

如何使用这样的数据流呢，Observable 对象提供了一个 subscribe 方法：

``` js
const numbers$ = Rx.Observable.fromEvent([1, 2, 3]);
numbers$.subscribe((num) => {
    console.log(num);
});
/*
> 1
> 2
> 3
*/
```

### 操作符

知道了数据流的表示方法和使用方法，类比普通的数据，我们还缺少操作方法，接下来要介绍的操作符就是做这件事的。

RxJS 内置了一些针对数据流的操作符，比如 map 和 merge：

```
------1--2---3---|-->
   map(x => x * 2)
------2--4---6---|-->

---1-----2-----3-----------|-->
------4------5-------6-----|-->
           merge()
---1--4--2---5-3-----6-----|-->
```

用代码表示就是：

``` js
// map
const newSource$ = source$.map(x => x * 2);
// merge
const mergedSource$ = source1$.merge(source2$);
```

### 转变思想

使用 RxJS 最难的地方在于如何选择操作符实现需求。想控制好这个强大的野兽需要了解它的习性，也就是需要用响应式编程的思想去思考问题。这里用一个例子来说明。

有一款闯关电子游戏《魂斗罗》，开发商为了提高可玩性，设置了一个彩蛋，如果玩家在进入游戏后按照顺序按下按键，上上下下左右左右BA，角色就会由原本的 3 条命变成了 30 条命。

按照一般的思路，我们会想到维护一个状态表示玩家已经按下的键，可能是一个数组，然后需要监听 keyup 事件，在事件回调中向这个数组插入 keyCode，同时保证数组最多只保留最新的 10 个值，并将其与彩蛋规定的 keyCode 进行对比，如果一致则成功开启彩蛋。

再来看用 RxJS 的方式怎么实现这个功能，首先玩家按键不断触发 keyup 事件，我们可以把这个事件转化成 Observable 对象。然后根据事件对象的 Observable 生成 keyCode 的 Observable。最关键的一步，就是如何每次按键都生成一个由最后 10 次按键的 keyCode 组成的数组，这里我们用一个操作符实现。最后对这个包含很多数组的 Observable 进行过滤，只留下与彩蛋规定的 keyCode 匹配的，对这个最终的生成的 Observable 进行 subscribe，在回调里面开启彩蛋，这样就实现了我们想要的功能。

抛开具体的代码实现，我们对上述两段描述进行对比，很明显第二段 RxJS 的描述字数比较多，但是仔细观察会发现，第二段转换成代码的过程要容易的多，因为它每一步的操作对象都是 Observable，而且只使用操作符来进行操作。

这是 RxJS 的代码实现：

``` js
const konami$ = Rx.Observable.from([38, 38, 40, 40, 37, 39, 37, 39, 66, 65])
const keys$ = Rx.Observable.fromEvent(document, 'keyup')
    .map(ev => ev.keyCode)
    .windowCount(10, 1)
    .flatMap(x => x.sequenceEqual(konami$))
    .filter(x => x);

keys$.subscribe(
    x => console.log('KONAMI!')
);
```

得益于 RxJS 这种单一的代码组织结构，不仅减少了流程描述与代码之间的差别，还可以让开发者进行更高程度的抽象，这也是 RxJS 的魅力所在。

## 适用的场景

软件开发中没有银弹，RxJS 也不是用来解决所有问题的。在最终结果依赖于多种数据源（如事件、定时器和异步方法等）的组合时，并且最终的计算结果的正确性和和时间相关，那么可以考虑使用 RxJS 了。
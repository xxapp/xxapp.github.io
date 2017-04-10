---
layout: post
title:  "分析框架 choo"
date:   2017-04-05 13:54:25
categories: frontend plugin analysis
---

PP: 已烂尾，不要学习，多批评。

## 简介

choo 是一个前端框架，项目地址是 https://github.com/yoshuawuyts/choo

> 有趣的函数式编程——用来创造健壮的前端应用且只有 4kb 的框架。
>
> <div align="right">-- <cite>Yoshua Wuyts</cite></div>

关于它的设计理念，项目 Readme 中描述得十分精辟，特此翻译出来：

> 我们认为编程应该是有趣且幸福的，不是苛刻的有压力的。小巧玲珑非常棒；使用高大上且缺乏解释的词汇并没有什么好处——如果这些词汇先把人吓跑的话。我们不想变得令人望而生畏，我们想变得出色、有趣然后*顺手*成为最好的选择，就是这样*随意*。
>
> 我们认为框架应该是一次性的，组件在框架之间是可再利用的。我们不喜欢 WEB 开发的现状，各类框架就像用高墙围住的花园，在彼此猜忌中竞争。我们希望你自由，不要被潮湿的地牢所困。通过将 DOM 作为基础，从一个框架切换到另一个可以无痛地进行。组件应该运行在任何有 DOM 的地方，不管使用的是什么框架。choo 在设计上很节制；我们认为它不会永远是一等的选择，所以我们使其从项目中被剔除非常容易，就像上手使用它那么容易。
>
> 我们不相信越大越好，大量的 API、大量的依赖和巨大的文件尺寸——我们将此视为用户复杂度的预兆。我们希望每一个在或大或小的团队中的人，可以充分地理解一个应用是怎么设计的。从一个应用开始构建时，我们就希望它是小巧的、高性能的并且容易推理的。这些都是为了更容易的代码调试、更好的结果和开心的笑脸。

这是一个简单的例子：

``` js
var html = require('choo/html')
var choo = require('choo')

var app = choo()
app.use(logger)
app.use(countStore)
app.route('/', mainView)
app.mount('body')

function mainView (state, emit) {
  return html`
    <body>
      <h1>count is ${state.count}</h1>
      <button onclick=${onclick}>Increment</button>
    </body>
  `

  function onclick () {
    emit('increment', 1)
  }
}

function logger (state, emitter) {
  emitter.on('*', function (messageName, data) {
    console.log('event', messageName, data)
  })
}

function countStore (state, emitter) {
  state.count = 0
  emitter.on('increment', function (count) {
    state.count += count
    emitter.emit('render')
  })
}
```

可以看出 choo 的 API 设计非常简洁。熟悉 express 的同学应该也看出它们之间的几分相似。

首先实例化了一个 app 对象，注册了 logger 插件和 countStore 插件，然后定义了一个路由并指定视图，最后将应用挂载到文档中。通过这些简单的 API 就可以实现一个前端路由+单向数据流+全局状态管理的单页面应用，配合 `app.toString` 还可以实现服务端渲染。

## 源码分析

### 全局状态管理和事件系统

一个 app 实例中有两个个支撑应用结构的对象：

``` js
var bus = nanobus()                 // 全局事件的实例，可以监听事件和发射事件
var state = {}                      // 全局状态对象，路由状态和文档等加载状态虽然没有存储在这里，但是为了方便说明，也将它们归为 state 的一部分
```

从上面的示例中可以看出，state 和 emitter(bus) 这两个对象是贯穿应用的始终的，state 存储应用大部分的状态信息，emitter 用来定义一些响应式操作，例子中就是利用 emitter 注册了一个 `increment` 事件，在处理函数中修改了 state，并触发框架内部注册的 `render` 事件。在用户点击按钮的时候会触发 `increment` 事件。

### 视图函数

视图部分使用了一个叫做 `bel` 的外部依赖，它可以将 ES6 新增的模板字符串转换为 DOM 节点，以备后续的视图更新使用。

### 视图更新

无论是路由改变引发的，还是全局状态改变引发的，都是需要触发 `render` 事件来更新整个视图，当然框架对此做了一些优化。

``` js
tree = router(createLocation())
rerender = nanoraf(function () {
    var newTree = router(createLocation())
    tree = nanomorph(tree, newTree)
})

bus.on('render', rerender)
```

`router(createLocation())` 根据最开始定义的路由及视图函数返回最新状态的 DOM 节点。nanomorph 的作用是根据原来的 DOM 节点和新的 DOM 节点做差异更新。（注意这里的差异算法是根据真实的 DOM 节点进行的）nanoraf 利用 `requestAnimationFrame` 进一步优化了视图更新的性能。
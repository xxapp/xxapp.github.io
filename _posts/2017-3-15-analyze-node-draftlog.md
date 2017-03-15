---
layout: post
title:  "分析工具库 draftLog"
date:   2017-03-15 09:46:57
categories: frontend library analysis
---

## 命令行的日志信息很单调？

可能我们需要 draftLog，因为：

> Because Logging can be pretty and fun
>
> -- <cite>ivanseidel</cite>

![](https://github.com/ivanseidel/node-draftlog/raw/master/midia/draftlog.gif)

draftLog 需要运行在 nodejs 环境下，可以帮你修改已经打印出来的内容。如果你用过一些命令行工具，就会发现其中有很多有意思的日志打印效果，比如进度条、复选/单选组件，加载状态和动画等等，这些用 draftLog 都可以做到。但是源码并不帮你实现这些，需要你尽情发挥你的想象力，制作自己的小部件。

## 用法

``` js
require('draftlog').into(console)

var update = console.draft('本年度最佳员工是……')

console.log('最佳员工评选')

setTimeout(function () { update('他就是……') }, 1000)
setTimeout(function () { update('张全蛋！') }, 2000)
setTimeout(function () { update('噢，不，是赵铁柱！') }, 3000)
```

## 源码分析

接下来从它的用法入手分析。

`require('draftlog').into(console)` 这一句根据上下文应该是把draft方法挂到console对象上。`console.draft` 打印一行初始内容，并返回一个函数 `update`，之后就通过调用这个函数来更新这一行的内容。然而源码向我们展示的内容往往要丰富得多。

### `update` 函数怎么更新已经打印的内容

将 `update` 的实现节选并稍作调整如下：

``` js
// 保存光标位置
this._stream.write('\u001b7')
// 光标向上移动 linesUp 行
this._stream.write(`\u001b[${linesUp}A`)
// 清空行
this._stream.write('\u001b[2K\u001b[1G')
// 写入日志
this.write.apply(this, arguments)
// 恢复保存的光标位置
this._stream.write('\u001b8')
```

其中 `this._stream` 为控制台的输出流， `this.write` 为原生打印日志的方法，`linesUp` 为光标距离到要修改的这一行的距离。有趣的是，这里通过向控制台输出一些编码来控制行为，这些编码叫做 `ANSI Escape sequences`。正如上面展示的那样，有些编码并不是用来显示字符，而是用来控制行为或者改变颜色等等。

### `linesUp` 是怎么计算的

如果在 `console.draft` 之后又有新的打印输出，`linesUp` 需要随之改变才能找到正确的要修改的位置。如果新的输出是 `console.draft` 造成的，我们还可以在其内部改变这个值，但是 `console.log` 正常情况下是无法监听到它的输出的。

draftLog 使用了一个 hack 技巧，它构造了一个 LineCountStream 来代理 console 的输出流，以此来计算行数，神不知，鬼不觉：

``` js
var lineCountStream = LineCountStream(console._stdout)
console._stdout = lineCountStream
```

## 总结

虽然图形界面早已替代命令行界面成为主流，但命令行因更接近底层，更受开发人员的青睐，draftLog 可以适当美化命令行输出，增强开发体验。
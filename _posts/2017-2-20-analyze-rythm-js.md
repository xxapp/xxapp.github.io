---
layout: post
title:  "分析插件Rythm.js"
date:   2017-02-20 18:39:18
categories: frontend plugin analysis
---

一个有趣的 JS 库“Rythm.js”（下文简称rythm），它的作者 Okazari 说这是：

> 一个会让你的页面跳舞的 javascript 库。

看到这我已经迫不及待地打开了在线 [DEMO] 玩耍一番，看看 Okazari 是如何让页面跳舞的。

可以看到，原本是静止的标题和按钮们都随着音乐的节奏“舞动起来”，非常的带感，也应了rythm（节奏感）这个名字。

虽然看起来没有广泛的实用价值，但可以看出 Okazari 是一位懂得生活乐趣的人。

安装和使用在 Github 上的 README 中写得非常详细了。提供了比较流行的 npm 安装方式，也能满足传统的 script 引入方式，UMD 的组织方式让使用者最后得到一个名为 Rythm 的对象。

为了让特定的元素按照不同音调的音乐节奏跳舞，通过class属性对其进行标注。

``` html
<!-- 默认提供了三种音调，rythm-bass（低音）、rythm-medium（中音）和 rythm-high（高音） -->
<!-- 你也可以定义自己的音调，后面会详细分析 -->
<div class="rythm-bass"></div>
```

``` js
// 启动机器，放上碟片，开始狂欢吧！
var rythm = new Rythm();
rythm.setMusic("../examples/sample.mp3");
rythm.start();
```

分析一下 rythm 的源代码。

首先载入声音源，rythm 提供了两种载入方式，一种是通过 AudioElement 获取声音源，另外一种是通过调用 `navigator.getUserMedia({ audio: true })` 获取。前者是从一个指定的声音地址载入，后者是从麦克风等设备的声音流载入。

接下来是分析音频数据，我们知道获取或者修改 HTML 结构是通过 DOM 操作来进行的，那如何获取和修改音频数据呢？HTML5 提供了 AudioContext 这个接口，就相当于 HTML 的 DOM。获取某一时刻音频数据的核心代码如下：

``` js
var analyser = new AudioContext().createAnalyser();
// 连接声音源
// ...
var frequences = new Uint8Array(1024);
analyser.getByteFrequencyData(frequences);

console.log(frequences); // [12, 255, 45, 33....] 索引代表频率，值代表经傅里叶变换后的值
```

再来看 rythm 定义好的三种音调，它们的定义如下：

``` js
rythm.addRythm('rythm-bass','size',0,10);
rythm.addRythm('rythm-medium','size',150,40);
rythm.addRythm('rythm-high','size',400,200);
```

其中 `size` 指的是让元素以变化大小来跳舞，算是一个“舞种”吧。目前只支持这个舞种，Okazari 说以后会添加其它的变化方式。后面的两个数字，指的是声音频率范围。使用者可以通过这个方法定义自己的音调。

最后是如何让元素跳得这么有节奏感的呢，最后的一个算法就是计算定义中的频率范围内在某一时刻的脉冲平均值，这里就要用到上面获取到的音频数据。脉冲计算公式为 `(当前值 - 历史最小值)/(历史最大值 - 历史最小值)`。

通过配置的参数和当前脉冲平均值计算元素的缩放值：`startingScale + (pulseRatio * currentPulse)`，应用到元素的 CSS scale 属性值上去。

首先 Okazari 的娱乐精神是值得我们学习的，有时候代码并不只是带来经济效益，还可以带来快乐。然后通过分析 rythm 增加了对 Audio API 的了解，增长了见识。

[DEMO]:         https://okazari.github.io/Rythm.js/
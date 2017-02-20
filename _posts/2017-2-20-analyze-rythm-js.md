---
layout: post
title:  "分析插件Rythm.js"
date:   2017-02-20 18:39:18
categories: frontend plugin analysis
---

给小伙伴们分析一个 JS 库“Rythm.js”（下文简称rythm），它的作者 Okazari 说这是：

> 一个会让你的页面跳舞的 javascript 库。

看到这我已经迫不及待地打开了在线 [DEMO] 玩耍一番，看看 Okazari 是如何让页面跳舞的。

可以看到，原本是静止的标题和按钮们都随着音乐的节奏“舞动起来”，非常的带感，也应了rythm（节奏感）这个名字。

虽然看起来没有广泛的实用价值，但可以看出 Okazari 是一位懂得生活乐趣的人，为了什么呢？Just For Fun！

安装和使用在 Github 上的 README 中写得非常详细了。提供了比较流行的 npm 安装方式，也能满足传统的 script 引入方式，UMD 的组织方式让使用者最后得到一个名为 Rythm 的对象。

that._analyser.getByteFrequencyData(that._frequences);
requestAnimationFrame(renderRythm);

[DEMO]:         https://okazari.github.io/Rythm.js/
---
layout: post
title:  "React Native 源码学习"
date:   2018-8-18 10:36:29
categories: reactnative sourcecode
---

git clone 源码仓库

切换到 0.43.4 分支

安装 node 依赖

npm run lint 未通过

npm run test 未通过，提示不支持 node@8.2.1

切换成 node@6.x.x 仍然未通过

----

## 从源码运行项目

https://facebook.github.io/react-native/docs/building-from-source.html

### 初始化工程

react-native init rn-source --verbose --version 0.43.4
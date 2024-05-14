---
title: 'React 18源码学习篇01 序言'
date: 2023-03-02
permalink: /posts/2023/03/react-source-01/
tags:
  - react
  - web
share: false
---

> 程序员对于“源码”似乎天然保持着“敬畏”的心理，尤其是对React这样知名的开源项目心里总不能避免产生这样的想法：这么多的包和源文件应该从哪里入手？怎么理清楚动辄十几层的函数调用和分支判断？会用React提供的接口并且掌握一些“最佳实践”不就足够了吗？源码的学习恰是一个“祛魅”的过程，在抽丝剥茧般细致的钻研和学习之后，有关React"最佳实践“的认知会更加深刻而Hooks的原理也不再神秘。

版本说明
======
React的版本发布信息维护在官方仓库的[release](https://github.com/facebook/react/releases)页面下，包括所有的React版本、对应版本的代码资源以及每次发布的更改(新功能、bug修复、不兼容变化和废弃的功能等)。当前最新的大版本为React 18，这个版本的新功能包括**自动合批**（**Automatic batching**）、**新的Root API**、**Concurrent Suspense**等。需要注意的是，在没有特别说明的情况下，本文以及后续源码学习系列的文章都以**v18.2.0**进行讲解。同时在讲解过程中就某些特定方面如**stack reconciler Vs Fiber reconciler**、**legacy mode Vs concurrent mode**等比较新老React版本的差异。

初步认识
======
<img src='/images/blog/react-source/01/react_overview.png'>

学习方法
======
由于React源码非常庞大复杂，因此非常不建议初学者在一开始就扎进具体的函数实现逻辑中去。初学者很容易迷失在琐碎的细节中，导致浪费了大量时间而收效甚微。比较推荐的方式是首先建立起对React源码的宏观认识，知道我们主要学习的是`react`、`react-dom`、`react-reconciler`、`scheduler`这四个包，然后再对React运行的运行过程有大概的了解，最后可以切入到具体的子模块进行学习。

学习React源码的方式就是亲自动手调试，查看函数调用栈和变量赋值。如果调用栈深、分支逻辑多，可以尝试用画图的方式进行辅助理解。另外官方文档、Github上React工作小组的讨论、每年的React Conf等第一手资料对于理解React的设计和演化都有莫大的帮助。

特点
======
React 18源码学习系列文章将以由浅入深、由宏观到微观的方式带领大家学习React源码。源码是**最直接**也是**最晦涩**的呈现方式，因此在本系列文章中会提供大量的图片辅助大家理解React源码。

期望
======
期望大家在完成本系列的学习之后可以达成以下几个目标:

- 理解React底层运行机制和原理

- 日常业务开发提效
  - bug定位&解决
  - 性能优化
  - 最佳实践

- 解释React常见问题
  - `vdom`、`jsx`、`fiber`之间的关系是什么
  - 为什么React由`stack reconciler`切换到`fiber reconciler`
  - 为什么`hooks`不能写在条件判断中
  - `setState`是同步的还是异步的
  - `diff`算法是如何实现的
  - `合成事件`是如何实现的
  - react常见的优化思路都有哪些
  - ...

如果你已经可以从源码的角度清楚地回答这些问题，那么本系列文章可能并不适合你继续阅读。如果你对这些问题仍然很迷惑，那么在完成本系列文章的学习后相信可以完美地回答出这些问题。

展望
======
react官方对react的定位是`library(库)`而不是`framework(框架)`，从个人理解的角度来说，react官方选择将生态交由社区进行开发，而自己只负责确定最核心的技术路线演进。因此一个技术栈以react为核心的软件工程师除了要熟悉react源码之外，更要对react和前端生态、对技术趋势有了解，包括不限于:

- 对react生态的熟悉程度
  - 路由: `react-router`
  - 状态管理: `react-redux`
  - 组件库: `ant-design`
- SSR框架
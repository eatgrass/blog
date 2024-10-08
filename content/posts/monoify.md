---
date: 2023-08-14
title: Monoify 一个文本绘图工具
tags: 
- Web
- Typescript
categories: Development
---

[Monoify](https://monoify.io) 是个文本绘图工具，这篇用来记录一下在做这个自娱自乐项目的过程中的一些想法。

## 动机

我在[上篇]({{<ref "/posts/how_terminal_works">}})博客里附上了一张终端风格的纯文本插图，那张图当时是逐个字符敲上去的，编辑修改过程会比较费时费力，如果图形再稍复杂一些，基本不可能再靠手敲了。所以产生了这个想法，我需要一个工具能让我方便的画纯文本线框图。网上搜一圈相关的工具，实在无法满意，另外想了一下复杂度应该在驾驭范围之内，于是决定自己造这个轮子。

## 技术选择

定位是一个轻量级的工具，利用Web做前端基本没有什么太多可说的，Vue3相对较熟，Typescript 是现学现卖，但我还是很庆幸选择使用 Typescript 的，这个待会再说。最不确定的一点就是我该用什么画图。虽然了解有 Canvas API[^1]，但是图形方面的编程从没做过，算是知识盲区。所以决定在两个方向上尝试调研最后做对比，一是学习了解原生的 Canvas API，一边去找现成的绘图工具库。

## Two.js

最后在众多工具库与原生API里选择了 Two.js[^2], 先说原生 API。

Path 2D[^3] 实际上可以满足开发需要，但是如果选择原生API，那意味着必须自己解决对画布上的图形和对象的映射和管理。缩放、拖动画布这类 Transform 操作可能会需要花更多的时间研究。如果已有的工具库也没有提供这类支持，那也只有硬着头皮写了，但事实证明这些是一类比较通用的关注点。

筛选工具库的过程跟自己实际需要相关，我的考量主要在以下几点
* 能简化开发过程
* 简单轻量, 提供 2D 图形绘制足矣，无需 3D, WebGL 渲染
* 成熟稳定

最后对比下来，Two.js 看上去是个非常不错的选择，简单的 API, 超 8k 的关注度，项目质量看上去也很不错， 只是看上去停更有一段时间了。但是我在做这个项目的时候贡献了一个 Bug 修复，现在我也进了贡献者列表了[^4]。

## Coding

写代码的过程还是比较愉快的，构思功能边界、设计模块、动手实现，两天一迭代，敏捷了两周，算一气呵成吧。还是那句话过程最有价值。

## 再看 Typescript

这次在使用 Typescript 的时候确实给了我一些很不一样的感觉，一度有一种在写 Java 的错觉，项目过程中也出现了一次大规模重构，在类型系统的帮助下，夸张点说，是闭着眼睛重构完的，但是如果是 Javascript ，我可能会抓狂的吧。以前只是单纯写写 Vue 页面组件的情况下, 感觉 Typescript 优势并不明显, 用不用差别并不大，但是在大量抽象和模块化复用的场景下，Typescript 确实是一个很好的选择。


[^1]: MDN [Canvas API](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API)
[^2]: [two.js](https://two.js.org/)
[^3]: MDN [Path 2D](https://developer.mozilla.org/en-US/docs/Web/API/Path2D)
[^4]: two.js [0.8.11 Release](https://github.com/jonobr1/two.js/releases/tag/v0.8.11)

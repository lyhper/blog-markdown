---
title: background-image和img对滚动性能的影响
date: 2018-09-25
categories: css
---

## 前言
在业务中遇到一个长图片列表的场景，图片数量为100张。由于图片大小不一，而又不能将所有图片大小设置为相同的（缩放导致图片失真），因此需要依据图片不同的长宽进行缩放，达到铺满内容区同时居中显示。这里很容易就想到使用background来实现，使用background-size来进行缩放，使用background-position来使其居中显示。但当我实现之后却发现在滚动图片列表的时候，出现了很严重的卡顿，并且图片数量仅仅为50张。当我将图片更改为使用img标签展示时，卡顿消失。

![使用background-image卡顿](https://raw.githubusercontent.com/lyhper/blog-markdown/master/img/background-image卡顿.gif)
使用background-image卡顿

![使用img标签流畅](https://raw.githubusercontent.com/lyhper/blog-markdown/master/img/img流畅.gif)
使用img标签流畅

## 排查

### 使用rendering
遇到流畅性的问题，立马想到使用chrome开发者工具的rendering来观察渲染情况（开发者工具 => 右上角三个小点 => more tools => rendering）。勾选paint flashing/layer borders/FPS meter。

![使用rendering观察background-image列表](https://raw.githubusercontent.com/lyhper/blog-markdown/master/img/background-image排查rendering.gif)
使用rendering观察background-image列表

这里可以看到帧率保持在50多fps，且不断波动，而高亮处也显示图片列表区域在滚动过程中在持续重绘。

当我切换为使用img标签时，发现帧率保持在59.9fps，基本不波动，且占用gpu内存小了一半。这里持续重绘的区域与使用background-image一致。

![使用rendering观察img列表](https://raw.githubusercontent.com/lyhper/blog-markdown/master/img/img排查rendering.gif)
使用rendering观察img标签列表

### 使用performance
到这里依然不知道是什么导致了卡顿，于是再使用performance来观察下耗时。发现使用了background-image的图片列表painting时间耗费了长达2000ms，而使用img标签的列表painting却仅仅花了100ms。于是断定是滚动过程的重绘导致了卡顿。

![使用performance观察background-image列表](https://raw.githubusercontent.com/lyhper/blog-markdown/master/img/background-image排查performance.jpg)
使用performance观察background-image列表

![使用performance观察img列表](https://raw.githubusercontent.com/lyhper/blog-markdown/master/img/img排查performance.jpg)
使用performance观察img列表

既然是重绘导致卡顿，立马想到把图片列表提升为合成层会不会解决这个问题。于是，我对图片列表设置了transform: translateZ(0)，测试验证了我的想法，卡顿情况好了很多。

### 结论
在滚动的过程中，浏览器需要持续地重绘图片列表，而使用了background-image的列表在重绘过程中，需要不断地重新应用图片样式，包括设置图片的background-image，而使用img标签的列表在滚动过程中，由于样式中不包含设置图片，因此省去了这一样式计算过程。由此看来，在使用长列表图片时，尽量不要使用背景图来实现。
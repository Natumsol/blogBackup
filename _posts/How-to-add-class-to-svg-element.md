---
layout: post
title: "给SVG DOM增加class属性的三种解决方案"
date: 2014-11-21 21:19:43
categories: 技术
tags:
	- JavaScript
---
# 问题说明
最近因为项目中要用到`highcharts`来绘图，所以理所当然的要学习svg的相关知识。在操作svg元素节点过程中，发现了利用jQuery的`addClass`不能给svg元素增加class,这是我很好奇，同样是DOM节点，为什么就不行呢？经过查资料找到了事情的真相[1] ([戳我](http://forum.jquery.com/topic/adding-svg-class-classname-support-to-jquery))。

原来，HTML 和SVG DOM扩展对`className`有着不同的定义。对于HTML DOM 元素，className设一个字符串，但是对于SVG，className则定义为在SVGStylable中的一种SVGAnimatedString类型。

# 解决方案
为了解决接口不统一的问题，在`jQuery`可以用`attr`这个函数（建议使用）来给SVG DOM增加`class`属性:
<iframe width="100%" height="300" src="http://jsfiddle.net/Yoghurts/xLywesx1/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>
<!--more-->
如果不通过`jQuery`，通过原生`javascript`可以采用两种方法来解决这个问题。

通过`setAttribute()`接口函数（建议使用）操作`class`
<iframe width="100%" height="300" src="http://jsfiddle.net/Yoghurts/nkxxpdmx/2/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

通过`classList`这个属性（不建议使用），它是一个[token list](https://developer.mozilla.org/zh-CN/docs/Web/API/Element.classList)，里面保存了当前DOM元素的calssName,不过这个属性存在兼容性问题，支持IE 10+，Chrome 31+，Firefox31+，Safari 31+,详情见[CanIUse](http://caniuse.com/#search=classList)
<iframe width="100%" height="300" src="http://jsfiddle.net/Yoghurts/koe0gfqd/1/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

title: 解决artDialog弹出层嵌入的iframe大小无法自适应的问题
date: 2015-10-12 08:13:18
categories: 技术
tags:
	- iframe
	- 自适应
---
# 问题描述
如果在弹出层页面中载入iframe，而iframe里面的内容是用Javascript动态生成的，那么弹出层计算iframe的高度是根据页面初始大小得到的（Javascript动态生成的内容没有计算在内），那么这就会造成大小无法自适应，页面显示时不全的问题。在项目中，我因为在iframe里用模版引擎动态生成了表格，而弹出层在初始化高度时并没有把动态生成的内容考虑进去，造成弹出层的大小和iframe的实际大小相差太大，页面显示不全。

# 解决思路
待iframe动态内容生成后，重新调整弹出层的大小。以artDialog为例，可以直接调用height()和width()函数重置弹出层的高度和宽度。需要注意的是，获取iframe的高度和宽度不能用`jQuery`直接获取`html`或`body`的`height`和`width`，因为这样这样获得的大小就是当前弹出层的大小，并不是iframe的实际大小。应该用`scrollHeight`和`scrollHeight`来替代，不清楚`scrollHeight`和`scrollWidth`可以参考[这篇文章](http://blog.csdn.net/woxueliuyun/article/details/8638427)。
<!-- more -->
# 代码实现
项目中用到的弹出层为[artDialog](http://aui.github.io/artDialog/doc/index.html)
，模板引擎为Trimpath，现在以这两者为例为例，演示如何解决iframe高度无法自适应的问题。

父页面js:
```javascript
$(".pop-2").click(function(){
	dialog({
		title:"修复后",
		url:"pop-2.html",
		data: function(){
			var pop = dialog.getCurrent();
			//重置高度和宽度，使其自适应
			pop.height($(pop.content().iframeNode.contentDocument)[0].documentElement.scrollHeight);
			pop.width($(pop.content().iframeNode.contentDocument)[0].documentElement.scrollWidth);
		}
	}).showModal();
});
```

子页面js:
```javascript
$(function(){
	$.getJSON("data.json", function(result){
		var html = TrimPath.processDOMTemplate("template", result);
		$("body").html(html);
		top.dialog.getCurrent().data();
	})
})
```
具体的Demo 请点击[这里](/project/popLayer-self-adaption/)。PS：代码里有详细的注释。



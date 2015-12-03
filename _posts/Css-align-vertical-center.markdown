---
layout: post
title: "CSS实现垂直居中"
date: 2014-09-09 17:20:00 +0800
comments: true
categories: 技术
tags:
	- css
---

用css实现水平居中很容易，因为css有专门干这个的属性`text-align:center`;虽然这个属性看似用来设置文本的居中，实际上，在应用样式`body{ text-align:center }`之后，整个`body`下的`行内元素`都会水平居中(块元素可以利用`{margin:0 auto}`实现水平居中)。但是要实现垂直居中就比较难办了。有两中思路来解决这个问题，一个是利用`JavaScript`动态获取父元素的高度，并设置相应的子元素的margin值来去的垂直居中的效果。另一种是利用纯`css`属性来实现垂直居中。用`Javascript`来实现垂直居中来得很自然，故重点介绍一下用`css`方法来实现垂直居中的解决方案。
<!-- more -->
## 利用table来实现

在表格的子元素`table-cell`里有一个`vertical-align`的属性，按照[`MDN`](https://developer.mozilla.org/en-US/docs/Web/CSS/vertical-align)上的解释，我们可以知道同时用来设置行内元素（`inline`）和表格元素（`table-cell`）的垂直对齐方式的。利用这一点，我们可以将需要垂直居中的元素显示的指定显示方式为`table-cell`	(`inline`仅支持行内元素，不支持块元素，故不选`inline`)。比如：
```html
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Test</title>
<style>
	#container{
		width:200px;
		height: 300px;
		display: table;
		border:1px solid red;
	}
	#box{
		height: 20px;
		width: 20px;		
		display: table-cell;
		vertical-align: middle;
	}
	#innerbox{
		width: 20px;
		height: 20px;
		background: blue;
	}
</style>
</head>
<body >
<div id = "container">
	<div id="box">
		<div id = "innerbox">			
		</div>
	</div>
</div>

</body>
</html>
```
效果如下：

<div style = "width:100% ;border: 1px solid #ccc;margin-top:5px"><div id = "container" style = "width:200px;height: 300px;display: table;border:1px solid red;margin:5px auto">
<div id="box" style = "height:50px;width:50px;display: table-cell;vertical-align: middle;">
<div  id = "innerbox" style = "width: 20px;height: 20px;background:blue;"></div></div></div></div>

## 使用`{top:50%;position:relative}`来实现垂直居中

```html
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Test</title>

<style>
	#container{
		width:200px;
		height: 300px;
		background: red;
	}

	#box{

		height: 10px;
		width: 10px;
		position:relative;
		top:50%;
		background: blue;


	}
</style>
</head>
<body >

<div id = "container">
	<div id="box">		
		</div>
	</div>
</div>

</body>
</html>
```
效果如下：
<div style = "width:100%;border: 1px solid #ccc;margin-top:5px"> <div id = "container" style = "width:200px; height: 300px; border:1px solid green;margin:5px auto"> <div id="box" style = "height: 30px; width: 30px; position:relative; top:50%; background: red; "> </div> </div> </div>

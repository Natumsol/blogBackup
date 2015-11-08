title: 表格布局的那些事（二）
date: 2015-04-05 23:02:47
categories: 技术
tags: 
	- 表格布局
	- css
---
上篇聊了表格宽度的渲染，今天来聊聊表格高度的渲染。
<!--more -->
## 表格的高度渲染
由上篇，我们知道表格的宽度渲染有两种算法可以选择，令人遗憾的是，表格高度的渲染只有一种选择，这种渲染方式类似于宽度的自动渲染方式，不会根据用户的设定进行精确地渲染，只会确定一个最小高度。[W3](http://www.w3.org/TR/CSS2/tables.html#height-layout)上详细的说明如下：

>The height of a 'table-row' element's box is calculated once the user agent has all the cells in the row available: it is the maximum of the row's computed 'height', the computed 'height' of each cell in the row, and the minimum height (MIN) required by the cells. A 'height' value of 'auto' for a 'table-row' means the row height used for layout is MIN. MIN depends on cell box heights and cell box alignment (much like the calculation of a line box height). CSS 2.1 does not define how the height of table cells and table rows is calculated when their height is specified using percentage values. CSS 2.1 does not define the meaning of 'height' on row groups.

>In CSS 2.1, the height of a cell box is the minimum height required by the content. The table cell's 'height' property can influence the height of the row (see above), but it does not increase the height of the cell box.

>CSS 2.1 does not specify how cells that span more than one row affect row height calculations except that the sum of the row heights involved must be great enough to encompass the cell spanning the rows.

Demo 6：

<iframe width="100%" height="300" src="//jsfiddle.net/Yoghurts/008wrdjr/19/embedded/result,html,css/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

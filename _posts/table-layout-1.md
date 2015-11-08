title: 表格布局的那些事（一）
date: 2015-04-04 23:18:58
categories: 技术
tags: 
	- 表格布局
	- css
---

最近在一个页面Demo中用到了表格布局，但是最后的显示的效果和预期差很远，通过查询相关文档，发现浏览器渲染表格的宽度和高度的方式真算是奇葩了，让人有种抓狂的感脚。不过只要摸清楚了它的来龙去脉，掌握它的Render规律，就能“运筹帷幄之中，决胜千里之外”了。
<!-- more-->
## 表格的宽度渲染
## 简介
对于表格宽度的渲染方式，[W3](http://www.w3.org/TR/CSS2/tables.html#width-layout)上有如下描述：
>CSS does not define an "optimal" layout for tables since, in many cases, what is optimal is a matter of taste. CSS does define constraints that user agents must respect when laying out a table. User agents may use any algorithm they wish to do so, and are free to prefer rendering speed over precision, except when the "fixed layout algorithm" is selected.

>Note that this section overrides the rules that apply to calculating widths as described in [section 10.3](http://www.w3.org/TR/CSS2/visudet.html#Computing_widths_and_margins). In particular, if the margins of a table are set to '0' and the width to 'auto', the table will not automatically size to fill its containing block. However, once the calculated value of 'width' for the table is found (using the algorithms given below or, when appropriate, some other UA dependent algorithm) then the other parts of [section 10.3](http://www.w3.org/TR/CSS2/visudet.html#Computing_widths_and_margins) do apply. Therefore a table can be centered using left and right 'auto' margins, for instance.

上面这段话的主要意思是，CSS标准并没有为表格选择一个最佳的渲染方式，"最佳"的渲染方式可以依据不同的浏览器的实现方式，利用` any automatic table layout algorithm`来渲染。但是当用户为表格指定了`table-layout: fixed;`时，就会强制浏览器使用` fixed table layout algorithm`来渲染。
## 利用`fixed table layout algorithm`渲染
>In the fixed table layout algorithm, the width of each column is determined as follows:
> 1. A column element with a value other than 'auto' for the 'width' property sets the width for that column.
>2. Otherwise, a cell in the first row with a value other than 'auto' for the 'width' property determines the width for that column. If the cell spans more than one column, the width is divided over the columns.
>3. Any remaining columns equally divide the remaining horizontal table space (minus borders or cell spacing).

>The width of the table is then the greater of the value of the 'width' property for the table element and the sum of the column widths (plus cell spacing or borders). If the table is wider than the columns, the extra space should be distributed over the columns.

>If a subsequent row has more columns than the greater of the number determined by the table-column elements and the number determined by the first row, then additional columns may not be rendered. CSS 2.1 does not define the width of the columns and the table if they are rendered. When using 'table-layout: fixed', authors should not omit columns from the first row.

>In this manner, the user agent can begin to lay out the table once the entire first row has been received. Cells in subsequent rows do not affect column widths. Any cell that has content that overflows uses the 'overflow' property to determine whether to clip the overflow content.


翻译过来就是：
>在fixed table layout 算法中，表格没列的宽度由如下三条规则决定：
1. 若列元素<的width属性为具体的数值，而不是'auto',则这一列的宽度由这个数值给出。
2. 另外，如果第一行的某个单元格的'width'属性为具体数值而不是'auto',那么这个数值将决定这一整列的宽度。
3. 其余没有指定具体数值的列将平分其余的宽度。

>表格的宽度是<span style = "color:red">为&lt;table&gt;元素指定的'width'值 </span> 和 <span style = "color:red">列表每列的宽度值相加（包括单元格间距和边框）</span> 中的较大者。如果表格宽度大于各列宽度之和，那么多出来的部分将按比例分给每一列。如果某一行拥有比第一行更多的列，那么新增的列可能不会被渲染（注意，是可能）。CSS 2.1没有定义被渲染之后的其的宽度。当第一行的宽度确定下来后，其后每一行的单元格的内容多少将不会影响每列的宽度。

可能这么生硬的翻译过来，理解起来有点费劲，接下来将会配合demo进行讲解。
Demo 1:

<iframe width="100%" height="300" src="//jsfiddle.net/Yoghurts/008wrdjr/13/embedded/result,html,css/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

从css代码`table-layout: fixed;`可以得出浏览器使用的是"fixed table layout"渲染算法，从html代码可以看出，table元素的width属性值为100 < 20 + 30 + 100 + 50 = 200,根据标准：

>表格的宽度是<span style = "color:red">为&lt;table&gt;元素指定的'width'值 </span> 和 <span style = "color:red">列表每列的宽度值相加（包括单元格间距和边框）</span> 中的较大者。

可以得出表格的宽度为200px;
	
Demo 2：
将Demo 1 &lt;table&gt;的width属性改为300，结果预览：

<iframe width="100%" height="300" src="//jsfiddle.net/Yoghurts/008wrdjr/11/embedded/result,html,css/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

此时，表格的总宽度为300，各列的宽度也不再是指定的值了。根据标准：
>如果表格宽度大于各列宽度之和，那么多出来的部分将按比例分给每一列。

现在，我们可以验证一下看是否如它所说的那样。按照规则，第一列的宽度应该为`20 + (300 - 200) * 20 / 200 = 30`；第二列为 `30 + (300 - 200) * 30 / 200 = 45`；第三列为 `30 + (300 - 200) * 100 / 200 = 150`；第四列为`30 + (300 - 200) * 50 / 200 = 75`。完全符合浏览器渲染出来的结果！

Demo 3：

<iframe width="100%" height="300" src="//jsfiddle.net/Yoghurts/008wrdjr/14/embedded/result,html,css/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

从Demo 3 中可以看出，当列表各列的宽度被确定之后，单元格里面内容的多少不会影响到单元格的宽度。

Demo 4：

<iframe width="100%" height="300" src="//jsfiddle.net/Yoghurts/008wrdjr/16/embedded/result,html,css/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

如果没有为table元素设置width属性（默认为`auto`），那么当表格的单元格内容超出设定的宽度时，单元格所在的那列宽度为自动增加，现在可以理解为表格的宽度为刚好装下表格内容的最小宽度。
## 利用自动表格布局渲染方式
[W3](http://www.w3.org/TR/CSS2/tables.html#width-layout)上对自动表格布局的描述如下：
>This algorithm may be inefficient since it requires the user agent to have access to all the content in the table before determining the final layout and may demand more than one pass.

>Column widths are determined as follows:
1. Calculate the minimum content width (MCW) of each cell: the formatted content may span any number of lines but may not overflow the cell box. If the specified 'width' (W) of the cell is greater than MCW, W is the minimum cell width. A value of 'auto' means that MCW is the minimum cell width.
Also, calculate the "maximum" cell width of each cell: formatting the content without breaking lines other than where explicit line breaks occur.
2. For each column, determine a maximum and minimum column width from the cells that span only that 	column. The minimum is that required by the cell with the largest minimum cell width (or the column 'width', whichever is larger). The maximum is that required by the cell with the largest maximum cell width (or the column 'width', whichever is larger).
3. For each cell that spans more than one column, increase the minimum widths of the columns it spans so that together, they are at least as wide as the cell. Do the same for the maximum widths. If possible, widen all spanned columns by approximately the same amount.
4. For each column group element with a 'width' other than 'auto', increase the minimum widths of the columns it spans, so that together they are at least as wide as the column group's 'width'.
This gives a maximum and minimum width for each column.

这段话的大概意思是：
>这种算法可能比较低效，因为UA(也就是浏览器)需要获取表格的所有内容才能最终决定表格的宽度。每列的宽度由如下四条规则确定：
1.计算每个单元格最小内容宽度（MCW）的方法：格式化的内容可能占用多行，但是不会溢出单元格。如果为单元格指定的宽度（W）比MCW大，那么W就为作为这个单元格的最小宽度。如果单元格的width属性被指定为`auto`的话，MCW就是这个单元格的最小宽度。
2.对于每列，决定其宽度的最小和最大宽度的单元格只能占用那列。每列的最小宽度是这列所有单元格最小宽度（或者是'width'属性，谁大选谁）中的最大者（译者注：比较拗口，哈哈）。每列的最大宽度是这列所有单元格中的最大宽度中的最大者。
3.&lt;!- -不甚理解，暂不译- -&gt;
4.对于width属性为具体数字而不是`auto`的列元素，会增加此列的最小宽度，这样他们就会至少和width属性指定的宽度一样宽了。

对于以上的quote，我自己的理解是浏览器并不会按照你给定的width精确地渲染列宽，你给出的width值只是一种基线，表示最终渲染出来的宽度绝对不会小于你给定的宽度，具体宽度会根据单元格的内容进行动态调整，简单的来说，如果指定的宽度能“装下”里面的内容，那么宽度就是指定的宽度，如果装不下就会适当增加宽度，直到能装下为止。。

Demo 5：

<iframe width="100%" height="300" src="//jsfiddle.net/Yoghurts/008wrdjr/17/embedded/result,html,css/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>


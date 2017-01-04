title: 利用jQuery Validation Engine 校验隐藏元素
date: 2015-08-13 16:46:39
categories: 技术
tags:
	- JavaScript
---
在前端的石器时代，表单验证一直是`jquery validation`的天下。如今的前端正处于战国时代，群雄并起，`jquery validation`这货实在不好用。特别是规定每个`input`标签要有一个出错信息容器，这常常迫使我牺牲页面的整洁和优雅，为每个`input`标签加个尾巴。

暑假期间,实验室要做新项目,既然是新项目，就必须要引入点新东西，是时候抛弃那些难看又不好用的插件了。调研发现`jQuery Validation Engine`这个表单校验插件很优雅，API设计的不仅简洁,而且错误信息也不需要为其指定一个容器，能够自适应各种输入框，简直是perfert。

在项目开发的过程中，我需要校验一个隐藏输入框，但是令人意外的是`jQuery Validation Engine`对隐藏输入框直接忽略(>_<)，经过一番折腾之后，找到了解决方案，分享如下：
<!-- more -->
待校验的表单为：

```html 待校验的表单
	<form action="" method="post" id="testForm">
		<table>
			<tr>
				<td>一般输入框:</td>
				<td><input type="text" name="test1"></td>
			</tr>
			<tr>
				<td>隐藏输入框:</td>
				<td><input type="hidden" name="test2"></td>
			</tr>
			<tr>
				<td><input type="submit" value="提交"></td>
			</tr>
		</table>
	</form>
	
```

初始化：

```js 初始化表单校验
		$("#testForm").validationEngine({
			validateNonVisibleFields: true, // 启用隐藏输入框校验
			promptPosition: 'centerRight',
			addPromptClass: 'formError-noArrow formError-text',
		})

```

这样还不够，因为`<input type="hidden" name="test2">`在界面上没有被定位，出错信息不知道在哪里显示，所以不能直接为`input `指定`hidden`属性，应该为其自定义一个css类，设置`visibility:hidden`，所以更改后的html代码为：

```html 待校验的表单(2)
<style>
	.input-hide{
		visibility:hidden;
	}
</style>
	<form action="" method="post" id="testForm">
		<table>
			<tr>
				<td>一般输入框:</td>
				<td><input type="text" name="test1"></td>
			</tr>
			<tr>
				<td>隐藏输入框:</td>
				<td><input type="text" class="input-hide" name="test2"></td>
			</tr>
			<tr>
				<td><input type="submit" value="提交"></td>
			</tr>
		</table>
	</form>
	
```

具体的栗子见 [ DEMO ](/project/validation-engine/index.html)
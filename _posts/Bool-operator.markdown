---
layout: post
title: "JavaScript中的布尔运算"
date: 2014-07-23 13:45:52 +0800
comments: true
categories: 技术
tags:
	- javascript
	- 布尔运算
---
JavaScript中的逻辑运算符和C++/Java语言中的逻辑操作符不一样。在C++/Java语言中，逻辑表达式的值一定是一个逻辑值`true`或`false`，但是在JavaScript中情况有所不同，逻辑表达式的值可以是`true`和`false`，也可以是对象`Object`。

一般说来，逻辑操作符在操作数都是布尔（`boolean`）类型的时候，返回布尔类型的值。但是当操作数里面有对象存在时，结果就不尽相同，具体说来有如下二种情况。

## 单目运算符非运算（`!`） ##
单目运算符非得返回值为布尔（`boolean`）类型，具体运算规则为：
<!-- more -->
- 如果操作数为一个对象，则返回`false`;
- 如果操作数为一个空字符串，则返回`true`;
- 如果操作数为非空字符串，返回`false`；
- 如果操作数为0，返回`true`；
- 如果操作数为任意非0数值（包括`Infinity`），返回`false`;
- 如果操作数为`null`，返回`true`；
- 如果操作数为`NaN`，返回`true`；
- 如果操作数为`undefined`，则返回`true`；


##  双目运算符（`||`和`&&`）##
双目运算符的结果根据两个操作数的类型不同返回值得类型也会不同。

### 逻辑或（`||`）运算 ###
- 如果第一个操作数为对象，则返回第一个操作数（对象）；
- 如果第一个操作数的求值结果为`false`，则返回第二个操作数；
- `null`，`NaN`，`undefined`的求值结果都为`false`。

For Example:

```javascript 准备工作
function student(name,age)
{
    this.name = name;
    this.age = age;
}//define constructor function

/*override prototype*/
student.prototype.getAge = function(){
    return this.age;
};
student.prototype.getName = function(){
    return this.name;
}

/*Generate two Object*/
var o1 = new student("张三",20);
var o2 = new student("李四",20);

```

```javascript Example 1
var result = o1 || o2;//o1
```

```javascript Example 2
var result = null || o2;//o2
```

```javascript Example 3
var result = o1 || "str";//o1
```

```javascript Example 4
var result = null || null;//null
```

```javascript Example 5
var result = undefined || undefined;// undefined
```

```javascript Example 6
ar result = NaN || NaN ;//NaN
```


### 逻辑与（`&&`）运算 ###

- 如果第一个操作数为对象，则返回第二个操作数（对象）；
- 如果第一个操作数的求值结果为`false`，则返回第一个操作数；
- `null`，`NaN`，`undefined`的求值结果都为`false`。

For Example：

```javascript Example 1
var result = o1 && o2;//o2
```

```javascript Example 2
var result = null && o2;//null
```

```javascript Example 3
var result = o1 && "str";//"str"
```

```javascript Example 4
var result = null && null;//null
```

```javascript Example 5
var result = undefined && undefined;// undefined
```

```javascript Example 6
var result = NaN && NaN ;//NaN
```

## 后记 #
这个小小的语法给我带来了很大的困扰。受到C++/Java先入为主的影响，想当然认为JavaScript的逻辑表达式结果一定是`boolean`类型，结果看到别人写的代码还以为是别人写错了....⊙﹏⊙b汗。

看书要认真，不要好高骛远，Don't hurry，just one step one footprint。	
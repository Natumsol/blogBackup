---
layout: post
title: "JavaScript基于原型链的继承原理"
date: 2014-07-08 22:35:02 +0800
comments: true
categories: 技术
tags:
	- javascript
	- 原型链
	- 继承
---
Javascript的继承是基于原型链的继承，这和其它OO语言例如`Java`和`C++`是不同的，特别是继承的细节，对初学者来说不容易把握。我（初学者）根据自己的理解和实践，整理归纳如下。

## 什么是原型链？
书本没有给出一个明确的定义。我的理解是：每个对象的构造函数都有一个原型对象，每个原型对象都有一个`constructor`和`__proto__`属性，其中`constructor`指向构造函数，`__proto__`属性指向它的父类原型。每当通过构造函数`new`一个对象时，这个新建对象都有一个`__proto__`属性，并指向它的原型对象。这样通过`__proto__`这个指针，一层一层的形成一条“链”，这条“链”就是原型链。整个过程如图所示：

![原型链](/images/blog/prototype_link.png)
<!-- more -->
## 继承的实现过程
知道了什么是原型链后，继承就好理解了。继承就是通过原型链来实现的。下面通过代码来具体说明。

```javascript A simple introduction to Javascript's inheritance
function superType()
{
	this.name = "superType";
}//超类构造函数

superType.prototype.getName = function(){
	return this.name;
}//超类原型

function subType(){
	this.name = "subType";
}//子类构造函数

subType.prototype = new superType(); //将子类的原型指向超类的实例，实现继承
//subType.prototype.constructor = subType; //最后将构造函数改为子类的构造函数

var supero = new superType();//实例化超类对象
var subo = new subType();//实例化子类对象

```

上面的代码中我先构造一个父类superType,然后构建子类构造函数，接下来`subType.prototype = new superType();`就是继承的关键了，将子类原型指向一个父类的实例，（注意这里，因为父类的实例不是父类的对象原型，只有`__proto__`属性，没有`constructor`属性。）这个父类的实例继续指向父类的原型，实现继承。整个过程如下图所示：

![继承图示](/images/blog/inherit_1.png)

通过实际调试的结果如下：

![调试1](/images/blog/inherit_2.png)

不过这样实现的继承的子类的原型的constructor属性指向的是父类的构造函数（不理解？仔细看上一段），所以需要手动改过来，增加一句`subType.prototype.constructor = subType;`。

调试结果如下：

![调试2](/images/blog/inherit_3.png)
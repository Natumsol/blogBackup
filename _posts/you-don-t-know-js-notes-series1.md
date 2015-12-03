title: "You don't know JS 笔记（一）"
date: 2015-12-03 10:51:44
categories: 技术
tags: JavaScript
---
## 前言
这里记录着自己阅读*You don't know JS*系列丛书的一些心得体会，也算是自己学习前端一年多，对`JavaScript`知识的一个梳理吧。
## Types（类型）
> Variables don’t have types, but the values in them do。

这句话的意思是说：变量是没有类型的，变量里面存的值才是有类型的。比如我声明一个变量`var a;`，此时`a`是不具有任何类型信息的。如果我给`a`赋值一个字符串`a = "Hello, 小草"`， 那么`typeof a`得到的信息`string`表示的是`a`里面的值`"Hello, 小草"`的类型信息。如果我现在重新给`a`赋值，`a = 0`，因为`a`里面存的值的类型已经发生变化，那么`typeof a`得到的信息也变成相应的`number`。这和静态语言比如`C/C++/Java`是不一样的，`C/C++/Java`在定义变量就指定了变量的类型，每个变量只能存储和自己类型匹配的值。

打个简单的比方，`JavaScript`的变量就像是一个瑞士军包，啥都能装，能装书、装水、装衣服，对了还能装逼，种类不限（通用）。但是`C/C++/Java`的变量就像是钱包，它设计之初就是为了装钱，所以它只能装钱（专用），如果要装其他的东西比如各种卡啊什么的，就要把这些卡也当作钱装进去，这就是所谓的**强制类型转换**。

>Many developers will assume “undefined” and “undeclared” are roughly the same thing, but in JavaScript, they’re quite different. undefined is a value that a declared variable can hold.“Undeclared”means a variable has never been declared.

这句话的意思是：`undefined` 和`undeclared`是两个完全不同的概念，很多人会混淆两者。在`JavaScript`中，`undefined`表示一个变量已经声明，但是还没有为其赋值，此时变量里面装的值就是`undefined`。比如：
```JavaScript
var a;
console.log(a);//undefined
```
但是`undeclared`表示一个变量从未被申明过，也就是在用这个变量之前，它从未在前面的代码中出现过（申明过），完全是突然出现的，此时对这个变量的读写操作都会导致解释器报错并停止运行。比如：
```JavaScript
console.log(b);//Uncaught ReferenceError: b is not defined(…)
```
有时候为了判断某个变量（比如`a`）是否为`undefined`，一般的做法是`if(a != undefined)`，但是这样存在一个问题，如果`a`没有申明过呢？这样就会造成解释器报错，为了防止这种情况发生，我们可以利用类型判断操作符`typeof`来进行安全的判断：`if(typeof a != undefined)`，`typeof`对与没有申明的变量也能安全的处理，并返回`undefined`。

## Values
> Using delete on an array value will remove that slot from the array, but even if you remove
the final element, it does not update the length property, so be careful!

这句话的意思是说：再利用`delete`操作符删除数组里面的某个元素时， 其实将那个元素里面的值给清空了，但是那个元素（slot）还在数组里，数组的

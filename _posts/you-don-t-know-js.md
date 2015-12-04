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

## Values（值）
> Using delete on an array value will remove that slot from the array, but even if you remove
the final element, it does not update the length property, so be careful!

这句话的意思是说：再利用`delete`操作符删除数组里面的某个元素时， 其实将那个元素里面的值给清空了（置为`undefined`），但是那个元素（slot）还在数组里，数组的长度并没有发生变化。
比如：
```JavaScript
var arr = [1,2,3,4,5];
delete arr[0];//arr => [undefined × 1, 2, 3, 4, 5]
for(var i = 1; i < arr.length; i ++) {
  delete arr[i];
}// arr => [undefined × 5]
```
补充一点，假设你想创建一个数组，要求里面的元素全是`undefined`，如果你这样做：
```JavaScript
var array = new Array(10);
```
其实是不对的。因为这样得到的数组实际上并没有10个插槽（slot），它是空的，只是它的`length`属性为`10`而已。如果在控制台里面查看，你会得到这样的结果：`[undefined × 10]`。想要真正得到10个元素全为`undefined`的数组，你应该这样做：
```JavaScript
var array = Array.apply(null, {length:10});
```
在控制台中查看`array`可以发现为`[undefined, undefined, undefined, undefined, undefined, undefined, undefined, undefined, undefined, undefined]`，和上面得到的结果明显不同。

> JavaScript strings are immutable, while arrays are quite mutable.
A further consequence of immutable strings is that none of the string methods that alter its contents can modify in-place, but rather must create and return new strings. By contrast, many of the array methods that change array contents actually do modify in place.

这句话的意思是：`JavaScript`的字符串是不可变的（类似于`C/C++`的字符串），而数组是可变的。字符串的不可变带来的影响就是其方法并不能直接修改字符串的内容，而是重新返回一个新的字符串。与此不同的是，数组的方法可以直接修改内容。

>Simple scalar primitives (strings, numbers, etc.) are assigned/passed by value-copy, but compound values (objects, etc.) are assigned/ passed by reference-copy. References are not like references/pointers in other languages—they’re never pointed at other variables/references, only at the underlying values.

基本类型（`string`,`number`,`boolean`）赋值/传值是进行的值传递，而混合类型（`object`,`array`等）是进行的引用赋值/传值，还有一点需要注意的是`JavaScript`的引用不像其他语言的引用，它是指向值本身。据个例子：
```JavaScript
var obj = {
  x: 1
};
function foo(obj) {
  obj = {x: 2};
}
foo();
console.log(x); // Object {x: 1}
```
`foo()`函数给`obj`重新赋值了，但是并没有改变外层定义的`obj`。

## Native（原生类型）
>Primitive values don’t have properties or methods, so to access .length or .toString() you need an object wrapper around the value. Thankfully, JS will automatically box (aka wrap) the primitive value to fulfill such accesses

这句话的意思是：基本数据类型并没有属性或方法，当你需要在基本数据类型上执行诸如`.length`或`.toString()`的操作的时候，你需要一个对象`Object`来包装这个基本数据类型。令人欣慰的是，在执行这样的操作的视乎，`JavaScript`会自动将其进行包装。

所以，对于一个字符串，我们是可以直接获取字符串的长度的，也可以执行许多字符串的方法，比如`.indexOf()`等等。

## Coercion（自动类型转换）
>if an object has its own toString() method on it, and you use that object in a string-like way, its toString() will automatically be called, and the string result of that call will be used instead.

如果一个对象自己有一个`toString`方法，那么当你把这个对象当作一个字符串用时（比如和另一个字符串相加），`toString`方法会自动调用，用返回的结果作为此对象。比如：
```JavaScript
var obj = {
  toString: function(){
    return "小草";
  }
}
var str = "hello, " + obj;
console.log(str);//"hello, 小草"
```
这里说一下操作符`+`和`-`，在`JavaScript`的世界中，`+`和`-`基本上可以作用与任何两种数据类型，但是，当作用与两种不同的数据类型，会以哪个数据类型为标准进行转换呢？这就涉及到一个优先级的问题，见下图：
<pre>
                    +---------------+     +-------------------------------------+
                    |               |     |                                     |
             +----->+  operator +   +----->   string > number > boolean         |
             |      |               |     |                                     |
+---------+  |      +---------------+     +-------------------------------------+
|         |  |
| casting +--+
|         |  |
+---------+  |
             |      +---------------+     +--------------------------------------+
             |      |               |     |                                      |
             +----->+  operator -   +----->    number > string > boolean         |
                    |               |     |                                      |
                    +---------------+     +--------------------------------------+

</pre>

举例说明：
```JavaScript
"1" + 1;//"11"
1 + true;// 2
"1" + true// "1true"

"1" - 1;//0
"1" - true //0
1 - "true" // 0
```

>bitwise operators in JS are defined only for 32-bit operations

位操作符的对象是32位的正数。如果作用对象超过了32位，那么将会进行`ToInt32`的操作。

title: "You don't know JS 笔记（一）"
date: 2015-12-03 10:51:44
categories: 技术
tags: JavaScript
---
## 前言
这里记录着自己阅读*You don't know JS*系列丛书的一些心得体会，也算是自己学习前端一年多，对`JavaScript`知识的一个梳理吧。
*高能预警：文章较长且琐碎，请自备板凳瓜子～*
## Types（类型）
> Variables don’t have types, but the values in them do。

这句话的意思是说：变量是没有类型的，变量里面存的值才是有类型的。比如我声明一个变量`var a;`，此时`a`是不具有任何类型信息的。如果我给`a`赋值一个字符串`a = "Hello, World"`， 那么`typeof a`得到的信息`string`表示的是`a`里面的值`"Hello, World"`的类型信息。如果我现在重新给`a`赋值，`a = 0`，因为`a`里面存的值的类型已经发生变化，那么`typeof a`得到的信息也变成相应的`number`。这和静态语言比如`C/C++/Java`是不一样的，`C/C++/Java`在定义变量就指定了变量的类型，每个变量只能存储和自己类型匹配的值。

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
<!-- more -->
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
自动类型转换比较复杂，为了说明他们各自的转换规则，我绘制了一张脑图来说明(图片较大，请保存下来用1:1的比例查看)：
![自动类型转换](/images/blog/20151204/coercion.png)

接下来说一下会引发饮食类型转换的两个操作，`+`和`-`运算。

首先说一下`+`运算。[ ECMA-262#11.6.1 ](http://www.ecma-international.org/ecma-262/5.1/#sec-11.6.1)对the Addition operator ( + )有如下描述：
>The addition operator either performs string concatenation or numeric addition.
The production AdditiveExpression : AdditiveExpression + MultiplicativeExpression is evaluated as follows:
1. Let lref be the result of evaluating AdditiveExpression.
2. Let lval be GetValue(lref).
3. Let rref be the result of evaluating MultiplicativeExpression.
4. Let rval be GetValue(rref).
5. Let lprim be ToPrimitive(lval).
6. Let rprim be ToPrimitive(rval).
7. If Type(lprim) is String or Type(rprim) is String, then Return the String that is the result of concatenating ToString(lprim) followed by ToString(rprim)
8. Return the result of applying the addition operation to ToNumber(lprim) and ToNumber(rprim). See the Note below 11.6.3.

>* **NOTE 1** No hint is provided in the calls to ToPrimitive in steps 5 and 6. All native ECMAScript objects except Date objects handle the absence of a hint as if the hint Number were given; Date objects handle the absence of a hint as if the hint String were given. Host objects may handle the absence of a hint in some other manner.
* **NOTE 2** Step 7 differs from step 3 of the comparison algorithm for the relational operators (11.8.5), by using the logical-or operation instead of the logical-and operation.

上面一段话比较晦涩难懂，大概意思是当执行相加操作时，如果左操作数或右操作数不是基本数据类型（`primitive`)时，会对其进行`ToPrimitive`转换（具体转换过程可见上面的脑图），这样，左操作数和右操作数都是基本数据类型了，但是如果左右两边的数据类型不一致该怎么办呢？这就涉及到一个优先级的问题，低优先级的数据类型会主动转换为和高优先级一样的数据类型，然后再进行相加操作。它们的优先级关系为：`string > num > boolean`，所以就会存在两种相加操作：字符串拼接和数字相加。具体的流程可以见下图：
![](/images/blog/20151204/1.png)
举例说明：
```JavaScript
"1" + 1;//"11"
1 + true;// 2
"1" + true// "1true"
```


对于`-`操作就单纯的多了，只是进行数值相减。[ECMA＃11.6.2](http://www.ecma-international.org/ecma-262/5.1/#sec-11.6.2)上的描述为：

>The production AdditiveExpression : AdditiveExpression - MultiplicativeExpression is evaluated as follows:
1. Let lref be the result of evaluating AdditiveExpression.
2. Let lval be GetValue(lref).
3. Let rref be the result of evaluating MultiplicativeExpression.
4. Let rval be GetValue(rref).
5. Let lnum be ToNumber(lval).
6. Let rnum be ToNumber(rval).
7. Return the result of applying the subtraction operation to lnum and rnum. See the note below 11.6.3.

大概意思为对于非数值类型的左右操作数进行`ToNumber`（详情见上面的脑图）,然后再进行数值相减运算。流程如下图所示：
![](/images/blog/20151204/2.png)

举例说明：
```JavaScript

"1" - 1;//0
"1" - true //0
1 - "true" // 0

```

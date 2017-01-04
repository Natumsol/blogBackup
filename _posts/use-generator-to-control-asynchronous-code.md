title: 利用Generator实现JavaScript异步流程控制
date: 2016-11-21 10:44:10
categories: 技术
tags:

    - JavaScript
    - ES6
    - Generator
---

# 前言

当初，`JavaScript` 引入异步（Asynchonrous）主要是为了解决浏览器端`同步IO`造成的UI假死现象，但是主流的编程语言和Web服务器都采取`同步IO`的模式，原因无非是：

1. 采用`同步IO`编写的代码符合人和直觉，代码容易编写和维护。
2. 对于`同步IO`造成的线程阻塞可以通过创建多线程（进程）的方式，通过增加服务器数量进行横向扩展来解决。

但是，在很多情况下，这种方式并不能很好地解决问题。比如对于静态资源服务器（CDN服务器）来说，每时每刻要处理大量的文件请求，如果对于每个请求都新开一个线程（进程），可想而知，性能开销是很大的，而且有种杀鸡用牛刀的感觉。所以`Nginx`采用了和`JavaScript`相同的策略来解决这个问题——单线程、非阻塞、异步IO。这样，当一个 IO 操作开始的时候，`Nginx` 不会等待操作完成就会去处理下一个请求，等到某个 IO 操作完成后，`Nginx` 再回过头去处理（回调）这次 IO 的后续工作。

然后2009年`NodeJS`的发布，又极大的推进了`异步IO`在服务端的应用，据闻，`NodeJS`在处理阿里巴巴“双十一”的海量请求高并发中发挥了很大的作用。

所以，`异步IO`真是个好东西，但是，编写异步代码却有一个无法回避的问题——回调函数嵌套太多、过多的回调层级造成阅读和维护上的困难——俗称“回调地狱（callback hell）”。

 ![屏幕快照 2016-11-21 下午2.37.50](/images/blog/20161121/1.png)

 为了解决这个难题，出现了各种各样的解决方案。最先出来的方案是利用任务队列控制异步流程，著名的代表有[`Async`](https://github.com/caolan/async)。然后[`Promises/A+`](https://promisesaplus.com/)规范出来了，人们根据这个规范实现了[`Q`](https://github.com/kriskowal/q)。那时候`Async`和`Q`各占边壁江山，两边都有不少忠实的拥趸， 虽然它们解决问题的思路不同，但是都很好的解决了地狱回调的问题。随着`ES6(ECMA Script 6)`将`Promise`标准纳入旗下，`Promise`成了真正意义上解决地狱回调的最佳解决方案（在支持`ES6`的环境中，开箱即用，不用引入第三方库）。

但是，虽然`Async` 和 `Promise`之流都在代码层面避免了地狱回调，但是代码组织结构上并没有完全摆脱异步的影子，和纯同步的写法相比，还是有很大的不同，写起来还是略麻烦。幸好，`ES6`引入了`Generator`的概念，利用`Generator`就可以很好的解决这个问题了。

![](/images/blog/20161121/2.png)

可以看到，经过Generator重写后，代码形式上和我们熟悉的同步代码没什么二样了。😁

下面我们就来介绍这种神奇的黑魔法！

<!-- more -->

# Generator 简介

[`Generator`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function)是`ES6`新引进的关键字，它用来定义一个`Generator`，用法和定义一个普通的函数（function）几乎一样，只是在`function`关键字和函数名之前加入了星号 ` *`。`Generator`最大的特点就是定义的函数可以被暂停执行，很类似我们打断点调试代码：点`Run`按钮代码就自动执行当前语句直到遇到下一个断点并暂停，不同的是`Generator`的这种暂停态和执行态是由代码来定义和控制的。

在`Generator`里，`yield`关键字用来定义代码暂停的地方，类似于给代码打断点（但不是真的打断点，不要和`debugger`关键字混淆），而`generator.next(value)`则用来控制代码的运行并处理输入输出。下面用代码来说明：

```js
/**
*@description 获取自然数
*/
function *getNaturalNumber(){
  var seed = 0;
  while(true) {
    yield seed ++;
  }
}

var gen = getNaturalNumber();// 实例化一个Generator

/* 启动Generator */
console.log(gen.next()) //{value: 0, done: false}
console.log(gen.next()) //{value: 1, done: false}
console.log(gen.next()) //{value: 2, done: false}
```

这是一个利用`Generator`实现的自然数生成器。通过实例化一个`Generator`，然后每次通过`gen.next()`取得一个自然数，此过程可以无限进行下去。不过值得注意的是，通过`gen.next()`取得的输出是一个对象，包含`value`和`done`两个属性，其中`value`是真正返回的值，而`done`则用来标识`Generator`是否已经执行完毕。因为自然数生成器是一个无限循环，所以不存在`done: true`的情况。

这个例子比较简单，下面来个稍微复杂点的例子（涉及到输入和输出）。



```js
/**
 * @description 处理输入和输出
 */

function * input(){
    let array = [], i = 4;
    while(i) {
        array.push(yield array);
        i --;
    }
}

var gen = input();
console.log(gen.next("西")) // { value: [], done: false }
console.log(gen.next("部")) // { value: [ '部' ], done: false }
console.log(gen.next("世")) // { value: [ '部', '世' ], done: false }
console.log(gen.next("界")) // { value: [ '部', '世', '界' ], done: false }
```

有人可能会对执行结果有疑问，不清楚外部的数据是如何传到`Generator`内部的，可能会猜想是通过`gen.next("西")`这句话传进去的，但是问题又来了，为什么`'部', '世', '界'` 都传进去了，`'西'`去哪了？别急，且听我慢慢道来。

首先我们要明白`yield`其实由两个动作组成，**输入** + **输出**（输入在输出前面），每次执行`next`，代码会暂停在`yield` **输出**执行后，其它的语句不再执行（**很重要**）。其次对于上面的例子来说，两次`next()`才真正执行完一次while循环。比如上面的例子里，为什么第一次输出的是`[]`,而不是`['西']`呢？那是因为第一次执行`gen.next("西")`的时候，首先会将`'西'`传进去，但是并没有接受的对象，虽然`西`确实是被传进来了，但是最后被丢弃了；然后代码执行完`yield array`输出之后就暂停。然后第二次执行`gen.next("部")`的时候，会先执行输入操作，执行`array.push('部')`, 然后进行第二次循环，执行输出操作。

现在总结一下：

1. 每个`yield`将代码分割成两个部分，需要执行两次`next`才能执行完。
2. `yield`其实由两个动作组成，**输入** + **输出**（输入在输出前面），每次执行`next`，代码会暂停在`yield` **输出**执行后，其它的语句不再执行（**很重要**）。

# 如何利用Generator进行异步流程控制？

利用`Generaotr`可以暂停代码执行的特性，我们通过将异步操作用`yield`关键字进行修饰，每当执行异步操作的时候，代码便在此暂停执行了。异步操作结束后，通过在回调函数里利用`next(data)`来控制`Generator`的执行流程，并顺便将异步操作的结果`data`回传给`Generator`，执行下一步。到此，整个异步流程得到了完美的控制，我们可以看一个小例子

<iframe width="100%" height="300" src="//jsfiddle.net/Yoghurts/xnqofbep/4/embedded/js,result/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

可以看到，Generator确实可以帮助我们来控制异步流程，但是上面的代比较很raw，存在以下两个问题：

- 不能自动运行，需要手动启动。
- 不能流程控制的代码需要自己写在异步回调函数里，且没有通用性。

所以我们需要构造一个运行器，自动处理上面提到的两个问题。

TJ大神的[co](https://github.com/tj/co)就是用来解决这个问题的。

下面我来详细说一下解决此问题的两种方法：利用`Thunk`和`Promise`

## 利用Thunk来构造generator自动运行器

这里引入了一个新的概念——thunk（ 读音 [θʌŋk] ），为了帮助理解，下面单独来介绍一下thunk。

### Thunk

维基百科上的介绍如下：

> In [computer programming](https://en.wikipedia.org/wiki/Computer_programming), a **thunk** is a [subroutine](https://en.wikipedia.org/wiki/Subroutine) that is created, often automatically, to assist a call to another subroutine. Thunks are primarily used to represent an additional calculation that a subroutine needs to execute, or to call a routine that does not support the usual calling mechanism. They have a variety of other applications to [compiler code generation](https://en.wikipedia.org/wiki/Code_generation_(compiler)) and in [modular programming](https://en.wikipedia.org/wiki/Modular_programming).

可以简单理解为，thunk就是为了满足函数（子程序）调用的**特殊需要**，对原函数（子程序）进行了特殊的改造，主要用在编译器的代码生成（传名调用）和模块化编程中。

在`JavaScript`中的`thunk`化指的是将多参数函数，将其替换成单参数的版本，且只接受回调函数作为参数，比如`NodeJs`的`fs.readFile`函数，`tnunk`化为：

```js
var fs = require("fs");
var readFile = function(filename){ // 包装为高阶函数
  return function(cb){
    fs.readFile(filename, cb);
  }
}
```

为了接下来的方便，我们在这里先构造一个`thunkify`函数，专门对函数进行`thunk`化:

```js
function thunkify(fn){
  return function(){
    var args = [].slice.call(arguments);
    var pass; 
    args.push(function(){
      if(pass) pass.apply(null, arguments);
	}); // 植入回调函数，里面包含控制逻辑
    fn.apply(null, args);
    return function(fn) {
      pass = fn; // 外部可注入的控制逻辑
    }
  }
}
```



###  运行器

现在开始构造我们的运行器。思路也很简单，运行器接受一个`Generator`函数，实例化一个`Generator`对象，然后启动任务，通过`next()`取得返回值，这个返回值其实是一个函数，提供了一个入口可以让我们可以方便的注入控住逻辑，包括：控制`Generator`向下执行、将异步执行的结果返回给`Generator`。

```js
function run(generator){
  var gen = generator();
  function next(data) {
    var ret = gen.next(data); // 将数据传回Generator
    if(ret.done) return;
    ret.value(function(err, data){
      if(err) throw(err);
      next(data);
	});
    next(); // 启动任务
  }
}
```



### 测试

下面我们来测试一下代码是否按照我们的预期运行。

<iframe width="100%" height="300" src="//jsfiddle.net/Yoghurts/727audwe/embedded/js,html/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

可以看到，完全符合我们的预期！

### 缺陷

虽然以上Thunk函数能完美实现我们对异步流程的控制，但是对于同步任务却不能正确的做出反应，比如我写一个同步版的`readFileSync`：

```js
function _readFileSync(filename, cb) {
  cb(null, file[filename]);
}
```

然后将`var readFile = thunkify(_readFile); // 将_readFile thunk化`改为`var readFile = thunkify(_readFileSync)`其余均保持不变，运行代码会发现执行不成功。什么原因造成的呢？其实只要仔细分析就会发现，主要问题主要出现在`thunkify`函数上面，在**流程控制函数**注入之前，任务函数就已经执行了，如果这个任务是异步的，那没问题，因为异步任务回调函数只会等主线程空闲了才会执行，所以异步任务能确保控制函数能够被成功注入。但是如果这个任务是同步的，那就不一样了。传给同步任务的回调函数会被立刻执行，之后给它注入控制逻辑已经没用了，因为同步任务早已执行完。

### 改进

为了改进`thunkify`函数，让它能适应同步的情况，可以考虑将**任务函数**的执行延后到控制逻辑注入后执行，这样就能确保无论任务函数异步也好，同步也罢，都能注入控制逻辑。

```js
/**
* 重写thunkify函数，使其能兼容同步任务
*/
function thunkify(fn) {
  return function(){
    var args = [].slice.call(arguments);
    var ctx = this;
    return function(done) {
      var called;
      args.push(function(){
        if(called) return ;
        called = true;
        done.apply(null, arguments);
      })
      try{
        fn.apply(ctx, args); // 将任务函数置后运行
      } catch(ex) {
        done(ex);
      }
    }
  }
}
```
改进后的：

<iframe width="100%" height="300" src="//jsfiddle.net/Yoghurts/65s0dpu2/embedded/js,html/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>


## 利用Promise来构造generator自动运行器

有了上面的基础，基于Promise就会容易理解多了。

### toPromise

首先，我们应该将异步任务改造成Promsie的形式，为了兼容同步任务，我们先对任务进行thunkify统一化，然后再转化为Promise。

```js

function toPromise(fn) {
    return function() {
        var thunkify_fn = thunkify(fn).apply(this, arguments); // 先thunkify化
        return new Promise(function(resolve, reject) { // 返回Promise
            thunkify_fn(function(err, data) {
                if (err)  reject(err);
                resolve(data);
            });
        });
    }
}
```

###  运行器

因为Promise是标准化的，所以构造Promise的运行器比较简单，我就直接show code了：

```js
function run(generator) {
  var gen = generator();
  function next(data) {
    var ret = gen.next(data);
    if(ret.done) return Promise.resolve("done");
   	return Promise.resolve(ret.value)
    	.then(data => next(data))
    	.catch(ex => gen.throw(ex));
  }
  
  try{
    return next();
  } catch(ex) {
    return Promise.reject(ex);
  }
  
}
```

### 测试
<iframe width="100%" height="300" src="//jsfiddle.net/Yoghurts/szkpdxf0/embedded/js,html/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

经测试，完全符合要求。



# 小结

异步流程的控制一直是`JavaScript`比较令人头疼的一点，`Generator`的出现无疑是一件囍事，相信随着ES6的普及以及ES7的推进（ES 7的`async`，`await`)，异步代码那反直觉的编写方式将一去不复返，编写和维护异步代码将会越来越容易，JavaScript也将会越来越成熟，受到越来越多人的喜爱。

# 参考文献

- [Generator与异步编程](http://www.infoq.com/cn/articles/generator-and-asynchronous-programming#)
- [Generators by Forbes Lindesay](https://www.promisejs.org/generators/)
- [ES6 Generator基礎](http://huli.logdown.com/posts/292331-javascript-es6-generator-foundation)
- [ECMAScript 6 入门](http://es6.ruanyifeng.com/#docs/async)
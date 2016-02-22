title: 如何构建一个微型的CMD模块化加载器
date: 2015-12-21 10:59:59
categories: 技术
tags:
    - 模块化加载器
    - CMD
---

# 前言
前端模块化是一个老生常谈的话题，模块化的好处是不言而喻，比如易于代码复用、易于维护、易于团队开发d等云云。对于前端模块加载器，以前仅仅止步于会用的阶段，为了加深对前端模块化的理解，大概花了一周的时间来学习、调研并尝试自己实现一个简易版的符合`CMD`规范的加载器。

# 设计
加载器是按照`CMD`规范进行设计的，具体的`CMD`规范就不列出了，详情请见[CMD规范](https://github.com/cmdjs/specification/blob/master/draft/module.md)。

## 入口函数 `use(ids, callback)`
![use](/images/blog/20151221/use.png)
<!-- more -->
## 模块定义函数 `define(factory)`
![define](/images/blog/20151221/define.png)
## 模块加载函数 `require(id)`
![require](/images/blog/20151221/require.png)
## 取得模块接口函数 `getModuleExports(module)`
![define](/images/blog/20151221/getModuleExports.png)
# 代码实现
## `use(ids, callback)`
`use`为程序启动的入口，主要干两件事：
1. 加载指定的模块
2. 待模块加载完成后，调用回调函数

```javascript
function use(ids, callback) {
     if (!Array.isArray(ids)) ids = [ids];
     Promise.all(ids.map(function (id) {
         return load(myLoader.config.root + id);
     })).then(function (list) {
         callback.apply(global, list);// 加载完成， 调用回调函数
     }, function (error) {
         throw error;
     });
 }
 ```
`use`会调用`load`函数，这个函数的作用是根据模块的`id`，加载模块，并返回一个`Promise`对象。
```javascript
 function load(id) {
        return new Promise(function (resolve, reject) {
            var module = myLoader.modules[id] || Module.create(id); // 取得模块或者新建模块 此时模块正在加载或者已经加载完成
            module.on("complete", function () {
                var exports = getModuleExports(module);
                resolve(exports);// 加载完成-> 通知调用者
            })
            module.on("error", reject);
        })
    }
```
## `define(factory)`
`define`的作用主要是用来定义一个模块。按照`CMD`的规范，定义一个模块的代码类似：
``` javascript
var factory = function(require, exports, module){
    // some code
}
define(factory);
```
为了方便说明，我给匿名函数取名为factory， factory就是我们模块定义的工厂函数，它只是define函数的一个参数，并不会被直接执行，而是会在需要的时候由专门的函数来调用生成接口。

所以， 一个模块文件被浏览器下载下来后，并不会直接运行我们的模块定义代码，而是会首先执行一个`define`函数，这个函数会取得模块定义的源代码（通过函数的`toString()`函数来取得源代码），然后利用正则匹配找到依赖的模块（匹配`require("dep.js")`这样的字符串)，然后加载依赖的模块，最后发射一个自定义事件`complete`，通知**当前模块**， 模块已经加载完成，此时，**当前模块**的就会调用与`complete`事件绑定的回调函数，完成与这个模块相关的任务，比如`resolve`与这个模块加载绑定的`Promise`。
具体实现为：
```javascript
function define(factory) {
  var id = getCurrentScript();
  id = id.replace(location.origin, "");
  var module = myLoader.modules[id];
  module.factory = factory;
  var dependences = getDependcencs(factory);
  if (dependences) {
      Promise.all(dependences.map(function (dep) {
          return load(myLoader.config.root + dep);
      })).then(function () {
          module.fire("complete"); // 依赖加载完成，通知模块。
      }, function () {
          module.fire("error");
      });
  } else {
      module.fire("complete");//没有依赖，通知模块加载完成
  }
}
```

## `require(id)`
`require`函数比较简单，主要作用就是根据模块`id`获取指定的模块，然后返回这个模块的对外接口。
```javascript
 function require(id) {
        var module = myLoader.modules[myLoader.config.root + id];
        if (!module) throw "can not load find module by id:" + id;
        else {
            return getModuleExports(module); // 返回模块的对外接口。
        }
    }
```
模块定义代码直到现在，才会被运行。运行模块定义代码的函数就是`getModuleExports`函数：
```javascript
    function getModuleExports(module) {
        if (!module.exports) {
            module.exports = {};
            module.factory(require, module.exports, module);
        }
        return module.exports;
    }
```
记得刚接触`sea.js`的时候，对接口暴露对象`module`和`exports`的区别不是很清楚，学习完别人的源码并尝试自己实现一遍的时候，它们的区别已经非常明朗了：
`exports`只是`module.exports`的一个引用，单纯的改变exports的值并不会对`module.exports`造成任何影响，所以通过
```javascript
exports = {
   foo: function(o){
      return o;
   }
}
```
这样的形式来定义接口是无效的。

# 测试
DEMO请见[这里](/project/microCMDLoader)

请打开控制台查看结果
# 总结
果然学习技术最好方法之一就是阅读别人的代码。阅读别人的代码是痛苦的，因为代码里充斥这他个人的代码癖好，有时候一个很简单的条件判语句可能用一些hack技巧实现了之后，在不了解的情况下，看的就比较痛苦了，以为另有玄机，傻乎乎的看了半天。不过，到最后搞明白之后，还是有些许成就感的。

前端模块化加载器，以前是只见树木不见森林，通过这次学习，不能说完全搞清楚了一个模块加载器的所有实现细节，但是对于像模块是怎样实现异步加载的，模块是如何定义的，模块间如何进行依赖分析的这些问题有了一个更深的认识和理解。

# 参考资料
- [JS模块加载器加载原理是怎么样的？](https://www.zhihu.com/question/21157540)
- [如何实现一个 CMD 模块加载器](http://annn.me/how-to-realize-cmd-loader/)
- [CMD Specification](https://github.com/cmdjs/specification/blob/master/draft/module.md)
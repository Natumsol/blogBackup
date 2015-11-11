title: 图解&lt;script&gt;的三种加载方式
date: 2015-11-11 15:52:06
categories: 技术
tags:
	- javascript

---

`javascript`脚本有三种加载方式，分别为`<script>`，`<script defer = "defer">`，`<script async = "async">`。这三种方式有诸多不同，网络上有许多文章花了很大功夫去解释它们的异同，但大部分如老太太的裹脚布，又臭又长，看得人云里雾里。今天在网路上看到相关文章，很短，只有一幅图和一点解释性的文字，但是让人一看就懂。真是一图胜千言啊。

摘录如下：

![三种脚本加载的异同点](/images/blog/20151111/script.jpg)

可以很清晰的看出：
* `<script>`: 脚本的获取和执行是同步的。此过程中页面被阻塞，停止解析。
* `<script defer = "defer">`：脚本的获取是异步的，执行是同步的。脚本加载不阻塞页面的解析，脚本在获取完后并不立即执行，而是等到`DOM`ready之后才开始执行。
* `<script async = "async">`: 脚本的获取是异步的，执行是同步的。但是和`<script defer = "defer">`的不同点在于脚本获取后会立刻执行，这就会造成脚本的执行顺序和页面上脚本的排放顺序不一致，可能造成脚本依赖的问题。

## 参考文献：
> [Asynchronous and deferred JavaScript execution explained](http://peter.sh/experiments/asynchronous-and-deferred-javascript-execution-explained/)
title: 利用redis实解决NodeJS多进程无法共享session的问题
date: 2016-03-14 11:30:17
categories: 技术
tags:
    - NodeJS
---
## 背景知识
我们知道后端是通过`session`来维持用户的会话的，每当用户发起一个请求的时候，用户的浏览器就会将用户的一个`sessionID`以`cookie`的形式发送到后端，后端接收到这个`sessionID`后，就会看内存中有没有`sessionID`为此`sessionID`的`session`，如果存在，则授权访问；否则重定向到授权页面或者返回错误码。
因为是`NodeJS`是单线程的，为了充分利用`CPU`的多核特性，采用了`cluster`模块，利用主从模式，生成与`CPU`核心数量相当的子进程，然后主进程用来捕获请求随机分配给子进程处理，并负责子进程的崩溃重启。但是，在引进了`cluster`模块之后，以前的`session`认证机制将完全不可用。原因如下：
>我们知道进程与进程之间是不能共享数据的，所以进程1的内存数据无法被进程2读取到，所以如果用户A认证的过程是被进程1所处理的，那么维持会话的`session`将保存在进程1的内存数据中；如果此用户接下的请求被进程2所处理，因为进程2没有处理过用户A的认证，没有维持这个会话的`session`，所以进程2会判断用户A并没有授权。这样用户A需要多次重复认证访问才能继续下去...

## 解决方案
我们现在的问题是`session`无法共享，所以我们可以想办法让`session`共享。通过将`session`存在基于内存的数据库`redis`里面，即可完美的解决了`session`共享的问题。
现在以`Express`框架为例，说一下具体做法：
1. 安装`redis`，安装过程参考 [ 官方文档 ](http://redis.io/download)
2. 安装`connect-redis`模块
```shell
npm install -g connect-redis
```
3. 配置`Express`
``` JavaScript
// 仅列出核心代码
var express = require("express");
var session = require("express-session");
var RedisStore = require('connect-redis')(session);
var config = require("./config");
var app = express();
app.use(session({
	saveUninitialized: true,
	resave: true,
	secret: config.sessionSecret,
	store: new RedisStore() // 利用redis存储session
}));
```
## 后记
其实，我们也可以换一种思路来解决这个问题。既然多进程之间无法共享`session`，那就索性不共享呗！放弃`cluster`，利用系统脚本开启与`CPU`数量相当的`NodeJS`进程，并且每个进程监听不同的网络端口，利用`Nginx`负载均衡机制，设置均衡模式为`ip_hash`，这样对于同一个客户端的请求转发到同一个端口，避免了将请求转发到没有认证的端口，这样也可以解决此问题。
基本思想如下图所示：
<pre>
                                                  +-------------------+
                                                  |                   |
                                            +----->  127.0.0.1:3000   |
                                            |     |                   |
                                            |     +-------------------+
                        +---------------+   |
+----------------+      |               +---+     +-------------------+
|                |      |               |         |                   |
|     user A     +------>               +--------->  127.0.0.1:3001   |
|                |      |               |         |                   |
+----------------+      |               |         +-------------------+
                        |    NGINX      |
+----------------+      |               |         +-------------------+
|                |      |               |         |                   |
|     user B     +------>               +--------->  127.0.0.1:3002   |
|                |      |               |         |                   |
+----------------+      |               +----+    +-------------------+
                        +---------------+    |
                                             |    +-------------------+
                                             |    |                   |
                                             +---->  127.0.0.1:3003   |
                                                  |                   |
                                                  +-------------------+

</pre>

---
layout: post
title: "Node.js开发环境的搭建"
date: 2014-09-12 16:52:17 +0800
comments: true
categories: 技术
tags:
	- NodeJS
---

如果你还在命令行里敲`node **.js`,`node --debug-brk=5858 **.js`,`supervisor **.js`来运行和调试NodeJs程序，那么你Out啦~~。用WebStorm + Nodejs 可以完美搭建NodeJs的集成开发环境，比直接在命令行里敲命令或者及利用node-inspector或eclipse等工具方便强大的多。个人认为具体优点如下：
<!-- more -->
- 可以添加断点，动态监视变量，解决NodeJs难调试的问题。
- WebStrom 会自动下载NodeJs源码，在调试时能跟踪代码底层运行情况。
- 方便快捷，提升工作效率。
- WebStorm 本身就是一款优秀的Javascript IDE，支持代码自动补全，校验等功能，而且还是跨平台的，全面支持Windows ,Mac, Linux 三大主流平台。

好了，说了这么多废话，现在就开始着手搭建开发环境了。

# 安装WebStrom 8 和 NodeJS

WebStorm 最新版：[下载链接](http://www.jetbrains.com/webstorm/) &nbsp;NodeJs 最新版：[下载链接](http://www.nodejs.org/download/)


# 配置NodeJs运行环境
打开Webstorm设置界面，选中`Node.js and NPM`选项，如果你安装了
`NodeJs`，软件会自动找到NodeJS的安装路径，然后按下图进行配置。
![运行环境配置](/images/blog/node_setup.jpg)


# 运行一个NodeJS程序
- 新建一个工程文件夹`test`,然后新建一个文件`index.js`,编写代码如下：
```javascript
var http = require("http");//引入依赖

http.createServer(function(request,response){
    response.writeHead(200,{"Content-type" : "text/html"});
    response.write("<p style = ''>Hello,World");
    response.end();
}).listen(8888);//创建http服务，监听端口为8888

```

# 执行	

![执行成功](/images/blog/node_success.jpg)
![执行成功](/images/blog/node_result.jpg)

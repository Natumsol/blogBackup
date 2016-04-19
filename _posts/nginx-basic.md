title: 利用 Nginx 进行反向代理和负载均衡
date: 2016-03-16 10:28:21
categories: 技术
tags:
      - Nginx
---
# Nginx简介
Nginx（发音同engine x）是一个网页服务器，它能反向代理HTTP, HTTPS, SMTP, POP3, IMAP的协议链接，以及一个负载均衡器和一个HTTP缓存。与旧版本（<=2.2）的Apache不同，nginx不采用每客户机一线程的设计模型，而是充分使用异步逻辑（这一点与`NodeJS`采取了相同的做法，支持高并发，Nginx在官方测试的结果中，能够支持五万个平行连接，而在实际的运作中，是可以支持二万至四万个平行链接），削减了上下文调度开销，所以并发服务能力更强。整体采用模块化设计，有丰富的模块库和第三方模块库，配置灵活。

# Nginx的配置文件
使用`Nginx`的方法就是写配置文件，配置文件能完全控制`Nginx`，使`Nginx`按照我们的需求进行运行，所以配置文件的每一项的是干啥的对我们来说就很重要了，具体配置文件各项的含义请参考 [nginx.conf配置文件详解](https://www.zybuluo.com/phper/note/89391)，我这里就不赘述了。

# 反向代理
## 准备工作
什么是`反向代理`？按照我的理解，`反向代理`是相对于`正向代理`来说的，因为他们都是代理，所以我先来解释一下什么是代理。所谓代理，就是在客户端和服务端之间强行添加了一层，用来实现流量转发的功能，粗略的框图如下：
<!-- more -->
<pre>
+----------------+         +---------------+        +--------------+
|                |         |               |        |              |
|                |         |               |        |              |
|                |  http   |               | http   |              |
|    client      <--------->    proxy      <-------->   server     |
|                |  https  |               | https  |              |
|                |         |               |        |              |
|                |         |               |        |              |
+----------------+         +---------------+        +--------------+
</pre>

所谓`正向代理`，是用于代理客户端的。举个很简单的例子：你直接在大陆地区访问`google.com`肯定是访问不了的，原因大家都知道，现在假如你有一台在美国的主机A，并且能够正常访问，那么你可以将浏览器对`google.com`的请求先转发给服务器A，服务器A收到请求后，扮演客户端的角色，发起对`google.com`的请求，服务器A收到响应后，又扮演服务端，将此响应原封不动的返回给你，自此，一次正向代理顺利完成。

`反向代理`顾名思义是用来代理服务端的。我们也举个简单的例子来说明：我们知道`google.com`没秒钟要处理如洪水般的网络请求，这些请求如果仅仅让一台单一的服务器处理，肯定是处理不过来的，我们自然而然的想到让多台服务器来处理这些请求，减少每台服务器的压力。但是现在有一个问题，多个服务器那就产生了多个IP，一般的，`google.com`只能解析到某个固定的IP（为了方便描述，我们暂且这样认为。实际情况下，通过设置也是可以让同一个域名解析到多个IP的），因为现在存在多个Server，我的一个`google.com`就不能解析到这些服务器上，而且用多个二级域名比如`server1.google.com`，`server2.google.com`等等也给用户造成了使用上的不便（一万台服务器，你咋不上天呢?），那该怎么办呢？通过反向代理可以很好的解决这个问题。为此，我绘制了下面的示意图：
<pre>
                                                                   +-------------------+
                                                                   |                   |
                                                            +------>   server 1        |
+-------------+                                             |      |                   |
|             +----------+                                  |      |                   |
|  client 1   |          |                                  |      +-------------------+
|             |          |                                  |
+-------------+          |                                  |
                         |         +------------------+     |      +-------------------+
                         |         |                  |     |      |                   |
+-------------+          |         |                  |     |      |   server 2        |
|             |          |         |                  +------------>                   |
|  clent 2    +-------------------->  reverse proxy   |     |      |                   |
|             |          |         |                  |     |      +-------------------+
+-------------+          |         |                  |     |
                         |         |                  |     |               .
      .                  |         |                  |     |               .
      .                  |         +------------------+     |               .
      .                  |                                  |
                         |                                  |      +--------------------+
+-------------+          |                                  |      |                    |
|             |          |                                  |      |   server m         |
|  client n   +----------+                                  +------>                    |
|             |                                                    |                    |
+-------------+                                                    +--------------------+

</pre>

而本文的猪脚——Nginx就是干这个事的。
## 配置反向代理
1. 我们利用 [http-server](https://github.com/indexzero/http-server) 启动一个本地server。
	``` shell
	http-server -p 3000
	```
	然后在所在的文件夹里新建一个`index.html`，输入`hello, nginx`。
2. 配置host文件
	以`Linux`为例：
	```shell
	sudo vim /etc/hosts
	```
	然后在里面添加一条记录：
	```shell
	test.com  127.0.0.1
	```
3. 配置nginx.conf
	在`http`里面添加：
	``` nginx
	    #本地http-server开启的server，命名为node-server，监听3000端口
	    upstream node-server{
	        server 127.0.0.1:3000;
	    }

	    # NGINX 虚拟主机，监听80端口
	    server {
	        listen 80;
	        server_name test.com;
	        access_log /var/log/nginx/node-server;

	        # Gzip Compression
	        gzip on;
	        gzip_comp_level 6;
	        gzip_vary on;
	        gzip_min_length  1000;
	        gzip_proxied any;
	        gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;
	        gzip_buffers 16 8k;

	        # 反向代理 node-server
	        location / {
	          proxy_set_header X-Real-IP $remote_addr;
	          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	          proxy_set_header Host $http_host;
	          proxy_set_header X-NginX-Proxy true;
	          # 代理的地址
	          proxy_pass http://node-server;
	          proxy_redirect off;
	        }
	     }
	```

4. 启动`Nginx`
	以`Linux`为例：
	``` shell
	sudo service nginx start
	```
5.测试
![server](/images/blog/20160316/1.png) ![proxy](/images/blog/20160316/2.png)
可以看到，对于用户的请求，我们成功反向代理到`127.0.0.1:3000`上！

# 负载均衡
`负载均衡`，这个名称听起来刁刁的有木有！其实嘛，也就那回事，不要被这牛逼哄哄的名词吓住了。之前我们分析了`google.com`是怎么对请求进行分流的，现在我们就来小小的试验一下。

1. 另开启一个本地server
	``` shell
	http-server -p 3001
	```
	在此server的根目录下新建`index.html`，输入'hello, nginx (server 2)'

2. 配置nginx.conf
	修改之前的配置为：
	``` nginx
	    #本地http-server开启的server，命名为node-server，监听3000和3001端口
	    upstream node-server{
	    # weight表示权重，数值越大，表示被分配到这个server的几率越大，这里我们让其相等。
	        server 127.0.0.1:3000 weight=1;
	        server 127.0.0.1:3001 weight=1;
	    }

	    # NGINX 虚拟主机，监听80端口
	    server {
	        listen 80;
	        server_name test.com;
	        access_log /var/log/nginx/node-server;

	        # Gzip Compression
	        gzip on;
	        gzip_comp_level 6;
	        gzip_vary on;
	        gzip_min_length  1000;
	        gzip_proxied any;
	        gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;
	        gzip_buffers 16 8k;

	        # 反向代理 node-server
	        location / {
	          proxy_set_header X-Real-IP $remote_addr;
	          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	          proxy_set_header Host $http_host;
	          proxy_set_header X-NginX-Proxy true;
	          # 代理的地址
	          proxy_pass http://node-server;
	          proxy_redirect off;
	        }
	     }
	```
3. 重启Nginx
	以`Linux`为例：
	``` shell
	sudo nginx -s reload
	```
	或者
	``` shell
	sudo service nginx restart
	```
4. 测试
![server1](/images/blog/20160316/1.png) ![server2](/images/blog/20160316/3.png)

我们可以看到，对于同一个请求，nginx会按照权重随机的分配到不同的server！这样就完成了均衡负载。对于client来说，好像就只有一台server！

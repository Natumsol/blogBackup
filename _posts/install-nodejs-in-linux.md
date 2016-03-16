title: 记一次奇葩的NodeJS的安装经历
date: 2016-03-14 09:59:24
categories: 技术
tags:
    - Ubuntu
    - NodeJS
---
## 现象描述
因为通过`apt-get`的方式安装的`NodeJS`版本已经过时，于是我想更新到最新的LTS版本。然后下载了已经编译好的`NodeJS`二进制源码，通过硬链接的方式将`node-v4.4.0-linux-x64/bin/node`链接到`/usr/local/bin/node`，但是在运行的时候，总会报如下错：
```shell
The program 'nodejs' is currently not installed. You can install it by typing:sudo apt-get install nodejs
```

但是直接在`/usr/local/bin`下运行`./node`是可以的，真的是百思不得其解，不知道是哪里出现了问题。以为是`NodeJS`版本有问题，于是通过源码安装的方式，花费了N久的时间去编译，最后居然...还是不行⊙﹏⊙b汗，真是呵呵哒...

于是回过头来重新看错误信息，发现提示的信息为`nodejs`没有安装，我明明输入的是`node`啊，怎么执行的是`nodejs`呢？在电石火光的一瞬间，我猛的一拍大腿，突然醒悟:这都是自己之前作的死啊...我是怎么作死的，容我慢慢道来。
<!--more -->
## 原因分析
之前重转系统的时候，为了方便，直接用`sudo apt-get install nodejs`的方式来安装`NodeJS`，但是没想到运行的时候要键入命令`nodejs`，而不是我们在Windows下的`node`命令，真不习惯。于是我自作聪明的想到了`alias`命令，通过在`~/.bashrc`文件加入`alias node='nodejs'`来实现重命名...
![原因分析](/images/blog/20160314/1.png)

所以我每次输入的命令`node`被shell自动解析为`nodejs`，所以自然就找不到了...

## 后记
没事不要瞎折腾，不作死就不会死，哈哈哈哈～

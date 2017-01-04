title: 局域网共享Lantern
date: 2015-11-27 09:58:27
categories: 技术
tags:
  - Lantern
  -	局域网共享
------------

最近发现`Lantern`这货翻墙巨好用，看`youtube`720P高清都不带卡的。虽然横跨`Windows`，`Linux`，`Mac`三大桌面平台，但是`IOS`和`Android`平台目前还未有动作。为了实现手机上`Google`全家桶和`Facebook`、`Twitter`死而复生，不得不另辟他径。

经过一番调研之后，找了一个可行的方法。但是目前此方法存在问题，每次启动`lantern`后都要修改配置文件（因为`lantern`每次启动的时候都会重新写入配置。。汗）。下面介绍具体操作方法：

准备条件
--------

`PC`和移动设备在同一个局域网里面，这样才能利用`PC`作为代理服务器（前级代理）。

修改`lantern`监听地址
---------------------

1.	快捷键`Win + R`， 输入`%AppData%\Lantern`， 回车。
2.	在资源管理器里找到`lantern-2.0.10.yaml`文件，用文本编辑器打开，找到`addr: 127.0.0.1:8787`这一行，将其更改为`addr: 0.0.0.0:8787`，并保存。

修改移动端代理
--------------

设置手机的`WiFi`代理： 1. 代理`IP`填写`PC`的`IP`地址。 2. 端口号填写`8787`。 3. 打开`Google`，`Facebook`，`Twitter`尽情的爽吧，哈哈！

注意事项
--------

因为`Lantern`启动的时候会重新写入配置文件，之前的配置文件会被覆盖掉。我试过将配置文件设定为只读属性，不让`Lantern`覆盖，但是这样`Lantern`会启动失败。。。汗。在`lantern`的移动版本没到来之前，凑合着用吧~

更新
-------
`Lantern`的 [安卓版](https://raw.githubusercontent.com/getlantern/lantern-binaries/master/lantern-installer-beta.apk)已经正式上线了，大家可以不用这个折腾了~
参考资料
--------

-	[“如何翻墙”系列：Lantern（蓝灯）——开源且跨平台的翻墙代理](https://program-think.blogspot.com/2015/08/gfw-lantern.html)
-	[多台电脑如何共享翻墙通道](https://program-think.blogspot.com/2013/01/cross-host-use-gfw-tool.html)
-	[如何在局域网中共享LANTERN来翻墙？](https://github.com/getlantern/lantern/issues/2940)

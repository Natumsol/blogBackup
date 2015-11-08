title: 解决Sublime-text 2在ubuntu 14.04 X64 下中文输入问题
date: 2015-09-09 09:26:45
categories: 技术
tags: 
	- Sublime Text2
	- Ubuntu
---
最近把实验室的工作环境换为Ubuntu 14.04（嘘！），配置`Ubuntu`真是一件苦逼的事，虽说这种事我白做不厌（嗯，不能告诉别人这是我的逗比属性之一）。`Sublime-text 2`一直是我最钟爱的编辑器，不过这货对中文支持似乎不太又好惹，不仅没有官方的中文版本，而且对中文输入法的支持也很烂惹。好吧，今天我就作死吐槽下： 
* 在`Windows`下输入中文输入框无法更随光标，这个bug好解决，直接安装`IMESupport`就可以fix了；
* 在`Ubuntu`下是直接不能输入中文（天啦撸，不能搞种族歧视啊，sublime君）。。

作为一个潮码农，经常会升级系统，重装系统，为了防止每次都`Google`一下（话说阅兵后`Google`香港可以正常访问了，难道是回光返照？），这就是我写这篇文章的意图（什么？为什么不用印象笔记呢？ 这位兄台，咳咳，有句古话说得好：阻人装逼如杀人父母...）
<!-- more -->
具体解决方法为：

## 保存下述代码为 sublime-imfix.c 文件

``` cpp
// sublime-imfix.c
// Use LD_PRELOAD to interpose some function to fix sublime input method support for linux.
// By Cjacker Huang <jianzhong.huang at i-soft.com.cn>
 
// gcc -shared -o libsublime-imfix.so sublime_imfix.c  `pkg-config --libs --cflags gtk+-2.0` -fPIC
// LD_PRELOAD=./libsublime-imfix.so sublime_text

#include <gtk/gtk.h>
#include <gdk/gdkx.h>
typedef GdkSegment GdkRegionBox;
 
struct _GdkRegion
{
  long size;
  long numRects;
  GdkRegionBox *rects;
  GdkRegionBox extents;
};
 
GtkIMContext *local_context;
 
void gdk_region_get_clipbox (const GdkRegion *region,
            GdkRectangle    *rectangle)
{
  g_return_if_fail (region != NULL);
  g_return_if_fail (rectangle != NULL);
 
  rectangle->x = region->extents.x1;
  rectangle->y = region->extents.y1;
  rectangle->width = region->extents.x2 - region->extents.x1;
  rectangle->height = region->extents.y2 - region->extents.y1;
  GdkRectangle rect;
  rect.x = rectangle->x;
  rect.y = rectangle->y;
  rect.width = 0;
  rect.height = rectangle->height; 
  //The caret width is 2; 
  //Maybe sometimes we will make a mistake, but for most of the time, it should be the caret.
  if(rectangle->width == 2 && GTK_IS_IM_CONTEXT(local_context)) {
        gtk_im_context_set_cursor_location(local_context, rectangle);
  }
}
 
//this is needed, for example, if you input something in file dialog and return back the edit area
//context will lost, so here we set it again.
static GdkFilterReturn event_filter (GdkXEvent *xevent, GdkEvent *event, gpointer im_context)
{
    XEvent *xev = (XEvent *)xevent;
    if(xev->type == KeyRelease && GTK_IS_IM_CONTEXT(im_context)) {
       GdkWindow * win = g_object_get_data(G_OBJECT(im_context),"window");
       if(GDK_IS_WINDOW(win))
         gtk_im_context_set_client_window(im_context, win);
    }
    return GDK_FILTER_CONTINUE;
}
void gtk_im_context_set_client_window (GtkIMContext *context, GdkWindow *window)
{
  GtkIMContextClass *klass;
  g_return_if_fail (GTK_IS_IM_CONTEXT (context));
  klass = GTK_IM_CONTEXT_GET_CLASS (context);
  if (klass->set_client_window)
    klass->set_client_window (context, window);
 
  if(!GDK_IS_WINDOW (window))
    return;
  g_object_set_data(G_OBJECT(context),"window",window);
  int width = gdk_window_get_width(window);
  int height = gdk_window_get_height(window);
  if(width != 0 && height !=0) {
    gtk_im_context_focus_in(context);
    local_context = context;
  }
  gdk_window_add_filter (window, event_filter, context); 
}
```

##  安装C/C++的编译环境和gtk libgtk2.0-dev 
	``` sh
	sudo apt-get install build-essential
	sudo apt-get install libgtk2.0-dev
	```

##  编译共享内库
	``` sh
	gcc -shared -o libsublime-imfix.so sublime-imfix.c  `pkg-config --libs --cflags gtk+-2.0` -fPIC
	```

##  修改Sublime-text 2启动参数
### 就编译好的`libsublime-imfix.so`文件移动到`/usr/lib` 目录下
```sh
	sudo mv libsublime-imfix.so /usr/lib
```
### 修改sublime-text-2.desktop
```sh
	cd /usr/share/applications
	sudo vim sublime-text-2.desktop
```
打开sublime-text-2.desktop后，将
``` sh
	Exec=/usr/bin/subl %F
```
修改为
``` sh
	Exec=bash -c 'LD_PRELOAD=/usr/lib/libsublime-imfix.so /usr/bin/subl' %F
```
将
``` sh
	Exec=/usr/bin/subl 
```
改为
``` sh
	Exec=bash -c 'LD_PRELOAD=/usr/lib/libsublime-imfix.so /usr/bin/subl' -n
```
将
``` sh
	Exec=/usr/bin/subl --command new_file
```
改为
``` sh
	Exec=bash -c 'LD_PRELOAD=/usr/lib/libsublime-imfix.so /usr/bin/subl' --command new_file
```

## 重新启动Sublime-text 2， enjoy~
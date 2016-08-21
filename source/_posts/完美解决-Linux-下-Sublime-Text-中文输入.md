---
title: 完美解决 Linux 下 Sublime Text 中文输入
date: 2016-08-09 16:29:33
tags:
---

### 1.保存代码为sublime-imfix.c文件

```bash
/*
sublime-imfix.c
Use LD_PRELOAD to interpose some function to fix sublime input method support for linux.
By Cjacker Huang
gcc -shared -o libsublime-imfix.so sublime-imfix.c `pkg-config --libs --cflags gtk+-2.0` -fPIC
LD_PRELOAD=./libsublime-imfix.so subl
*/
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
void
gdk_region_get_clipbox (const GdkRegion *region,
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
void gtk_im_context_set_client_window (GtkIMContext *context,
          GdkWindow    *window)
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

### 2.安装 C/C++ 的编译环境和 gtk libgtk2.0-dev

```bash
sudo apt-get install build-essential
sudo apt-get install libgtk2.0-dev
```

### 3. 编译共享类库

```bash
gcc -shared -o libsublime-imfix.so sublime-imfix.c `pkg-config --libs --cflags gtk+-2.0` -fPIC
```

### 4.然后将libsublime-imfix.so拷贝到sublime_text所在文件夹

```bash
sudo mv libsublime-imfix.so /opt/sublime_text/
```

### 5.修改文件/usr/bin/subl的内容

```bash
sudo gedit /usr/bin/subl
```
将
```bash
#!/bin/sh
exec /opt/sublime_text/sublime_text "$@"
```
修改为
```bash
#!/bin/sh
LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so exec /opt/sublime_text/sublime_text "$@"
```
此时，在命令中执行 subl 将可以使用搜狗for linux的中文输入

### 6.为了使用鼠标右键打开文件时能够使用中文输入，还需要修改文件sublime_text.desktop的内容
```bash
sudo gedit /usr/share/applications/sublime_text.desktop
```
将[Desktop Entry]中的字符串
```bash
Exec=/opt/sublime_text/sublime_text %F
```
修改为
```bash
Exec=bash -c "LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so exec /opt/sublime_text/sublime_text %F"
```
将[Desktop Action Window]中的字符串
```bash
Exec=/opt/sublime_text/sublime_text -n
```
修改为
```bash
Exec=bash -c "LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so exec /opt/sublime_text/sublime_text -n"
```
将[Desktop Action Document]中的字符串
```bash
Exec=/opt/sublime_text/sublime_text --command new_file
```
修改为
```bash
Exec=bash -c "LD_PRELOAD=/opt/sublime_text/libsublime-imfix.so exec /opt/sublime_text/sublime_text --command new_file"
```

>注意： 修改时请注意双引号"",否则会导致不能打开带有空格文件名的文件。 此处仅修改了/usr/share/applications/sublime-text.desktop，但可以正常使用了。 opt/sublime_text/目录下的sublime-text.desktop可以修改，也可不修改。












---
title: python win32api 学习笔记
abbrlink: 57357
tags:
---

通过窗口类名获取窗口句柄:

`win32gui.FindWindow("ClassName",None)`

获取当前窗口句柄:

`win32gui.GetForegroundWindow()`

通过窗口句柄获取窗口类名:

`win32gui.GetClassName(hwnd)`

通过窗口句柄获取窗口名

`win32gui.GetWindowText(hwnd)`

关闭指定句柄的窗口:

`CloseWindow(hwnd)`

新建窗口:

`os.system('start cmd')`

遍历窗口名和窗口句柄:

```
def get_hwnd():
    hwnd_title = dict()

    def get_all_hwnd(hwnd, mouse):
        if win32gui.IsWindow(hwnd) and win32gui.IsWindowEnabled(hwnd) and win32gui.IsWindowVisible(hwnd):
            hwnd_title.update({hwnd: win32gui.GetWindowText(hwnd)})

    win32gui.EnumWindows(get_all_hwnd, 0)

    for h, t in hwnd_title.items():
        if t is not "":
            print(h, t)
```

截屏:


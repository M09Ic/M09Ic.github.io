---
title: 刷SRC的实用小技巧-自动截图
categories: 编程
tags: 学习笔记
abbrlink: 14990
date: 2019-08-04 20:06:12
---

*本文原发于<https://www.secquan.org/Notes/1070004>,仅在自己博客做个备份*

## 写在之前

各大网站以及SRC之类的地方提交漏洞免不了需要截图,手动截图费时费力,我也深受其扰.使用python调用windows api实现自动截图才是正道.可能有很多师傅已经知道了,但是还是厚颜无耻写下.

## 实现

思路很简单,呼出一个新窗口,运行poc,截图运行成果.适用于新漏洞快速刷钱.代码实现也不难.直接进入正题.

首先需要安装python 和 pywin32这个库(具体安装方法自行查阅百度google).

然后需要通过windows api实现截图功能.

因为批量刷,少不了大量后台运行的子进程,用一些python图形库截图只能截显示屏显示的内容.想要截到非活跃页面的图,只能用windows api.

pywin32这个库就封装了大量windows api操作.

首先需要实现截图功能.

直接上关键代码,这个代码是网上抄来的稍微修改了一下.第一个参数填截图的保存路径,第二个参数填窗口句柄(window handle)

```python
def window_capture(filename,hwnd):
 	# 窗口的编号，0号表示当前活跃窗口
    # 根据窗口句柄获取窗口的设备上下文DC（Divice Context）
    hwndDC = win32gui.GetWindowDC(hwnd)
    # 根据窗口的DC获取mfcDC
    mfcDC = win32ui.CreateDCFromHandle(hwndDC)
    # mfcDC创建可兼容的DC
    saveDC = mfcDC.CreateCompatibleDC()
    # 创建bigmap准备保存图片
    saveBitMap = win32ui.CreateBitmap()
    # 获取监控器信息
    MoniterDev = win32api.EnumDisplayMonitors(None, None)
    rect = win32gui.GetWindowRect(hwnd)

    w = rect[2]-rect[0]
    h = rect[3]-rect[1]
    # print w,h　　　#图片大小
    # 为bitmap开辟空间
    saveBitMap.CreateCompatibleBitmap(mfcDC, w, h)
    # 高度saveDC，将截图保存到saveBitmap中
    saveDC.SelectObject(saveBitMap)
    # 截取从左上角（0，0）长宽为（w，h）的图片
    saveDC.BitBlt((0, 0), (w, h), mfcDC, (0, 0), win32con.SRCCOPY)
    saveBitMap.SaveBitmapFile(saveDC, filename)
```

但是windows api没有获取当前代码运行窗口的窗口句柄,因此,只能绕路.通过遍历窗口的pid进行对比,反查到句柄.上代码(也是抄的).

```python
def get_hwnds_for_pid(pid):
    def callback(hwnd, hwnds):
        if win32gui.IsWindowVisible(hwnd) and win32gui.IsWindowEnabled(hwnd):
            _, found_pid = win32process.GetWindowThreadProcessId(hwnd)
            if found_pid == pid:
                hwnds.append(hwnd)
        return True

    hwnds = []
    win32gui.EnumWindows(callback, hwnds)
    return hwnds
```

需要注意的是,这里有个坑.因为如果调用cmd运行poc,会生成一个子进程,直接`os.getpid()`获取的是子进程的pid.需要注意获取的pid不能是子进程的pid,而是获取`os.getppid()`.

将以上代码放在同一个文件里,调用系统命令运行poc文件. 把`window_capture()`放在末尾行,如果需要还可以写一个导出报告的函数.

这里又有一个小坑. 最好在前面加上一个`time.sleep(1)`.因为比如git bash或pycharm的输出不是实时的,可能导致已经截图了,但是输出还没出来.经过测试cmd和powershell不会有这种情况.

我们需要的肯定是批量,刚才的能做到的只是单个.

所以需要一些操作.在另外一个文件中调用cmd,运行刚才这个文件.

比如`subprocess.Popen("start python poc.py -t %s -p %s" % (ip, port), shell=True)`

windows cmd中可以用start创建一个新窗口,并且该窗口是独立的进程.因此在`poc.py`就只需要`os.getpid()`.

如果需要批量,使用for循环即可.

效果如下:

![1](1.png)

![2](2.png)




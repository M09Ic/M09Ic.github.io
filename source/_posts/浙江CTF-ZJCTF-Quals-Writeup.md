---
title: 浙江CTF(ZJCTF) Quals Writeup
categories: CTF&AWD
tags:
  - writeup
abbrlink: 50801
date: 2019-09-12 17:08:51
---

*本文原发于sec圈 : <https://www.secquan.org/Discuss/1070283> ,博客转自本人仅做备份用途*

## WEB

### 签到题略

### 黑白分明

![1568279431617](1.png)

一个js写的游戏,成绩要求600000分以上

```javascript
view = function (score, token) {
    // Too young to simple
    if (score < 600000) {
        return "hello kitty";
    }
    // Quickly tell me, I`m thirsty to death.
    var http = new XMLHttpRequest();
    var url = "/game/push";
    var params = "token=" + token;
    http.open("post", url, true);
    // Send the proper header information along with the request
    http.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
    http.onreadystatechange = function() {
        if(http.readyState == 4 && http.status == 200) {
            // Tell u the f-l-a-g...
        }
    }
    http.send(params);
    return "hello world";
}
```

抓包提交就完事了,然后就有个天坑.因为有反作弊机制

![1568279431617](2.jpg)

换多几台电脑,用多几个浏览器,就成功了.

毒瘤!

### 内容包含

又是包含,又是给临时文件,还以为通过session进行文件包含

![1568279431617](3.png)

试了半天,没找到包含的攻击点.

经过学弟提示,发现shtml存在ssi注入.

<https://www.secpulse.com/archives/66934.html>

没有任何过滤.

`<!--#exec cmd="cat ../flag" -->`

成功getflag

![1568279431617](4.png)

### 大宽碗面

给了一个输入框,过滤了单引号,如果不是其他什么骚操作,就是宽字节注入了.

![1568279431617](6.png)

需要提一下,如果只输入单引号,虽然会有不同的返回值,但实际上并没有过滤,语句还是执行了.

常规操作来说是爆库名表名,但是select from被过滤了.

接下来就是一通各种各样的绕过技巧的尝试.但是又不支持堆叠查询,很多可以不用select的注入姿势用不了,比如`SET` 定义语句再`EXECUTE`执行,或`SHOW column from tables`都不能使用.

陷入僵局.

第二天学弟给我发了张图.

![1568279431617](7.png)

我测试了下,这样也可以

![1568279431617](8.png)

猜测,该waf是外挂在代码之外的,有检测参数上限,并且以`&`为分隔符.

当`&`超过100时,多于的参数就直接传入.

## CRYPTO

#### 非黑即白

xor加密,老套路,护网被2018有类似原题fez.

核心加密函数如下,`gen_keys`是固定的:

```python
def zjctf_encrypt(gen_keys, hahahah):
    i = 0
    d1 = hahahah[:8]
    d2 = hahahah[8:]
    for i in gen_keys:
        d1 = xor(xor(HASH(d2), i), d1)
        print('d1:'+d1.encode('hex'),'d2:'+d2.encode('hex'))
        d1, d2 = d2, d1
    return d2 + d1
```

![1568279431617](5.png)

可以发现有规律.最后一次`HASH(d2)`和i是已知的,`a=a^b^a`

因此,可以一次次倒推flag,解密代码如下

```python
def decrypt():
    keys = ['w\xed\xe5wJ\xb1B\x87', 'G\xb8:\xaa\xf7f\xcd\xe9', '\xa2\xdd=\x01\xfe\xba\x12\xdb', 'j[\xdb\x16\xf5\x7ft\xa2', '\xe7x$\xda\xde\xf1\xfd\x05', '\xf3\xf1O\x837\xc0~\xea', '"\xa9\xf7Y\t}R:', 'G\x87\xcb\xa9Z%\xda\xd2', '\xef\x8c\xaa\x95\x16u\xbf6', '\xa8\xe3C\x0b1\xdd\x90\x95', '\xacZ\x10\xdeD\x8c\xa0\x0c', '\x88K\xa8\xd4\x95\xf1|\xec', '\x97\xbe<\xd2F\xc2\xceh', '\xa9\x8b\x88\x03\x83!k\x02', 'c\xd4\xca\xc7\xaa,Tf', 'p\xd1\xbf0\x02E\x0e\x1e']

    d1= '3a6dc48f313deb27'.decode('hex')
    d2= '07204bdf8af35c65'.decode('hex')
    for i in keys[::-1]:
        d1 = xor(xor(d1,HASH(d2)),i)

        d1 ,d2 = d2, d1
    return d2+d1
```

## Reverse

### 序列变换

apk拖到apkkiller里,发现transfrom.smail比较奇怪.右键查看java代码,如下:

```java
package com.example.zjctf_android;

public class tansform
{
  public static String transformSeq(String paramString)
  {
    int j = paramString.length();
    StringBuilder localStringBuilder = new StringBuilder(paramString);
    int i = 0;
    for (;;)
    {
      if (i >= j) {
        return localStringBuilder.toString();
      }
      if ((paramString.charAt(i) >= 'A') && (paramString.charAt(i) <= 'Z')) {
        localStringBuilder.setCharAt(i, (char)(paramString.charAt(i) + '\001'));
      }
      if ((paramString.charAt(i) >= 'a') && (paramString.charAt(i) <= 'z')) {
        localStringBuilder.setCharAt(i, (char)(paramString.charAt(i) - '\001'));
      }
      i += 1;
    }
  }
}

```

可以看到如果是大写ascii加一,如果是小写ascii减一

又根据题目提示,解包看到pwd.txt,需要其中第12701行.找到它

因为不太会写java,所以用python来解密

```python
import string
s = 'j4nlO512Y82Swe44CoNlVzWM'
def tran(s):
    ans = []
    for i in s:
        if i in string.uppercase:
            ans.append(chr(ord(i)+1))
        elif i in string.lowercase:
            ans.append(chr(ord(i)-1))
        else:
            ans.append(i)
    ans = ''.join(ans)
    return ans
#flag: i4mkP512Z82Tvd44DnOkWyXN
```



## 小结

第一次参加的ctf就是两年前的杭电ctf,当时基本啥都不会.

现在又遇到这种难度的比赛,有种守得云开见月明的感觉,大部分题目都思路清晰,还有几道没接出来的题目也都有线索,但因知识积累不够未能解开.

比如流量题目,很快就找到了隐藏的破损的二维码,但是不熟悉二维码算法无法还原.

解开wp中的这几道题目甚至只用了一个小时.
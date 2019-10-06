---
title: Ogeek Final Writeup
tags:
  - Writeup
abbrlink: 56679
---



靠着大腿混进决赛.膜ex师傅,膜impakho师傅

### web1

题目是一个java web,tomcat框架,不太熟悉

随手翻了下目录,一眼就看到shiro1.2.4 , 反序列化漏洞.

还以为是白给,跟着网上步骤复现了一遍,失败了.仔细看了源码,发现出题人重写了cookie的加密函数.

暂时放弃,直到有队成功攻击了我们一次,才着手防守.

防守并不困难,修改硬编码的rememberkey ,后续再也没有被攻击.

### web2

好像是zero-cms,有代码备注时间在2009年,但是网上并没有找到相关信息.


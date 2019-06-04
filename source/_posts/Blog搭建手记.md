---
title: hexo博客搭建手记
abbrlink: 64867
date: 2019-05-17 09:40:39
categories: 其他
tags: 
- Blog
- 学习笔记
---

## 0x00 开端

搭建博客有两个主要目的,一个是学过的东西过个半年基本都忘了,特别是一些细节的东西,浏览器书签收藏了一堆,也基本没再打开过.另一方面也是想分享一些基本的东西,最开始会以总结为主.自己现在也有一些比较简单的研究,等更加深入一点就分享出来(太菜了,别打我)

博客内容主要会以安全方向为主.

选博客框架,首先就把wordpress这种笨重的东西排除了,不优雅.静态博客框架主要以Hexo , Jekyll,其他的不太了解.不选用jekyll主要是因为主题太多了,选择困难症发作,看了一个小时还没开始动手.还是hexo好,比较流行的主题next,功能强大,拓展丰富,唯一的缺点就是用的人太多,逼格稍微降低了点.这都是可以自己定制解决的事情.

写这篇文章也不是教学向,毕竟都是网上都有的东西,单纯的记录下博客搭建的过程.

## 0x01 搭建

blog搭建主要参考了这两篇文章:

[打造个性超赞博客Hexo+NexT+GitHubPages的超深度优化](<https://io-oi.me/technology/computer/computer-aided-art/2017/06/09/hexo-next-optimization.html>)

[hexo史上最全搭建教程](<https://blog.csdn.net/sinat_37781304/article/details/82729029>)

### 安装

我主要还是在windows下写文章.所以只写windows下安装过程.linux其实更简单.

首先去官网下载[git](<https://git-scm.com/>)和[nodejs](<https://nodejs.org/zh-cn/>)

nodejs windows安装包自带npm,也就不用特别安装了.

安装好这两个工具后安装hexo,git bash 执行

`npm install -g hexo-cli`

初始化hexo,并且安装依赖

```
hexo init blogname
cd blogname
npm install
```

hexo 常用命令只有这么几个,也比较好记

```
init 							# 初始化博客
hexo new [layout] <title>		# 发表新内容
publish [layout] <filename>		# 发布草稿
clean							# 清理缓存
generate|g 						# 生成静态文件
deploy|d						# 发布到github
server							# 本地运行
```



### github

注册账号不用说,需要注意一下创建的仓库名要和你的账号同名.比如我的github叫`M09Ic`,那仓库名就得叫`M09Ic.github.io`,不然不会自动识别github page.

github 添加ssh key就不多说了.

配置完ssh key,在`_config.yml`中添加

```
deploy:
  type: git
  repo: https://github.com/M09Ic/M09Ic.github.io.git
  branch: master
```

在执行`hexo g -d`就可以发布博客了.

### 基础配置

[配置文件说明](<https://hexo.io/zh-cn/docs/configuration.html>)

大部分内容保持默认即可,要改的只有

`# site`

下的内容都是博客的基本设置,如标题,副标题,描述,作者等,按需填写.

```
title: M09ic's Blog
subtitle: 独自行走于莽荒之地
description: 网络安全,编程学习记录的破地方
keywords: 安全,学习笔记,blog,博客
author: M09ic
language: zh-CN
url: https://m09ic.top/  
```

下载[next主题](<https://github.com/iissnan/hexo-theme-next>)

解压到`themes`文件夹下,记得把`.git`文件删了,不然可能会报错.

然后主题改成`theme: next`

### 连接固定

默认连接是年月日的多层子目录,不利于seo,也不美观.所以用`hexo-abbrlink`插件固定连接.

执行命令`npm install hexo-abbrlink --save`

修改hexo配置文件:`permalink: posts/:abbrlink/`

这样文章的链接就是如`https://m09ic.top/posts/48529`

## 0x02 NexT

修改next主题的`_config.yml`

这里注意一下,hexo和next的配置文件都叫`_config.yml`,我会注明修改的是哪个文件,比如hexo的配置文件或next的配置文件.

### favicon

自定义favicon

```
favicon:
  small: /images/favicon16.png
  medium: /images/favicon32.png
```

### footer

显示采用hexo和NexT,但是关闭hexo和主题版本信息

```
  powered:
    # Hexo link (Powered by Hexo).
    enable: true
    # Version info of Hexo after Hexo link (vX.X.X).
    version: false

  theme:
    # Theme & scheme info link (Theme - NexT.scheme).
    enable: true
    # Version info of NexT after scheme info (vX.X.X).
    version: false
```

### creative_commons

知识共享协议,署名+相同方式共享+非商业使用.

虽然没啥内容质量也低,但是态度还是有的,支持知识共享.

```
creative_commons:
  license: by-nc-sa
  sidebar: true
  post: true
```

### menu

默认只有两栏,添加多几栏.

```
menu:
  home: / || home
  about: /about/ || user
  tags: /tags/ || tags
  categories: /categories/ || th
  archives: /archives/ || archive
```

### scheme

自带四种风格,我喜欢`Gemini`的左右两栏的设计,取消对应那行的注释即可.

```
scheme: Gemini
```

### social

添加社交账号

```
social:
  GitHub: https://github.com/M09Ic || github
  E-Mail: mailto:n8c4899812@gmail || envelope
```

### 图标字体

这是一个小坑,我的解决方法官方稍微有点不同.

上传到github的时候vendors会被过滤,导致font awesome不可使用.

手动将next主题下的`font-awesome`文件夹复制到`next/source/lib/`下

然后修改next配置文件:

```
  fontawesome: /lib/font-awesome/css/font-awesome.min.css
```

### RSS

执行命令安装插件:

```
npm install hexo-generator-feed --save
```

修改hexo配置文件:

```
Plugins:
- hexo-generate-feed
```

修改next配置文件:

```
rss: /atom.xml
```



## 0x03 第三方服务

### 评论

因为不喜欢评论还需要登录账号之类的操作,所以用valine.

去valine官网上注册一下,把id和key填一下就好了

```
valine:
  enable: true 
  appid: *********************************
  appkey: **********************
  notify: false 
  verify: false 
  placeholder:  
  avatar: mm 
  guest_info: nick
  pageSize: 10 
  language: 
  visitor: false 
  comment_count: false 

```

### 访问量统计

```
busuanzi_count:
  enable: true
  total_visitors: true
  total_visitors_icon: user
  total_views: true
  total_views_icon: eye
  post_views: true
  post_views_icon: eye
```

本来还想添加站内搜索,但是与主题UI风格不搭,就放弃了.

如果要搜索,用google 高级语法吧,搞安全的没有人不会用吧.

## 0x04 SEO

### 域名

既然搭建博客了,就肯定要自定义域名,本想买godaddy,因为不用备案,但是价格原因买了阿里云,百来块钱三年.

然后阿里云配置域名解析,填两个CNAME记录,主机记录填`www`和`@`记录值都填`m09ic.github.io`

这里有个小坑

如果要自定义非www的二级域名,就不能从一级域名自动跳转.

 如果不填写www，如`example.com`，那么无论是访问`http://www.example.com`还是`http://example.com`，都会自动跳转到`http://example.com`  

如果填写www，如`www.example.com`，那么无论是访问`http://www.example.com`还是`http://example.com`，都会自动跳转到`http://www.example.com`  

如果填写的是其它子域名，如`abc.example.com`，那么访问`http://abc.example.com`没问题，但是访问`http://example.com`，不会自动跳转到`http://abc.example.com`

本来想自定义二级域名`blog.m09ic.top`但是因为不会自动跳转就放弃了.选用一级域名作博客的域名.

然后需要`source`下填写创建一个文件名为`CNAME`的文件,里面填`m09ic.top`

并且在`_config.yml`修改为`url: https://m09ic.top/  `

### config

打开一些next自带的seo优化配置

```
canonical: true

seo: true

index_with_subtitle: true

exturl: true

baidu_push: true
```

### sitemap

添加sitemap,需要安装两个插件,一个是google的一个是百度的.执行:

```
npm install hexo-generator-sitemap --save
npm install hexo-generator-baidu-sitemap --save
```

在hexo的`_config.yml`添加

```
Plugins:
- hexo-generator-baidu-sitemap
- hexo-generator-sitemap

sitemap: 
  path: sitemap.xml
baidusitemap: 
  path: baidusitemap.xml
```

然后注册google seo,验证网站所有权后提交sitemap.

百度也类似操作,但是现在要上传身份证之类乱七八糟的实名验证,就不想弄了.

### robots

在hexo的source下添加`robots.txt`文件,添加如下内容:

```
User-agent: *
Allow: /
Allow: /archives/
Allow: /categories/
Allow: /tags/

Disallow: /vendors/
Disallow: /js/
Disallow: /css/
Disallow: /fonts/
Disallow: /vendors/
Disallow: /fancybox/

Sitemap: https://m09ic.top/sitemap.xml
Sitemap: https://m09ic.top/baidusitemap.xml
```

### 域名

既然搭建博客了,就肯定要自定义域名,本想买godaddy,因为不用备案,但是价格原因买了阿里云,百来块钱三年.

然后阿里云配置域名解析,填两个CNAME记录,主机记录填`www`和`@`记录值都填`m09ic.github.io`

这里有个小坑

如果要自定义非www的二级域名,就不能从一级域名自动跳转.

 如果不填写www，如`example.com`，那么无论是访问`http://www.example.com`还是`http://example.com`，都会自动跳转到`http://example.com`  

如果填写www，如`www.example.com`，那么无论是访问`http://www.example.com`还是`http://example.com`，都会自动跳转到`http://www.example.com`  

如果填写的是其它子域名，如`abc.example.com`，那么访问`http://abc.example.com`没问题，但是访问`http://example.com`，不会自动跳转到`http://abc.example.com`

本来想自定义二级域名`blog.m09ic.top`但是因为不会自动跳转就放弃了.选用一级域名作博客的域名.

然后需要`source`下填写创建一个文件名为`CNAME`的文件,里面填`m09ic.top`

并且在`_config.yml`修改为`url: https://m09ic.top/  `

### Analytics

去百度和google的analytics注册,会拿到一串key.填进去就好了

Google Analytics:

```
google_analytics:
  tracking_id: UA-**********-1
  localhost_ignored: true
```

Baidu Analytics:

```
baidu_analytics: *******************************
```

## 0x05 多端同步

之前就已经创建了一个Github repo ,现在需要新建一个branch, 比如叫`hexo`(默认的branch叫master),然后将默认branch修改为`hexo`.

然后clone 一个,会选择默认分支clone.

再将文件复制到刚刚clone的文件夹下.记得添加`.gitigore`,内容如下:

```
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
```

再执行:

```
git add . 
git commit -m "update"
git push 
```


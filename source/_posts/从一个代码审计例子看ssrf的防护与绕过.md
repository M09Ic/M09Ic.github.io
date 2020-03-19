---
title: 从一个代码审计例子看ssrf的防护与绕过
tags:
  - 代码审计
abbrlink: 5871
date: 2019-12-17 13:09:47
categories: 网络安全

---

(本文首发于先知社区,链接:https://xz.aliyun.com/t/7256)

## 前言

最近开始实战代码审计,去github上找了几个100+ stars 的项目练手.

看到有个cms中有个SSRF漏洞点多次被绕过,修复再被绕过.打开commit记录浏览,测试了一番,发现还能再次绕过.

绕过技巧并不新颖,2017年blackhat上orange师傅演讲的:[ A New Era of SSRF ]( https://www.blackhat.com/docs/us-17/thursday/us-17-Tsai-A-New-Era-Of-SSRF-Exploiting-URL-Parser-In-Trending-Programming-Languages.pdf )中提出的.

## 正文

### 0x01  代码审计中的信息收集

一个cms代码量确实不少,通读代码耗时长,效果也不一定好.而一个功能点如果之前出过漏洞,开发者对安全的理解又不充分,很容易绕过修复.这就类似一道ctf题目,对于经常参加ctf的各位大佬来说,这样的代码审计更加简单休闲.

我以icms为例.

icms github链接: https://github.com/idreamsoft/iCMS 

在issue中搜索SSRF [https://github.com/idreamsoft/iCMS/issues?utf8=%E2%9C%93&q=is%3Aissue+ssrf](https://github.com/idreamsoft/iCMS/issues?utf8=✓&q=is%3Aissue+ssrf) 

![image-20191115200723191](image-20191115200723191.png)

在cve列表中查找,应该对应的就是这三个cve了

![image-20191115200653356](image-20191115200653356.png)

可以看到这个功能点已经出现了三次的绕过与过滤.

大致了解下这个功能点,是一个自动更新文章的爬虫,多处都可以控制url参数.

点开issue查看具体信息,我们从最早出现漏洞的版本看起.

通过查看具体的commits,可以找到开发者修复漏洞的思路.这给我们代码审计带来很大的便利.

### CVE-2018-14514 漏洞分析

 commit: https://github.com/idreamsoft/iCMS/issues/29 

提交者详细描述了漏洞信息,只提交了一个点,但根据作者修复的commit,应该有两处都存在ssrf漏洞.

``icms7.0.9\app\spider\spider.class.php` 361行 ,关键代码:

```php
    public static function postUrl($url, $data) {
        is_array($data) && $data = http_build_query($data);
        $options = array(
            CURLOPT_URL    => $url,
			...
        );

        $ch = curl_init();
        curl_setopt_array($ch,$options);
        $responses = curl_exec($ch);
        curl_close ($ch);
        return $responses;
    }

```

与`icms7.0.9\app\spider\spider_tools.class.php`  604行,关键代码:

```php
public static function remote($url, $_count = 0) {
    $url = str_replace('&amp;', '&', $url);
    if(empty(spider::$referer)){
        $uri = parse_url($url);
        spider::$referer = $uri['scheme'] . '://' . $uri['host'];
    }
    self::$curl_info = array();
    $options = array(
        CURLOPT_URL                  => $url,
		...
    );
    spider::$cookie && $options[CURLOPT_COOKIE] = spider::$cookie;
    if(spider::$curl_proxy){
        $proxy   = self::proxy_test();
        $proxy && $options = iHttp::proxy($options,$proxy);
    }
    if(spider::$PROXY_URL){
        $options[CURLOPT_URL] = spider::$PROXY_URL.urlencode($url);
    }
    $ch = curl_init();
    curl_setopt_array($ch,$options);
    $responses = curl_exec($ch);
    ...
}
```

可以看到icms7.0.9版本没有做任何的验证.可以直接使用所有协议访问所有ip与端口.因此如果有redis或无密码的mysql或者一些其他容易被攻击的服务,可以getshell.因为这里重点不是ssrf getshell ,因此不做验证.

很多处都调用了remote函数,全局搜索即可,我们找一个能即时回显的点测试.

payload:`http://ip/admincp.php?app=spider&do=testdata&url=dict://127.0.0.1:8000&rid=2&pid=0&title=m09ic`

![image-20191115210301369](image-20191115210301369.png)

监听本地端口是否有数据过来.

![image-20191115205544907](image-20191115205544907.png)

很明显收到了.

### CVE-2018-14514 漏洞修复

我们再来看看作者是如何修复的,commit: https://github.com/idreamsoft/iCMS/commit/64bb0bdf77febbd6ac0ccb6658ee1ddc71530bb1

```php
public static function remote($url, $_count = 0) {
    if(!iHttp::is_url($url,true)){
        if (spider::$dataTest || spider::$ruleTest) {
            echo "<b>{$url} 请求错误:非正常URL格式,因安全问题只允许抓取 http:// 或 https:// 开头的链接</b>";
        }
        return false;
    }
```

作者添加了一个判断函数

```php
public static function is_url($url,$strict=false) {
    $url = trim($url);
    if($strict){
    return (stripos($url, 'http://') === 0 || stripos($url, 'https://') === 0);
    }

    if (stripos($url, 'http://') === false && stripos($url, 'https://') === false) {
    return false;
    } else {
    return true;
    }
}
```

先判断url是否以`http://`开头,才开始解析,这样就限制了乱七八糟协议,减轻了危害程度,大部分情况很难getshell.但是ssrf漏洞依然存在.

可以看到,作者对ssrf漏洞的认识并不到位,认为不能getshell就可以了.但是实际上,用http协议也并非完全不可能getshell,内网有可能存在一些可以被get请求getshell的服务,比如thinkphp最近的那几个RCE.

### CVE-2018-14858 漏洞修复

显然只允许http与https开头的url访问,ssrf依然存在,于是在icms7.0.11版本,又有人提交了ssrf漏洞,并获得了一个cve编号.漏洞分析与上一个漏洞一直,因此省略.

提交者除了提交漏洞,还简单说明了几种常见的ssrf绕过手法,比如不同格式的ip地址.

这是作者在[issue33]( https://github.com/idreamsoft/iCMS/issues/33 )下的回复.

>  I know this question, but if the IP format is banned, the website using the IP format will not be collected. Although it is not used a lot, it will still be encountered. There is no better way to think about it now. 

可以看到作者对ssrf的理解有些偏差,导致修复完全没有起到作用.

过了几天,作者似乎认识到了这样并不算修复了ssrf漏洞,commit了一下过滤代码.

具体更新内容在commit中: https://github.com/idreamsoft/iCMS/commit/62de04e57a67f2690dbf88b7d381af61a0969ef3 

添加了过滤代码,关键代码如下:

```php
public static function remote($url, $_count = 0) {
    if(!iHttp::is_url($url,true)){
        $parsed = parse_url($url);//解析url
        $validate_ip = true;
        preg_match('/\d+/', $parsed['host']) && $parsed['host'] = long2ip($parsed['host']);//获取host部分,如果是十进制或其他进制的ip地址,转化成标准的ip地址
        if(preg_match('/\d+\.\d+\.\d+\.\d+/', $parsed['host'])){
            $validate_ip = filter_var($parsed['host'], FILTER_VALIDATE_IP, FILTER_FLAG_NO_PRIV_RANGE | FILTER_FLAG_NO_RES_RANGE);//匹配正确的ip格式,过滤非法ip地址字符与内外地址
        }
        if(!in_array($parsed['scheme'],array('http','https')) || !$validate_ip|| strtolower($parsed['host'])=='localhost'){
            if (spider::$dataTest || spider::$ruleTest) {
                echo "<b>{$url} 请求错误:非正常URL格式,因安全问题只允许抓取 http:// 或 https:// 开头的链接</b>";
                echo "<b>{$url} 请求错误:非正常URL格式,因安全问题只允许抓取 http:// 或 https:// 开头的链接或私有IP地址</b>";
            }
            return false;
        }
    }
        ...
    
}
```

可以看到,这次添加了检查ip地址的格式,以及是否是内网ip.

以普通开发者的角度思考,很多情况都是哪里出了问题就修哪里,什么东西能绕过就过滤什么.也很难要求他们完全了解安全漏洞,因此也导致了修复再次被绕过.

### CVE-2018-15895

与上个漏洞提交者是同一个人. https://github.com/idreamsoft/iCMS/issues/40 

看似万无一失,更强的黑名单过滤,也确实不能通过绕过正则与`filter_var` 再次进行ssrf攻击.

但ssrf的常用绕过手法还有302重定向与dns重绑定.漏洞提交者也演示了这两种方式.具体poc可以看issue内容.

我们来看看作者的修复, https://github.com/idreamsoft/iCMS/commit/59ad0f303bf900f552f737b63c6fa8d92c1403d7 ,关键代码如下:

```php
    public static function safe_url($url) {
        $parsed = parse_url($url);
        $validate_ip = true;

        if($parsed['port'] && is_array(self::$safe_port) && !in_array($parsed['port'],self::$safe_port)){
            if (spider::$dataTest || spider::$ruleTest) {
                echo "<b>请求错误:非正常端口,因安全问题只允许抓取80,443端口的链接,如有特殊需求请自行修改程序</b>".PHP_EOL;
            }
            return false;
        }else{
            preg_match('/^\d+$/', $parsed['host']) && $parsed['host'] = long2ip($parsed['host']);
            $long = ip2long($parsed['host']);
            if($long===false){
                $ip = null;
                if(self::$safe_url){
                    @putenv('RES_OPTIONS=retrans:1 retry:1 timeout:1 attempts:1');
                    $ip   = gethostbyname($parsed['host']);
                    $long = ip2long($ip);
                    $long===false && $ip = null;
                    @putenv('RES_OPTIONS');
                }
            }else{
                $ip = $parsed['host'];
            }
            $ip && $validate_ip = filter_var($ip, FILTER_VALIDATE_IP, FILTER_FLAG_NO_PRIV_RANGE | FILTER_FLAG_NO_RES_RANGE);
        }

        if(!in_array($parsed['scheme'],array('http','https')) || !$validate_ip){
            if (spider::$dataTest || spider::$ruleTest) {
                echo "<b>{$url} 请求错误:非正常URL格式,因安全问题只允许抓取 http:// 或 https:// 开头的链接或公有IP地址</b>".PHP_EOL;
            }
            return false;
        }else{
            return $url;
        }
    }
```

可以看到,使用了第17行使用了`gethostbyname` 确定parse_url解析后的host部分,来防护dns rebinding 攻击.

并且在curl的options中,注释了`// CURLOPT_FOLLOWLOCATION => 1,// 使用自动跳转`,来防护302重定向绕过.

那么如果想再次绕过,经常打ctf的小伙伴可能就会注意到了,攻击的思路可以针对parse_url的解析问题上.历代parse_url有不少方式可以绕过scheme,host,port,path等.而这些细节,是开发者很难注意到的.

### 新的绕过

光黑名单和检查host真实ip来说,基本上是万无一失.但是作者万万没想到,来自php自身的背后一刀.在开头提到的[ A New Era of SSRF ]( https://www.blackhat.com/docs/us-17/thursday/us-17-Tsai-A-New-Era-Of-SSRF-Exploiting-URL-Parser-In-Trending-Programming-Languages.pdf )中,有一个新的攻击方式,利用php中的parse_url函数和libcurl对url的解析差异,导致了对host的过滤失效,进行ssrf.

从orange师傅的pdf中偷一张图来解释.

![image-20191116004557748](image-20191116004557748.png)



php-curl拓展解析的url host在第二个@之后,而parse_url则是最后一个@之后.

因此我们可以使用如下payload绕过:

`http://ip/admincp.php?app=spider_project&do=test&url=http://root:root@127.0.0.1:81@baidu.com/info.php&rid=2&pid=1&title=`


![image-20191116003035777](image-20191116003035777.png)

可以看到,同时绕过了端口和host的限制,访问到了只对本地开放的81端口的phpinfo内容.成功绕过过滤实现ssrf,~~并且水个cve编号~~



这里有一个小坑,在较新版本的curl中,已经修复了多个@的解析问题,使用多个@会报错.

![image-20191116005331010](image-20191116005331010.png)

但是大多数linux发行版并没有使用最新版本的curl.可以在 https://curl.haxx.se/download.html  这里查询linux发行版与curl版本的对应关系.

我只测试了ubuntu,在ubuntu16.04以下均可以使用该方式绕过.而在ubuntu18.04中,已经不再可以.`exec_curl`函数执行会直接返回false.

ubuntu16.04的curl版本是:

```
# curl -V
curl 7.47.0 (x86_64-pc-linux-gnu) libcurl/7.47.0 GnuTLS/3.4.10 zlib/1.2.8 libidn/1.32 librtmp/2.3
Protocols: dict file ftp ftps gopher http https imap imaps ldap ldaps pop3 pop3s rtmp rtsp smb smbs smtp smtps telnet tftp 
Features: AsynchDNS IDN IPv6 Largefile GSS-API Kerberos SPNEGO NTLM NTLM_WB SSL libz TLS-SRP UnixSockets 
```

已经提交了[issue]( https://github.com/idreamsoft/iCMS/issues/88 ),坐等作者的修复,期待是否还有被绕过的可能:D

## 小结

一个开发人员很难有精力去了解一个攻击方式的方方面面,也很难让开发者紧跟攻击手法的趋势.在刚才的例子看到,虽然开发者积极的解决漏洞,但是并不能有效缓解漏洞,总有普通开发者不知道的方式再次绕过.我个人认为,解决这个问题的方法只有让专业的安全专业人员去协助修复.

另外,这个漏洞在利用要进入后台,又过滤了各种敏感协议,实际上危害并不大,仅仅用来学习ssrf的绕过与防护.

*第一次写代码审计相关文章,难免有疏漏,如有错误,还望指正*


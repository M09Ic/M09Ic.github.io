---
title: windows server 应急响应查表
abbrlink: 31524
date: 2019-05-09 10:19:14
categories: 其他
tags: 
- 应急响应
- windows  
---

## 日志审计 
查询安全日志: `Get-WinEvent -FilterHashtable @{LogName='Security'}`   
指定id查询: `Get-WinEvent -LogName Microsoft-Windows-PowerShell/Operational | Where-Object {$_.ID -eq "4100" -or $_.ID -eq "4104"}`  
指定时间查询: 
```
$StartTime=Get-Date  -Year  2017  -Month  1  -Day  1  -Hour  15  -Minute  30
$EndTime=Get-Date  -Year  2017  -Month  2  -Day  15  -Hour  20  -Minute  00

Get-WinEvent -FilterHashtable @{LogName='System';StartTime=$StartTime;EndTime=$EndTime}
```
GUI: `eventvwr.msc`  

<style type="text/css">
	table {width: 50%;}
</style>

重要日志id: 

| event id | description |
| :--------: | :-----------: |
| 4624 | 登陆成功 |
| 4625 | 登陆失败 |
| 4720 | 创建用户 |
| 4726 | 删除用户 |
| 4738 | 用户账户更改 |
| 4698 | 创建计划任务 |
| 4700 | 启用计划任务 |
| 5025 | 关闭防火墙 |
| 5030 | 防火墙无法启动 |
| 7045 | 创建服务 |
| 7030 | 创建服务错误 |
| 18456 | mssql登陆失败 |  



## 检查账户  
打开"本地和用户组": `lusrmgr.msc`  
列出当前所有用户: `net user` , `wmic UserAccount get`  
用命令行无法列出隐藏用户,得在GUI界面看.或注册表: `HKEY_LOCAL_MACHINE\SAM\SAM\Domains\Account`  

删除用户: `net user test$ /del`

## 网络连接
查看连接和监听的端口: `netstat -anob`  
路由: `netstat -rn`  
防火墙: `netsh firewall show all`

查到可疑链接的pid,再查进程.  
可疑链接的ip可疑通过[微步威胁情报](https://x.threatbook.cn/)查询.  
## 进程分析
导出进程参数:  `wmic process get caption,commandline /value >> tmp.txt`  
指定进程名称信息:  `wmic process | findstr "name" >> proc.csv`  
查看系统占用: `Get-Process`    
查看网络连接: `Get-NetTCPConnection`  
查看服务与进程对应关系: `tasklist /svc`    
查看进程与dll关系: `tasklist -M`

工具: `SysinternalsSuite`的`procexp `

## 开机自启

注册表开机自启位置: 
```
HKLM\Software\Microsoft\Windows\CurrentVersion\Runonce
HKLM\Software\Microsoft\Windows\CurrentVersion\policies\Explorer\Run
HKLM\Software\Microsoft\Windows\CurrentVersion\Run
HKCU\Software\Microsoft\Windows NT\CurrentVersion\Windows\Run
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce
(ProfilePath)\Start Menu\Programs\Startup
```

` SysinternalsSuite` 工具集的 `Autoruns `  
GUI: `gpedit.msc`  
## 查看服务

查看服务: `Get-Service`  
GUI: `services.msc`  


## 计划任务 
```
C:\Windows\System32\Tasks\
C:\Windows\SysWOW64\Tasks\
C:\Windows\tasks\
*.job（指文件）
```

命令: `schtasks`  
(若报错运行`chcp 437`)
GUI: `taskschd.msc`


## 其他工具 
火绒剑

参考链接: 
[Windows 系统安全事件应急响应](https://xz.aliyun.com/t/2524)  
[Sysinternals Utilities Index](https://docs.microsoft.com/en-us/sysinternals/downloads/)  
[渗透技巧-Windows系统的帐户隐藏](https://3gstudent.github.io/3gstudent.github.io/%E6%B8%97%E9%80%8F%E6%8A%80%E5%B7%A7-Windows%E7%B3%BB%E7%BB%9F%E7%9A%84%E5%B8%90%E6%88%B7%E9%9A%90%E8%97%8F/)



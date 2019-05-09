---
title: windows server 挖矿木马应急响应
abbrlink: 31524
date: 2019-05-09 10:19:14
categories: 其他
tags: 
- 应急响应
- windows  
---

## 日志审计
windows server 2008 日志位置: `\%SystemRoot%\System32\winevt\Logs\*.evtx`   
查询安全日志: `Get-WinEvent -FilterHashtable @{LogName='Security'}`   
指定id查询: `Get-WinEvent -LogName Microsoft-Windows-PowerShell/Operational | Where-Object {$_.ID -eq "4100" -or $_.ID -eq "4104"}`  
指定时间查询: 
```
$StartTime=Get-Date  -Year  2017  -Month  1  -Day  1  -Hour  15  -Minute  30
$EndTime=Get-Date  -Year  2017  -Month  2  -Day  15  -Hour  20  -Minute  00

Get-WinEvent -FilterHashtable @{LogName='System';StartTime=$StartTime;EndTime=$EndTime}
```


## 检查账户  
打开"本地和用户组": `lusrmgr.msc`  
列出当前所有用户: `net user` `wmic UserAccount get`

## 网络连接
查看连接和监听的端口: `netstat -anob`  
路由: `netstat -rn`  
防火墙: `netsh firewall show all`

## 进程分析
导出进程参数:  `wmic process get caption,commandline /value >> tmp.txt`  
指定进程名称信息:  `wmic process | findstr "name" >> proc.csv`  
查看系统占用: `Get-Process`    
查看网络连接: `Get-NetTCPConnection`  
查看服务与进程对应关系: `tasklist /svc`    
查看进程与dll关系: `tasklist -M`

`SysinternalsSuite`的`procexp `

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

`Get-Service`  
GUI: `services.msc`  
## 计划任务 
```
C:\Windows\System32\Tasks\
C:\Windows\SysWOW64\Tasks\
C:\Windows\tasks\
*.job（指文件）
```

命令: `schtasks`  
GUI: `taskschd.msc`
---
title: SQLMap Cheatsheet
categories: 网络安全
tags:
  - 速查表
  - 学习笔记
abbrlink: 26976
date: 2019-05-24 15:30:54
---



## 0x00 介绍

[sqlmap源码](<https://github.com/sqlmapproject/sqlmap>)

整理自多篇文章,参考链接在文章最后.

## 0x01 功能介绍

基本上支持所有常见的数据,列表如下:

- MySQL
- Oracle
- PostgreSQL
- Microsoft SQL Server
- Microsoft Access
- IBM DB2
- SQLite, Firebird
- Sybase
- SAP MaxDB

主要分为5中不同的注入模式:

- 基于布尔的盲注(B)，即可以根据返回页面判断条件真假的注入。
- 基于时间的盲注(T)，即不能根据页面返回内容判断任何信息，用条件语句查看时间延迟语句是否执行（即页面返回时间是否增加）来判断。
- 基于报错注入(E)，即页面会返回错误信息，或者把注入的语句的结果直接返回在页面中。
- 联合查询注入(U)，可以使用union的情况下的注入。
- 堆查询注入(S)，可以同时执行多条语句的执行时的注入。

## 0x02 基本使用

```shell
# 常用选项
-h					# 显示帮助信息

-hh					# 显示详细帮助信息

-u [url]			# 指定url

-l [filename]		# 使用burpsutie log 文件

-r [filename]		# 从文件中加载HTTP请求

-m [filename]		# 从文件中批量选取url

-x	[sitemap url]	# 从目标网址选择sitemap.xml

-g	[google]		# 使用Google选取目标,国内使用需翻墙

-v [0-6]			# 输出详细程度

--wizard			# 向导模式
# 0:只显示python错误以及严重的信息。
# 1:同时显示基本信息和警告信息。（默认）
# 2:同时显示debug信息。
# 3:同时显示注入的payload。
# 4:同时显示HTTP请求。
# 5:同时显示HTTP响应头。
# 6:同时显示HTTP响应页面。
```



## 0x03 高级使用

#### 基础选项

```shell
-p				# 指定参数

--skip=			# 跳过指定参数

--batch			# 不询问用户输入，使用所有默认配置

--level [1-5]	# 设置探测等级,默认为1,越高检测越详细,耗时也越久

--risk [1-3]	# 设置风险等级,默认为1,越高越容易对目标造成影响

-f				# 检测DBMS指纹

--eta			# 预估完成时间

--flush-session	# 刷新sqlmap session文件

--forms			# 自动获取form表单测试

--dbms=[DBMS]	# 指定数据库类型

--dbms-cred		# 使用dbms身份验证

-os=[OS]		# 指定操作系统

--technique		# 指定SQL注入技术,默认BEUST

--smart			# 启发式判断注入,目标较多时使用这个参数节省时间

--proxy=[PROXY]	# 指定代理

--tor			# 使用tor

--force-ssl		# 强制使用ssl/https

-identify-waf	# 检测WAF|IPS|IDS

--crawl			# 爬行网址url

--output-dir=	# 指定输出路径,默认在 ~/.sqlmap/output 下
```

#### 注入

```shell
# 数据库信息
-a				# 获取所有信息

-b				# 获取banner信息

--hostname		# 获取数据库主机名称

--search [-C|-T|-D name]	# 搜索字段,表,数据库

--sql-query=[QUERY]	# 执行sql语句

--sql-file=[FILENAME] # 执行sql文件

--sql-shell		# 获取交互式的sql-shell

# 获取数据
--current-db	# 获取当前数据库名

-dbs			# 枚举所有数据库

-D				# 指定数据库名

--tables		# 枚举所有表

-T				# 指定表名

--columns		# 枚举所有字段

-C				# 指定字段名

--schema		# 枚举数据库架构

--count			# 查看数据条数

--dump			# dump数据库

# 获取权限
--current-user	# 获取当前数据库账户

--users			# 枚举数据库账户

-U				# 指定使用的数据库账户

--is-dba		# 检查当前账户是否是DBA权限

--roles			# 枚举数据库角色

--privileges	# 枚举数据库账户权限

--passwords		# 枚举数据库账户密码hash
```

#### http包定制

```shell
--user-agent=[data]		# 指定user-agent

--random-agent			# 随机user-agent

--data=[postdata] 		# POST提交数据

--cookie=[cookiedata] 	# 指定cookie

--host=[hostdata]		# 指定host

--referer=[refererdata]	# 指定referer

-H|headers	[HEADER:headerdata]	# 指定额外http头
```

#### getshell

```shell
--os-cmd=[command]		# 指定执行的命令

--os-shell				# 获取一个交互式的shell

--os-pwn				# 反弹msf下的shell或者vnc

--os-smbrelay			#  一键获取一个OOB shell，meterpreter或VNC

--os-bof 				# 存储过程缓冲区溢出利用

--priv-esc 				# 数据库进程用户权限提升

--msf-path=[MSFPATH]  	# MetasploitFramework本地的安装路径

--tmp-path=[TMPPATH]  	# 远程临时文件目录的绝对路径

--udf-inject    		# 注入用户自定义函数

--shared-lib=[SHLIB]   	# 共享库的本地路径
```

#### 文件操作

```shell
--file-read=[filename]		# 读文件

--file-write=[filename]		# 写文件

--file-dest=[AbsolutePath]	# 绝对路径
```

#### Windows 注册表操作

```shell
--reg-read          	# 读一个Windows注册表项值

--reg-add           	# 写一个Windows注册表项值数据

--reg-del				# 删除Windows注册表键值

--reg-key=[REGKEY]    	# Windows注册表键

--reg-value=[REGVAL]  	# Windows注册表项值

--reg-data=[REGDATA]  	# Windows注册表键值数据

--reg-type=[REGTYPE]  	# Windows注册表项值类型
```

#### 优化性能

```shell
-o              	# 打开所有的优化开关

-predict-output    	# 预测普通查询输出

--keep-alive        # 使用持久HTTP（S）连接

--null-connection   # 获取页面长度

--threads=[THREAD number]  # 当前http(s)最大请求数 (默认 1)

--timeout			# 设置超时时间,默认30s

--retries			# 设置重连次数,默认为3次

```

#### 绕过

```shell
--hex				# 使用hex函数

--hpp				# 使用http参数污染   

--moble				# 模拟智能手机

--second-order		# 二阶注入

--safe-url=[SAFEURL]  # 提供一个安全的连接，每隔一段时间都会去访问一下

--skip-urlencode  	# 跳过URL编码

--no-cast          	# 关闭有效载荷铸造机制

--no-escape         # 关闭字符串逃逸机制

--prefix=PREFIX     # 注入payload字符串前缀

--suffix=SUFFIX     # 注入payload字符串后缀

--invalid-bignum    # 使用大数字使值无效

--invalid-logical   # 使用逻辑操作使值无效

--invalid-string    # 使用随机字符串使值无效

--string=[STRING]   	# 查询时有效时在页面匹配字符串

--not-string=[NOTSTRING]# 当查询求值为无效时匹配的字符串

--regexp=[REGEXP]     	# 查询时有效时在页面匹配正则表达式

--code=[CODE]       	# 当查询求值为True时匹配的HTTP代码

--text-only        		# 仅基于在文本内容比较网页

--titles           		# 仅根据他们的标题进行比较
```

## 0x04  进阶使用

#### tamper列表

```shell
--tamper [tampername]	 	# 使用脚本绕过waf,逗号分隔多个

space2comment				# 用/**/代替空格

apostrophemask				# 用utf8代替引号

equaltolike					# like代替等号

space2dash					# 绕过过滤‘=’ 替换空格字符（”），（’–‘）后跟一个破折号注释，一个随机字符串和一个新行（’n’）

greatest					# 绕过过滤’>’ ,用GREATEST替换大于号。

space2hash					# 空格替换为#号,随机字符串以及换行符

apostrophenull				# encode绕过过滤双引号，替换字符和双引号。

halfversionedmorekeywords	# 当数据库为mysql时绕过防火墙，每个关键字之前添加mysql版本评论

space2morehash				# 空格替换为 #号 以及更多随机字符串 换行符

appendnullbyte				# 在有效负荷结束位置加载零字节字符编码

ifnull2ifisnull				# 绕过对IFNULL过滤,替换类似’IFNULL(A,B)’为’IF(ISNULL(A), B, A)’

space2mssqlblank			# (mssql)空格替换为其它空符号

base64encode				# 用base64编码替换

space2mssqlhash				# 替换空格

modsecurityversioned		# 过滤空格，包含完整的查询版本注释

space2mysqlblank			# 空格替换其它空白符号(mysql)

between						# 用between替换大于号（>）

space2mysqldash				# 替换空格字符（”）（’ – ‘）后跟一个破折号注释一个新行（’ n’）

multiplespaces				# 围绕SQL关键字添加多个空格

space2plus					# 用+替换空格

bluecoat					# 代替空格字符后与一个有效的随机空白字符的SQL语句,然后替换=为like

nonrecursivereplacement		# 双重查询语句,取代SQL关键字

space2randomblank			# 代替空格字符（“”）从一个随机的空白字符可选字符的有效集

sp_password					# 追加sp_password’从DBMS日志的自动模糊处理的有效载荷的末尾

chardoubleencode			# 双url编码(不处理以编码的)

unionalltounion				# 替换UNION ALLSELECT UNION SELECT

charencode					# url编码

randomcase					# 随机大小写

unmagicquotes				# 宽字符绕过 GPCaddslashes

randomcomments				# 用/**/分割sql关键字

charunicodeencode			# 字符串 unicode 编码

securesphere				# 追加特制的字符串

versionedmorekeywords		# 注释绕过

space2comment				# 替换空格字符串(‘‘) 使用注释‘/**/’

halfversionedmorekeywords	# 关键字前加注释
```

## 参考链接

[sqlmap用户手册](<http://www.anquan.us/static/drops/tips-143.html>)

[sqlmap用户手册续](<http://www.anquan.us/static/drops/tips-401.html>)

[超详细SQLMap使用攻略及技巧分享](<https://www.freebuf.com/sectool/164608.html>)

[sqlmap的使用 ---- 自带绕过脚本tamper](<https://xz.aliyun.com/t/2746>)


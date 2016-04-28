> CentOS 7.x

## df
	
	df
	//扫描磁盘使用情况
		
---

## firewall-cmd

	firewall-cmd --zone=public --add-port=22/tcp --permanent
	firewall-cmd --reload
	//打开端口

# 文件处理

## ls

	ls /xx/yy
	//指定列出 /xx/yy 目录下的文件和目录，不指定则默认指定当前目录
---

	ls -a 
	//all 所有文件（包括隐藏）
---

	ls -lh
	/*
	-l long list 长列表（包含详细信息），信息依次为
	类型权限 |　硬连接引用次数 | 所属者 | 所属组 | 大小 | 创建日期 | 名称
	*/
	//-h human 人性化显示（例如大小单位）
---

	ls -id
	//-d 展示目录本身，而不展示其下...
	//-i 查看 i 节点（文件/目录 的 id）
	
## mkdir

	mkdir -p
	// -p 递归创建目录

## pwd

	pwd
	// 显示当前目录

## rmdir

	rmdir
	// 删除空目录
---

## cp

	cp file1 file2 file3 /to/dir
	// 拷贝多个文件

	cp -rp /from/dir /to/dir
	// -r 拷贝目录
	// -p 保留文件/目录属性（例如创建时间）
	
## mv

	mv /source/file/or/dir /to/dir
	//剪切 或 更名

## rm

	rm file
	// 删除文件
---

	rm -r /dir
	// -r 删除目录
---

	rm -rf /not/empty/dir
	// -f 强制删	
	
## touch

	touch newfile.txt
	// 创建文件
---

	touch "programe files"
	// 特殊命名要带引号，不建议

## cat

	cat -n /to/watch/this/file
	// 查看文件内容
	// -n 带行号

## more

	more /to/watch
	// 查看文件， 空格-翻页， 回车-下一行，q-退出

## less

	//查看文件， 可以往回翻
	//可以搜索关键词，输入斜杠搜索关键字
	
## head
	
	head -n 3 file.txt
	// 查看文件前三行

## tail
	
	tail -n 3 file.txt
	// 查看文件后三行
	
	tail -f file.txt
	// -f 动态显示文件末尾

## ln

	ln -s /src/file  //hard.link
	// 生成硬链接
---

	ln -s /src/file  /soft.link
	// 生成软链接
---

## mount
> 挂载

	mount -t iso9660 /dev/src0 /mnt/mount
	
	// 设备/dev/src0 挂到 /mnt/mount
	// -t 指定文件系统
---

# 权限管理

## chmod

> change ... mode of permission 更改权限

	chmod u+r /target/file
	
	// u-表示所有者，还可以是 g-所属组, o-其他
	// '+' 表示添加权限，还可以是 '-' 或 '='
	// r 是 rwx 中的一个
	/*
		权限		对文件				对目录
		------------------------------
	r	读		查看文件内容		列出目录中的内容
	w	写		修改文件内容		目录中创建、删除文件
	x	执行		执行文件			可以进入目录
	
	*/
---

	chmode -R 777 /target/dir
	
	// r:4, w:2, x:1, rwxrw-r--:764
	// -R 表示递归，不加则默认只会改变目录本身权限

## chown

> change file onwership 改变文件所有者， root 用户才可以

	change newowner /target
	// 改变 /target 的所有者为 newowner


## chgrp

> change file group ... 改变文件的所有组，文件所有者才可以

## umask

> the user file-creation mask 新建文件的缺省权限
> 
> ps: 无论怎么设置，新建文件是没有 x 权限的，目录可以有

	umask -S
	
	//用 rwx 形式显示...
---

	umask 023
	
	// 设置为 rwxr-xr--
	/*
	需求：设置为 rwxr-xr--
	1）换算为 754
	2）777 - 754 = 023
	3）输入 023（掩码）
	*/

# 文件搜索

## find
> 扫描硬盘，插在文件

	find /search/from/dir -name ?key*
	
	// -name 表示按名称搜索
	// -iname 不区分大小写
	// ? 表示单个字符， * 表示任意个 
---

	find /dir -size 204800
	
	// -size 按大小搜索，单位是数据块 (512字节)
	// -size +10 表示大于 10个数据块
	// -size -10 表示小于 10哥数据块
---

	find /dir -user username
	
	// -user 根据所有者
	// -group 根据所属组
---


	find /dir -cnmin -5
	
	// 搜索5分钟内文件属性属性被修改过的
	// -amin 访问时间(access)
	// -cmin 文件属性(change)
	// -mmin 文件内容(modify)	
	// +5 表示超过5分钟
---

	find /dir -size +10 -a -size -100
	
	// -a 表示条件并
	// -o 表示条件或
---

	find /dir -type f
	
	// -type 表示根据文件类型查找
	// f:文件  d:目录  l:软链接
---

	find -inum 32112
	
	// 根据文件 i 节点查找
	// hack: 可以找硬链接
---

	find ./ -name notebook -exec ls -l {} \;
	
	// 对查询结果执行命令


## locate
> 在文件资料库中查找文件，不是扫描硬盘，资料库不包含某些目录（/tmp）

	updatedb
	// 更新文件资料库
---

## which
> 搜索命令所在的目录和别名信息

## whereis
> 搜索命令所在的目录和帮助文档路径

## grep
> 关键词逐行搜索文件内容

	grep -i keyword file.txt
	
	// -i 不区分大小写
	// -v 『排除关键词』搜索

# 帮助

## man
> 查看命令的帮助信息，查看配置文件的帮助信息

	man ls
	
	// 查看 ls 命令的帮助文档
	// 空格:往下翻页，b:往上翻页
	/*
	 ➜  ~ whereis ls
	ls: /usr/bin/ls /usr/share/man/man1/ls.1.gz
	形如 /man/man1/xx.1.gz 这类带 『1』 的帮助文档都是命令相关
	带 『5』 的则是配置文件的帮助文档
	man 优先显示命令的帮助
	*/
---

	man services
	
	// 查看 /etc/services 配置文件的帮助信息
	// ps: man 后面的配置文件不需要带绝对路径
	// 如果有同名的命令和配置文件，那么需要
	// man 5 password 指定显示配置文件的帮助信息
---
	
## whatis
> 查询类似命令的简单描述

	whatis ls
	
	// 显示所有带 ls 的命令的简要说明

## apropos
> 查询类似配置文件的简单描述

## info
> 跟 man 差不多，内容格式排版不一样

## help
> 查看内置命令（找不到路径的命令）的帮助

	help cd
	// cd 是内置命令
	// which cd 
	// cd: shell built-in command


# 用户管理

## useradd
> 添加用户

	useradd peter
---

## passwd
>  为用户设置密码

	passwd peter
	
	// 用 peter 设置密码

## who
> 查看当前登陆的用户们

	➜  ~ who
	root     pts/0        2016-04-28 04:23 (183.237.64.198)
	peter    pts/1        2016-04-28 05:29 (183.237.64.198)
	// pts：远程终端登录   tty:本地终端登

## w
> 查看谁登录以及 uptime 信息

## uptime
> 系统运行摘要

# 压缩

## gzip
> GUN zip, 压缩为 .gz，

	gzip srcfile
	
	// 只能压缩文件（windows压啥都行）
	// 不保留源文件

## gunzip
> 解压缩

## tar
> 打包压缩 / 解包解压

	tar -cvf result.tar /src/sth
	
	// -c 打包, 可以是目录（通常后缀为 .tar）
	// -v 显示详情
	// -f 指定结果名
	// -z 打包同时使用zip压缩（通常后缀为 .tar.gz）
	// -j 打包同时使用bzip压缩（通常后缀为 .tar.bz2）
---

	tar -xvfz result.tar.gz
	
	// -x 解包
	// -f 指定文件
	// -z gzip 解压
	// -j bz2 解压
	// -v 显示详情
---

## zip
> 压缩为 .zip, 文件/目录，保留原文件

	zip -r result.zip /src/dir
	
	// -r 目录
---

## unzip
> 解压缩

## bzip2
> gzip压缩升级版， 后缀通常为 .bz2

	bzip2 -k /src/any
	
	// -k 保留原文件
---

# 网络

## write
> 古老的向用户发信息

	write username
	> blablabla
	> ctrl+d 保存发送
---

## wall
> 广播
	
## ping
> 测试网络连通性
	
	ping -c 10 192.168.1.1
	
	// -c 发送次数
---

## ifconfig
> interface configure

    ifconfig
    // 查看网卡情况（ip）
    
---

    ifconfig et0 192.x.x.x
    //设置设备临时ip（重启后无效）

## last
> 列出目前和过去每次登录的信息

## lastlog
> 列出所有用户最后一次登录

	lastlog -u nameorid
	
	// -u 指定查看某个用户

## traceroute
> 探测到达主机间的路径
	
## netstat
> 显示网络相关信息

	// -t TCP协议
	// -u UDP协议
	// -l 监听
	// -r 路由
	// -n 显示ip和端口
---

	netstat -tlun
	// 查看本机监听的端口
---

	netstat -an
	// 本机所有网络连接
---
	
	netstat -rn
	// 本机路由表

## setup
> redhat 系提供的图形配置网络

	
	
	
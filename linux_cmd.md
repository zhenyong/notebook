> CentOS 7.x

## ifconfig
    
    ifconfig
    //查看网卡情况（ip）
    
---

    ifconfig et0 192.x.x.x
    //设置设备临时ip（重启后无效）

## df
	
	df
	//扫描磁盘使用情况
		
---

## firewall-cmd

	firewall-cmd --zone=public --add-port=22/tcp --permanent
	firewall-cmd --reload
	//打开端口



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
	











	

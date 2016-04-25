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


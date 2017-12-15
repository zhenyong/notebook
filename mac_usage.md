## mac 快捷

- 切换

	* cmd + tab
		* tab 向右切换
		* ～ 向左切换
		* 松开前，摁住 option 可以把最小化的窗口打开
	* cmd + shift + tab 从右向左切换
	* cmd + ~ 同个程序多个窗口间切换
	* 四指上滑（control + up），切换程序
	* 四指下滑 (control + down)，切换某程序多窗口

- 窗口

	* cmd + H 隐藏 app，四指切换程序时看不到
	* cmd + alt + H 只显示当前应用窗口，其他隐藏
	* cmd + M 最小化窗口，四指切换程序可见
	* cmd + W 关闭当前窗口
	* cmd + Q 退出当前程序

- 截图

	* cmd + shift +3 全屏
	* cmd + shift + 4 选定
		* space 进入窗口模式
		* 选择范围松开鼠标前，shift（切换） 锁定 X/Y 轴
		* 选择范围松开鼠标前，option 锁定圆心

	截图时摁住 control 则进入剪贴板

- 屏幕放大
	
	* 摁住 control + 双指触屏滑动

- emoji
	
	* 默认 ⌘ ⌃ + space 打开特殊符号选择

- finder
	
	+ ⌘⇧. 切换显示隐藏文件
	
## bash 文件

file  | means
------------- | -------------
/etc/profile | 此文件为系统的每个用户设置环境信息,当用户第一次登录时,该文件被执行.并从/etc/profile.d目录的配置文件中搜集shell的设置.
/etc/bashrc | 为每一个运行bash shell的用户执行此文件.当bash shell被打开时,该文件被读取.
~/.bash_profile | 每个用户都可使用该文件输入专用于自己使用的shell信息,当用户登录时,该文件仅仅执行一次!默认情况下,他设置一些环境变量,执行用户的.bashrc文件.
~/.bashrc | 该文件包含专用于你的bash shell的bash信息,当登录时以及每次打开新的shell时,该该文件被读取.
~/.bash_logout | 当每次退出系统(退出bash shell)时,执行该文件.

## 更换 shell 为 zsh

https://github.com/robbyrussell/oh-my-zsh

## 命令

### echo

- 在终端打印内容，如打印当前环境变量

	```
	echo $PATH
	```

- 把某些内容打印到某个文件，例如修改某个 bash 配置文件

	```
	echo "export PATH=xxxxxx:$PATH" >> ~/.bash_profile'
	```

## 剪贴板

- 复制内容到剪贴板

	```
	pbcopy < in.txt
	```

- 输出剪贴板内容

	```
	pbpaste >> out.txt
	```

## 我要

### ssh 设置登录别名

~/.ssh/config 文件写入

	Host theName
   		HostName xx.xx.xx.xx
   		Port 22
   		User root

之后就可以 ssh theName 快速登录了

### 设置默认打开方式

- 方式一：选中文件 -> control + 单点 -> 摁住 alt -> 选择始终以此方式打开（我的mac居然无效）
- 方式二：选中文件 -> cmd + i -> 更改打开方式 -> 点击全部更改（靠谱）

### 在命令行用 sublime 打开文件

1. 对 sublime 的启动脚本建立一个软连接，放在可执行目录下

	```
	ln -f -s /Applications/Sublime\ Text.app/Contents/SharedSupport/bin/subl /usr/local/bin/
	```

2. 当命令行需要文本窗口输入时，默认使用 sublime

	```
	echo "EDITOR='subl -w'" >> ~/.bash_profile
	```

btw：查看 sublime 参数 用 subl -help 命令

### 给自带词典添加离线字典

[导入工具](https://github.com/jjgod/mac-dictionary-kit)
[字典列表](http://abloz.com/huzheng/stardict-dic/zh_CN/)

在字典列表下载离线字典，然后打开导入工具，把字典文件拖入，
然后在词典软件的 preference (偏好设置)把新增的字典勾选。

### 屏幕内容不小心放大，溢出屏幕

ctrl + 双指下滑


### 强制关闭

- 方式一：Cmd + Alt + Esc 关闭
- 方式二：打开 Activity Monitor 选中，点击关闭按钮
- 方式三：
	+ 命令行 top，找到 pid
	+ q 退出
	+ kill <pid>
	
ctrl + 双指下滑三
ctrl + 双指下滑

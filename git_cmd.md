## git diff

	git diff HEAD
	// 工作目录与最近一次提交的区别, HEAD 是最近一次提交id的别名
---

	git diff -stat
	// 更改过的文件列表和 diff 总览
---
	
	
	
	

## git rm

	git rm --cache
	// 移除对某个文件的管理，但是想把它留在文件夹里
---

## git mv
> 移动


## git log

	git log --oneline
	// 提交摘要（id+描述）
---

	git log --stat
	// 包括涉及的文件和改动概要
---

	git log --patch
	// 包括详细改动
---

	git log -- /see/this/file/only.txt
	// 查看某个路径相关的日志
---

	git log -M --follow -- /a/path
	// 文件如果移动后，要追踪显示移动之前的日志
---
	
	
	
## git add

	git add -A .
	// 当你在电脑上移动一堆文件，避免手动 rm&add, 批量更新 rename 状态
	
	//note: 如果修改了再移动，有坑
	1)修改
	2)手动移动文件目录
	3)执行 `git add -A .`
	4）`git status`只显示 renamed，不会提示 modified
---

	
	

## 额~

- 你在文件夹窗口删除了一堆文件，避免手动一个个 `git rm`
	
		git add -u .
		这个命令是逐层扫描当前目录下可以添加的文件，然后添加，在当前场景下，就相当于寻找所有以及删除的文件
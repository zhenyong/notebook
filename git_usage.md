## Story

## 修改前几次的提交信息

```
git rebase -i 父提交
```

使用交互式的 rebase 操作，背后就是搞出 detached-Head （理解成 checkout 出某个提交，此时没有分支），然后修改完，refs/heads/master 再指向它，
这就是当我们要修改某个提交的 message 时，至少需要 rebase 它的父提交

## 查看 暂存区 跟 HEAD 的差异

```
git diff --cached
```

## 工作区文件恢复跟暂存区一致

```
git checkout -- [file]
```

## 暂存区文件退出暂存

```
git reset HEAD -- [file]
```

## 把最近一堆提交干掉

```
git reset --hard [commit]

git reset --hard HEAD // 暂存区&工作区都恢复跟头指针一样
```

## 忽然接到紧急任务，手上有改动但是没完成代码怎么办

```
git stash 当前改动入缓存栈
git stash apply 只是使用栈顶缓存的改动，但是不出栈
git stash pop 弹出改动
git stash list 查看 stash 栈
```

## 备份本地仓库

```
git clone --bare file:///本地仓库路径/.git  备份目标路径.git
```

这样创建的裸仓库没有工作区（就是没有源文件），之后把这个 备份目标路径.git 作为一个远程仓库，push 到这个备份.git

```
git remote add backup 备份目标路径.git
```


## Command

### git diff

	git diff HEAD
	// 工作目录与最近一次提交的区别, HEAD 是最近一次提交id的别名
---

	git diff -stat
	// 更改过的文件列表和 diff 总览
---
	

### git rm

	git rm --cache
	// 移除对某个文件的管理，但是想把它留在文件夹里
---

### git mv
> 移动


### git log

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
	
	
	
### git add

	git add -A .
	// 当你在电脑上移动一堆文件，避免手动 rm&add, 批量更新 rename 状态
	
	//note: 如果修改了再移动，有坑
	1)修改
	2)手动移动文件目录
	3)执行 `git add -A .`
	4）`git status`只显示 renamed，不会提示 modified
---

	
	

### 额~

- 你在文件夹窗口删除了一堆文件，避免手动一个个 `git rm`
	
		git add -u .
		这个命令是逐层扫描当前目录下可以添加的文件，然后添加，在当前场景下，就相当于寻找所有以及删除的文件
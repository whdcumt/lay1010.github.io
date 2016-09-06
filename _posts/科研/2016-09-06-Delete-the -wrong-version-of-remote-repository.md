---
layout: post
title: 删除远程仓库中的错误版本
category: 技术
tags: Github Git 
keywords: Github，Git
description: 删除远程仓库中的错误版本
---

##遇到问题：
如果提交到Github上的数据存在错误，那么我们如何将错误的版本删除掉呢？
##解决方法：
假如你有3个commit如下：<br>
```
    commit 3
    commit 2
    commit 1
```
如果最后一次提交**commit3**提交的版本有错误， 那么执行：
```
    git reset --hard HEAD~1
```
现在，**HEAD is now at commit 2**
然后使用
```
    git push --force
```
将本次变更强行推送到远程服务器。**值得注意**的是这类的操作比较危险，比如：你的**commit3**之后别人又提交了新的**commit4**,那么这位仁兄的commit4也一并消失了。


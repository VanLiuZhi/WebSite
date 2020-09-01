---
weight: 1
title: "git 版本控制工具"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "git 版本控制工具"
resources:
- name: "base-image"
  src: "base-image.jpg"

tags: [Web, Note, Git]
categories: [Web开发]

lightgallery: true

toc:
  auto: false
---

Git 是 Linus Torvalds 为了帮助管理 Linux 内核开发而开发的一个开放源码的版本控制软件。是一个开源的分布式版本控制系统，可以有效、高速地处理从很小到非常大的项目版本管理。

<!-- more -->

# Git

使用GitHub，给出的地址我们一般用ssh。使用ssh需要https，如果不支持只能使用http，但是每次都要输口令。

## 用法总结

在idea工具盛行的时代，使用idea工具能高效快速的使用Git。如果你纯命令，那么解决冲突和合并分支都是要用类似vim的工具，其效率可想而知

推荐用idea

另外，大部分情况我们操作本地仓库就足够了，本地仓库也创建分支，打标签，然后开发到一定程度也可以合并，这些都是本地仓库操作的。真正要提交到远程仓库的，一定是开发完成的了

## 术语

工作区
暂存区
本地仓库
远程仓库

## 标签 tag

Git的 tag 和 branch 是比较容易混淆的概念

标签分为轻量标签和附注标签，简单说就是附注标签会加上一些描述信息，轻量就没有了。然后标签是指向某个提交对象，也就是说你可以把本次提交打个标签，这样可以很方便的回到这个提交

常见用法: 选一个commit对象，把它打上标签，一个commit对象可以打多个

## 分支

Git鼓励大量使用分支：

- 查看分支：git branch
- 创建分支：git branch \<name>
- 切换分支：git checkout \<name>
- 创建+切换分支：git checkout -b \<name>
- 合并某分支到当前分支：git merge \<name>
- 删除分支：git branch -d \<name>

主要理解分支，克隆远程仓库，将本地和远程仓库关联，搭建git服务器

## git pull 命令

作用：取回远程主机某个分支的更新，再与本地的指定分支合并

格式：git pull  <远程主机名> <远程分支名>:<本地分支名>

1. 如果与当前分支合并，则可省略本地分支名git pull <远程主机名> <远程分支名> 相当于：git fetch <远程主机名> <远分支名> git merge <远程主机名>/<远程分支名>
2. 如果当前分支与远程分支存在追踪关系 git pull <远程主机名>
3. 如果当前分支只有一个追踪关系 git pull
4. 手动建立追踪关系 git branch --set-upstream master origin/next
5. 清理远程已删除本地还存在的分支 git fetch --prune origin 或者 git fetch -p 或者 git pull -p


## 如何上传GitHub

1. 在用户目录下 .ssh
2. ssh-keygen -t rsa -C "1441765847@qq.com" 
3. 把 id_rsa.pub  添加到GitHub的ssh上
4. git init  把当前目录变为仓库
5. git add  把文件添加进仓库  git commit 把文件提交到仓库  
6. git add --all 当我们在一个不是空目录下init需要把所有文件添加到仓库的时候使用

## 文件的标记解释：

```s
A: 你本地新增的文件（服务器上没有）.

C: 文件的一个新拷贝.

D: 你本地删除的文件（服务器上还在）.

M: 文件的内容或者mode被修改了.

R: 文件名被修改了。

T: 文件的类型被修改了。

U: 文件没有被合并(你需要完成合并才能进行提交)。

X: 未知状态(很可能是遇到git的bug了，你可以向git提交bug report)。
```

```s
git pull

git pull origin master

git pull origin master
 --allow-unrelated-histories
```

## git的hook(钩子)

为了防止一些不规范的代码 `commit` 并 `push` 到远端，我们可以在  `git` 命令执行前用一些钩子来检测并阻止。
在node中，安装需要的模块：`husky`, `pre-commit` 配置package.json在提交代码前执行自定义的脚本。

```sh
cd .git/hooks
ls -l
```

该目录提供了git的各个钩子的脚步案例。


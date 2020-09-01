---
weight: 1
title: "linux-shell 环境变量"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "linux-shell 环境变量"
resources:
- name: "base-image"
  src: "base-image.jpg"

tags: [Linux, Note]
categories: [操作系统]

lightgallery: true

toc:
  auto: false
---

linux shell学习笔记

<!-- more -->

## 定位系统环境变量

登录shell的时候，默认情况下bash会在几个文件中查找命令，这些文件称为启动文件或环境文件。
这就是我们经常设置的把某个程序的目录加到环境变量，如果你跟风，用了什么 `item2` 这样的第三方shell，并对系统做了一些修改，那么原来安装的软件默认设置的环境变量就没有了，需要把他们迁移到新的shell中。

    bash检查的启动文件，取决于你启动shell的方式：
    - 登录时作为默认登陆shell
    - 作为非登录shell的交互式shell
    - 作为运行脚本的非交互shell
    虽然都是进入了shell，但是它们的环境变量有区别。

## 登录shell
这种shell就是你登陆后启动的shell。

常见启动文件：
- /etc/profile
- $HOME/.bash_profile&nbsp;&nbsp; $HOME/.bashrc
- $HOME/.bash_login&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; $HOME/.profile

上述是 `bash shell` 的启动文件。如果你使用了一些第三方安装了 `zsh` 应该可以在~目录找到 `.zshrc`。<br/>
`/etc/profile` 是系统环境变量，剩余的是用户的，每个用户都可以编辑这些文件添加自己的环境变量。
这些环境变量在启动 `bash shell` 的时候生效。

shell会按照按照下列顺序，运行第一个被找到的文件，余下的则被忽略:
- $HOME/.bash_profile  
- $HOME/.bash_login
- $HOME/.profile

注意，这个列表中并没有 `$HOME/.bashrc文件`。这是因为该文件通常通过其他文件运行的，比如在有些Linux系统中 `./bash_profile` 文件会去找有没有 `.bashrc`，有的话先执行它。

## 交互式shell
就是通过命令行启动shell，比如 `/bin/sh 如果安装zsh /bin/zsh` 这种情况不会访问系统变量。

## 非交互式shell
最后一种shell是非交互式shell。系统执行shell脚本时用的就是这种shell。不同的地方在于它
没有命令行提示符。但是当你在系统上运行脚本时，也许希望能够运行一些特定启动的命令。**非交互，理解继承，如果你要编写脚本要知道这是什么情况**

为了处理这种情况，`bash shell` 提供了 `BASH_ENV` 环境变量。当shell启动一个非交互式shell进程时，它会检查这个环境变量来查看要执行的启动文件。如果有指定的文件，shell会执行该文件里的命令，这通常包括shell脚本变量设置。

在CentOS Linux发行版中，这个环境变量在默认情况下并未设置。如果变量未设置，printenv命令只会返回CLI提示符:
$ printenv BASH_ENV $

Ubuntu发行版中，变量BASH_ENV也没有被设置。记住，如果变量未设置，echo 命令会显示一个空行，然后返回CLI提示符:
$ echo $BASH_ENV

那如果BASH_ENV变量没有设置，shell脚本到哪里去获得它们的环境变量呢?别忘了有些shell脚本是通过启动一个子shell来执行的。子shell可以继承父shell导出过的变量。举例来说，如果父shell是登录shell，在/etc/profile、/etc/profile.d/*.sh和$HOME/.bashrc文件中
设置并导出了变量，用于执行脚本的子shell就能够继承这些变量。 

    要记住，由父shell设置但并未导出的变量都是局部变量。子shell无法继承局部变量。
    对于那些不启动子shell的脚本，变量已经存在于当前shell中了。
    所以就算没有设置BASH_ENV，也可以使用当前shell的局部变量和全局变量。

## 环境变量持久化

现在你已经了解了各种shell进程以及对应的环境文件，找出永久性环境变量就容易多了。也可以利用这些文件创建自己的永久性全局变量或局部变量。
对全局环境变量来说(Linux系统中所有用户都需要使用的变量)，可能更倾向于将新的或修改过的变量设置放在 `/etc/profile` 文件中，但这可不是什么好主意。如果你升级了所用的发行版，这个文件也会跟着更新，那你所有定制过的变量设置可就都没有了。

最好是在 `/etc/profile.d` 目录中创建一个以.sh结尾的文件。把所有新的或修改过的全局环境变 量设置放在这个文件中。
在大多数发行版中，存储个人用户永久性 `bash shell` 变量的地方是 `$HOME/.bashrc` 文件。这一点适用于所有类型的shell进程。但如果设置了BASH_ENV变量，那么记住，除非它指向的是 $HOME/.bashrc，否则你应该将非交互式shell的用户变量放在别的地方。
图形化界面组成部分(如GUI客户端)的环境变量可能需要在另外一些配置文件中设置，这和设置 `bash shell` 环境变量的地方不一样。
你可以把自己的alias设置放在 `$HOME/.bashrc` 启动文件中，使其效果永久化。

:sunny:**总结：**<br/>
全局环境变量可以在对其作出定义的父进程所创建的子进程中使用。局部环境变量只能在定义它们的进程中使用。
Linux系统使用全局环境变量和局部环境变量存储系统环境信息。可以通过shell的命令行界面或者在shell脚本中访问这些信息。`bash shell` 沿用了最初 `Unix Bourne shell` 定义的那些系统环境变量，也支持很多新的环境变量。PATH环境变量定义了 `bash shell` 在查找可执行命令时的搜索目录。可以修改PATH环境变量来添加自己的搜索目录(甚至是当前目录符号)，以方便程序的运行。也可以创建自用的全局和局部环境变量。一旦创建了环境变量，它在整个shell会话过程中就都是可用的。

`bash shell` 会在启动时执行几个启动文件。这些启动文件包含了环境变量的定义，可用于为每个bash会话设置标准环境变量。每次登录Linux系统，`bash shell`  都会访问/etc/profile启动文件以及3个针对每个用户的本地启动文件 **:$HOME/.bash_profile、$HOME/.bash_login和$HOME/.profile。** 用户可以在这些文件中定制自己想要的环境变量和启动脚本。
最后，我们还讨论了环境变量数组。这些环境变量可在单个变量中包含多个值。你可以通过指定索引值来访问其中的单个值，或是通过环境变量数组名来引用所有的值。

    重点：永久环境变量，用户环境变量，配置顺序也很重要，直接使用符号链接和配置环境变量

## ssh免密登录

ssh-keygen 在本地生成密钥和私钥，有时候会看到带 -t DSA 参数，这是选择加密算法，一般用默认就行。
把公钥复制到目标机器的authorized_keys中即可


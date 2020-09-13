---
weight: 1000
title: "Linux 开发环境 shell 配置"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Linux 开发环境 shell 配置"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Linux, Note]
categories: [操作系统]

lightgallery: true

toc:
  auto: false
---

shell 配置和使用 zsh

<!-- more -->

## Xshell

在使用Xshell的时候，出现本地终端排版错误问题，解决方案：文件 - 属性 - 终端 - 高级 - 用CR-LF接受LF(R)

## 安装 zsh

通过命令就可以安装了

CentOS 安装：sudo yum install -y zsh
Ubuntu 安装：sudo apt-get install -y zsh
在检查下系统的 shell：cat /etc/shells，你会发现多了一个：/bin/zsh

安装完成zsh后，再安装oh-my-zsh会自动切换过去，echo $SHELL 可以查看当前shell，如果shell已经切换了，再执行切换命令是不行的

## 安装 oh-my-zsh

oh-my-zsh 是 zsh 扩展的集合，有了扩展集合包，通过配置zsh文件就能很方便的开启功能
当然有时候不管用，可能因为扩展包版本的问题，或是相关插件没在扩展包里面，这个时候手动下载插件即可

`sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"`

{% blockquote %}
最好还是使用官方的安装，遇到用别人的，安装后插件无法识别，我也是奇怪了就是用GitHub的脚本，差距有点大
{% endblockquote %}

常用插件：

插件安装，有默认自带的(.oh-my-zsh/plugins)，和通过安装的(.oh-my-zsh/custom/plugins)，两个路径会不一样，在配置文件 .zshrc 中有描述，记得看看

如果是自带就有的，通过修改plugins=(git docker) 这样的形势就可以增加插件，修改后执行 source .zshrc让插件生效，自带没有的（就是自带目录找不到，可以去下载它，然后放到安装目录，这样方便管理自带和下载的，同样修改plugins，然后执行生效），下面给出了常用的两个通过下载安装的插件

1. zsh-syntax-highlighting 语法高亮，能提示错误的语法

在配置插件了加入 zsh-syntax-highlighting，如果不管用，可能就是扩展包没有这个插件，通过手动下载即可。

```
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git 
echo "source ${(q-)PWD}/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh" >> ${ZDOTDIR:-$HOME}/.zshrc
```

or

git clone git://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting.git

生效 `source ~/.zshrc`

就是下载插件，并加入当前用户的环境变量中

2. 自动补全插件

- zsh-autosuggestions
`git clone git://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions`
or
`git clone git://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions`
`plugins=(git zsh-autosuggestions)`

- incr
```
下载 http://mimosa-pudica.net/src/incr-0.2.zsh
复制插件到扩展包插件目录 .oh-my-zsh/plugins/创建incr文件夹放到该文件夹中
当前用户环境变量添加 source ~/.oh-my-zsh/plugins/incr/incr*.zsh
```

上面的命令一气呵成，可能不同的环境，移动文件会不对，提前检查一下

## 配置vim

下载好vim后，通常还需要进行配置，只是想行号显示这种功能我认为是必须的，不然系统报错了告诉你行号你都不知道是哪行。

另外有些系统带了vim，是vi命令启动的，应该是阉割版本，所以还是再重新装一个。

vim配置文件分为系统和用户的，系统修改就是全局的了，所以好的习肯定是修改自己用户的，如果你不知道配置文件在什么地方，打开vim，输入`:version` 即可查看相关信息。

    cd ~ 
    touch .vimrc

Mac 上的配置文件，比较简单的配置

```
" Configuration file for vim
set modelines=0		" CVE-2007-2438

" Normally we use vim-extensions. If you want true vi-compatibility
" remove change the following statements
set nocompatible	" Use Vim defaults instead of 100% vi compatibility
set backspace=2		" more powerful backspacing

" Don't write backup file if vim is being called by "crontab -e"
au BufWrite /private/tmp/crontab.* set nowritebackup nobackup
" Don't write backup file if vim is being called by "chpass"
au BufWrite /private/etc/pw.* set nowritebackup nobackup

let skip_defaults_vim=1
syntax on
set nu!
set autoindent
```
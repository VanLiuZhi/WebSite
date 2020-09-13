---
weight: 1000
title: "nvm 环境管理工具"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "nvm 环境管理工具"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Node.js, Note]
categories: [Node.js]

lightgallery: true

toc:
  auto: false
---

nvm 是 node的环境管理工具，可以同时安装多个node版本，具体实现是通过修改环境变量切换到对应的node上，不同的node版本拥有独立的包文件。

<!-- more -->

# nvm

nvm 是 node的环境管理工具，可以同时安装多个node版本，具体实现是通过修改环境变量切换到对应的node上，不同的node版本拥有独立的包文件。

## 安装

Mac 下安装使用github提供的脚本安装，安装完成添加对应shell的配置

nvm 使用brew安装会有一些小问题 [正确的安装和使用nvm(mac)](https://www.imooc.com/article/14617)

```sh
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
# This loads nvm bash_completion
```

github 地址 [https://github.com/creationix/nvm/blob/master/README.md](https://github.com/creationix/nvm/blob/master/README.md)


## 命令

| Command         	            |  Description                     
| ----------------------------- |:-------------------------------: 
| nvm install stable		    | 安装最新稳定版            
| nvm install \<version>		| 安装指定版本，可模糊安。如：安装v4.4.0，既可nvm install v4.4.0，又可nvm install 4.4   
| nvm uninstall \<version>		| 删除已安装的指定版本，语法与install类似  
| nvm ls		                | 列出所有安装的版本
| nvm ls-remote		            | 列出所有远程服务器的版本（官方node version list）
| nvm current		            | 显示当前的版本
| nvm alias \<name> \<version>  | 给不同的版本号添加别名
| nvm unalias \<name>		    | 删除已定义的别名
| nvm reinstall-packages \<version>	| 在当前版本 node 环境下，重新全局安装指定版本号的 npm 包


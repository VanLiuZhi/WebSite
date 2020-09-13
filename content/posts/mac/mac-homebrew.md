---
weight: 1000
title: "HomeBrew mac-os 软件管理工具"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "HomeBrew mac-os 软件管理工具"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Mac, Util]
categories: [Mac]

lightgallery: true

toc:
  auto: false
---

用mac电脑，你需要学会HomeBrew

<!-- more -->

## HomeBrew

mac 平台的包管理工具，官网地址[https://brew.sh/](https://brew.sh/)

```sh
安装命令
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

记得先安装Xcode，保证安装脚步需要的环境都是可行的。

常用命令

| Command         	            |  Description                    
| ----------------------------- |:-------------------------------:
| brew info [name]		        |           查看已安装包都信息       
| brew search [name]		    |           搜索包                 
| brew install remove rm [name] |           安装包                 
| brew uninstall [name]	        | 	        卸载包                 
| brew list		                |           查看已安装的包列表       
| brew cleanup		            |           删除文件残留            
| brew cleanup [name]		    |                                 
| brew deps [name]		        |           查看包的依赖            
| brew outdated                 |           查看需要更新的包         
| brew update                   |           更新包
| brew home [name]              |       用浏览器打开，查看包的网页信息
| brew options [name]           |       查看包的安装选项
| brew services list            |       查看homebrew安装的服务情况
| brew services start           |       启动服务，后面跟服务名称
| brew services stop            |       停止服务，后面跟服务名称
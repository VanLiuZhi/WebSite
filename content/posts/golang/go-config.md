---
weight: 1
title: "Go Config"
subtitle: "Go Config"
date: 2021-01-20T11:00:05+08:00
lastmod: 2021-01-20T11:00:05+08:00
draft: true
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Go Config"
# featuredImagePreview: "images/base-image.jpg"
# featuredImage: "/images/base-image.jpg"
# resources:
# - name: "featured-image"
#   src: "images/base-image.jpg"

tags: 
categories: 

lightgallery: true

toc:
  # 自动展开目录
  auto: false
---



<!--more-->

## 环境配置

内容来自知乎：https://zhuanlan.zhihu.com/p/111722890

1. go mod

go version >= 1.11

GO111MODULE一般有三个值

GO111MODULE=off，go命令行将不会支持module功能，寻找依赖包的方式将会沿用旧版本那种通过vendor目录或者GOPATH模式来查找。
GO111MODULE=on，go命令行会使用modules，而一点也不会去GOPATH目录下查找。
GO111MODULE=auto，默认值，go命令行将会根据当前目录来决定是否启用module功能。(新版本默认)
这种情况下可以分为两种情形：

```
1. 当前目录在GOPATH/src之外且该目录包含go.mod文件
2. 当前文件在包含go.mod文件的目录下面。
当modules 功能启用时，依赖包的存放位置变更为$GOPATH/pkg，允许同一个package多个版本并存，且多个项目可以共享缓存的 module。
```

2. goproxy代理

首先保证你的go版本是Go 1.13以上

$ go env -w GO111MODULE=on
$ go env -w GOPROXY=https://goproxy.cn,direct
作用：使 Go 在拉取模块版本时能够脱离传统的 VCS 方式从镜像站点快速拉取。

它拥有一个默认值proxy.golang.org在中国无法访问。

目前常用的代理是：

1、https://goproxy.cn

2、https://goproxy.io

其中，direct的作用是：特殊指示符，用于指示 Go 回源到模块版本的源地址去抓取

3. goprivate有什么用？

go 命令会从公共镜像 http://goproxy.io 上下载依赖包，并且会对下载的软件包和代码库进行安全校验，当你的代码库是公开的时候，这些功能都没什么问题。但是如果你的仓库是私有的怎么办呢？

环境变量 GOPRIVATE 用来控制 go 命令把哪些仓库看做是私有的仓库，这样的话，就可以跳过 proxy server 和校验检查，这个变量的值支持用逗号分隔，可以填写多个值，例如：

GOPRIVATE=*.corp.example.com,rsc.io/private
这样 go 命令会把所有包含这个后缀的软件包，包括 http://git.corp.example.com/xyzzy , http://rsc.io/private, 和 http://rsc.io/private/quux 都以私有仓库来对待。

举个例子：

GOPRIVATE=*.domain.cc  //一般domain.cc是你公司私有git仓库的域名地址，这样就可跳过proxy的检查

4. GOSUMDB

GOSUMDB（go checksum database）是Go官方为了go modules安全考虑，设定的module校验数据库，服务器地址为：sum.golang.org

你在本地对依赖进行变动（更新/添加）操作时，Go 会自动去这个服务器进行数据校验，保证你下的这个代码库和世界上其他人下的代码库是一样的。

和go.mod一样，Go 会帮我们维护一个名为go.sum的文件，它包含了对依赖包进行计算得到的校验值

环境变量GOSUMDB可以用来配置你使用哪个校验服务器和公钥来做依赖包的校验

Go1.13 中当设置了 GOPROXY="https://proxy.golang.org" 时 GOSUMDB 默认指向 "sum.golang.org"，其他情况默认都是关闭的状态。如果设置了 GOSUMDB 为 “off” 或者使用 go get 的时候启用了-insecure参数，Go 不会去对下载的依赖包做安全校验，存在一定的安全隐患，所以给大家推荐接下来的环境变量。

如果你的代码仓库或者模块是私有的，那么它的校验值不应该出现在互联网的公有数据库里面，但是我们本地编译的时候默认所有的依赖下载都会去尝试做校验，这样不仅会校验失败，更会泄漏一些私有仓库的路径等信息，我们可以使用GONOSUMDB这个环境变量来设置不做校验的代码仓库， 它可以设置多个匹配路径，用逗号相隔.

举个例子：

GONOSUMDB=*.corp.example.com,rsc.io/private
这样的话，像 "http://git.corp.example.com/xyzzy", "http://rsc.io/private", 和 "http://rsc.io/private/quux"这些公司和自己的私有仓库就都不会做校验了。


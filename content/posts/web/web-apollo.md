---
weight: 9900
title: "Apollo配置中心"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Apollo配置中心"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Linux, Note]
categories: [Middleware(中间件)]

lightgallery: true

toc:
  auto: false
---

Apollo配置中心

## 各模块的作用

`Config Service`提供配置的读取、推送等功能，服务对象是`Apollo客户端`
`Admin Service`提供配置的修改、发布等功能，服务对象是`Apollo Portal（管理界面）`
`Config Service`和`Admin Service`都是多实例、无状态部署，所以需要将自己注册到Eureka中并保持心跳
在Eureka之上我们架了一层`Meta Server`用于封装Eureka的服务发现接口
Client通过域名访问Meta Server获取Config Service服务列表（IP+Port），而后直接通过IP+Port访问服务，同时在Client侧会做load balance、错误重试
Portal通过域名访问Meta Server获取Admin Service服务列表（IP+Port），而后直接通过IP+Port访问服务，同时在Portal侧会做load balance、错误重试
为了简化部署，我们实际上会把Config Service、Eureka和Meta Server三个逻辑角色部署在同一个JVM进程中

## k8s部署难点

流程总结：

1. 从官网下载文件和源码jar包，准备构建镜像
2. 由于dockerfile需要执行脚本，最好把整个目录都赋权限777，公司内部构建，需要加代理(dockerfile 中增加ENV的代理环境变量)
3. 除了权限，执行脚本的模式也是个坑，在linux上通过vim进去，如果显示doc模式的，需要修改
doc模式启动容器会报错standard_init_linux.go:211: exec user process caused "no such file or directory"，可能和使用的基础镜像有关
两种解决方案，换基础镜像，或者改模式 set ff=unix 改模式(fileformat)
4. 如此后，便可以构建镜像了
5. 修改yaml文件，准备mysql，通过server和endpoint连接mysql服务给k8s中其它服务使用
6. 部署服务等待pod正常运行，有错误排查，基本可能出现的就是镜像构建有问题

## Core Concepts

应用: appid，应用标识
环境：应用在dev,pro环境下可以有不同的配置
集群：比环境更大的分组，比如数据中心是一个集群，这个集群下可以有dev,pro环境
命名空间：配置分组
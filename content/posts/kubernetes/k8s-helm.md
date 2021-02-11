---
weight: 1
title: "K8s Helm"
subtitle: "K8s Helm"
date: 2021-01-06T15:45:09+08:00
lastmod: 2021-01-06T15:45:09+08:00
draft: true
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "K8s Helm"
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

## 基础

- helm：
一个命令行下客户端工具，主要用于kubernetes应用chart的创建/打包/发布已经创建和管理和远程Chart仓库。
- Tiller：
helm的服务端，部署于kubernetes内，Tiller接受helm的请求，并根据chart生成kubernetes部署文件（helm称为release），然后提交给 Kubernetes 创建应用。Tiller 还提供了 Release 的升级、删除、回滚等一系列功能。
- Chart： 
helm的软件包，采用tar格式，其中包含运行一个应用所需的所有镜像/依赖/资源定义等，还可能包含kubernetes集群中服务定义
- Release：
在kubernetes中集群中运行的一个Chart实例，在同一个集群上，一个Chart可以安装多次，每次安装均会生成一个新的release。
- Repository：
用于发布和存储Chart的仓库

## 渲染模板

helm template 仓库名/包名 > test.yaml 
helm template ./文件夹名 > test.yaml 
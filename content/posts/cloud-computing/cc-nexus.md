---
weight: 3300
title: "Nexus3"
subtitle: "Nexus3"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Nexus3"
# featuredImagePreview: "images/Kubernetes/k8s.jpg"
# featuredImage: "/images/Kubernetes/k8s.jpg"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Cloud-Native]
categories: [Cloud-Native] 

lightgallery: true

toc:
  auto: false
---


<!--more-->

## 创建

从dockerHub下载镜像，按照文档说明创建容器服务

mkdir /data/nexus-data && chown -R 200 /data/nexus-data

docker run -d -p 8081:8081 --name nexus -v /data/nexus-data:/nexus-data -e INSTALL4J_ADD_VM_PARAMS="-Xms2g -Xmx2g -XX:MaxDirectMemorySize=3g -Djava.util.prefs.userRoot=/nexus-data/javaprefs" sonatype/nexus3:3.29.2

admin/admin123

## 基本

先创建一个仓库的存储目录，之后就可以创建仓库了

各个包管理模块的仓库类型不同，比如docker的，有这三种类型

hosted : 本地存储，即同 docker 官方仓库一样提供本地私服功能。
proxy : 提供代理其他仓库的类型，如 docker 中央仓库。
group : 组类型，实质作用是组合多个仓库为一个地址。

更多参考

https://www.cnblogs.com/sanduzxcvbnm/p/13099635.html

## go proxy

由于本地无法拉取镜像，所以我们使用proxy模式

https://goproxy.cn  只需要配置这个即可，然后把go的环境变量设置一下

## helm proxy

使用proxy模式

https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts

helm repo add nexus 代理地址
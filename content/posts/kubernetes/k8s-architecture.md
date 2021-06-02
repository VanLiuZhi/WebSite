---
weight: 1
title: "K8s Architecture"
subtitle: "K8s Architecture"
date: 2021-03-22T10:11:13+08:00
lastmod: 2021-03-22T10:11:13+08:00
draft: true
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "K8s Architecture"
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

## Containerd 

## Source to Image(S2I)

Source to Image(S2I) 是一个创建 Docker 镜像的工具，主要是openshift上用的比较多
一句话总结: 从源码构建镜像的一种方式，有自己的一定优势，适合Python这种依赖环境的语言

比如Python通过S2I构建镜像，基于基础环境替换代码就行

个人总结：感觉是openshift自己玩的东西，而且说在Python上有优势，但是Python的依赖改了基础环境也要改，只不过它都是从源码来的，总之记录一下即可，不做深入学习
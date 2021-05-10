---
weight: 1
title: "K8s Core"
subtitle: "K8s Core"
date: 2021-04-12T19:22:56+08:00
lastmod: 2021-04-12T19:22:56+08:00
draft: true
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "K8s Core"
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

## Informer 

Informer 是 client-go 下的一个工具包。在kubernetes的源码中，如果我们需要Get/List某个Object，绝大多数情况下，会直接使用Informer实例中的
Lister方法，很少去请求kubernetes API(一般就是 Kubernetes Get API 获取某个Object)

Informer 最基本 的功能就是 List/Get Kubernetes 中的 Object

Informer 只会调用 kubernets List/Watch 这两种API，初始化的时候通过List获取资源对象并缓存，然后通过Watch监听这种资源对象，之后就不再调用
k8s 的 API，通过watch维护缓存，没有resync机制，这要求etcd不会出错

二级缓存：DeltaFIFO 用来存储 Watch API 返回的各种事件 ，LocalStore 只会被 Lister 的 List/Get 方法访问


client-go 之 DeltaFIFO 实现原理: https://cloud.tencent.com/developer/article/1692474
深入理解k8s中的informer机制: https://www.cnblogs.com/yangyuliufeng/p/13611126.html
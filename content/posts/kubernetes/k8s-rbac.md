---
weight: 1103
title: "K8s Rbac"
subtitle: "K8s Rbac"
date: 2020-09-24T09:57:54+08:00
lastmod: 2020-09-24T09:57:54+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "K8s Rbac"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Cloud-Native, K8s]
categories: [Kubernetes] 

lightgallery: true

toc:
  auto: false
---



<!--more-->

## Service Accent 和 普通用户

在k8s中，我们关注的是Service Accent，它是一个名称空间级别的对象。而普通用户就是我们广义上的用户，k8s没有资源对象来表示这些用户

service accent由k8s的API调用创建。service accent都绑定了一组secret，secret被挂载到pod中，这样pod就可以获得调用kubernetes API的权限

## ServiceAccount 资源对象

和操作pod一样，我们可用获取ServiceAccount资源对象，它通过命名空间来划分。default命名空间下有一个默认的default名称的ServiceAccount
我们创建pod的时候如果不指定ServiceAccount，默认就是default

`spec.serviceAccountName` pod 通过这个字段来关联 ServiceAccount。创建pod的时候ServiceAccount必须存在，否则就报错
不能更新已经创建好的 Pod 的服务账户，可以清除服务账户

Service Account Controller 控制器确保每个新创建的命名空间都有一个名为default的 ServiceAccount

创建ServiceAccount的时候，会默认创建一个secrets

## 授权

创建ServiceAccount，并且绑定了secret，认证的过程交由系统去做，接下来就是授权的过程

## Role和ClusterRole

## 创建用户

https://blog.csdn.net/cbmljs/article/details/102953428

## 安全与上下文

如何在Pod容器中去执行宿主机命令

https://blog.gmem.cc/exec-host-cmd-from-within-a-pod
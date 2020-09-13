---
weight: 1000
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
categories: [Middleware]

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
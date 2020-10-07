---
weight: 9900
title: "Web Xxl Job"
subtitle: "Web Xxl Job"
date: 2020-09-21T10:39:41+08:00
lastmod: 2020-09-21T10:39:41+08:00
draft: true
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Web Xxl Job"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Note]
categories: [Middleware(中间件)]

lightgallery: true

toc:
  auto: false
---

XXL-JOB

<!--more-->


# XXL-JOB

基础架构:

由调度中心和执行器组成，调度中心需要用到数据库

执行器就是任务的定义源，引入特定的依赖后，按照规范编写任务，执行器可以和spring boot，spring等集成

调度中心就是负责去执行器中执行任务的，一个执行器可以定义多个任务

## 关于任务

有两种类型

1. Bean

这种类型的任务就是代码直接写好在执行器中的，通过调度中心配置后，和执行器中的代码做关联，实现任务调度

2. GLUE

这种类型任务代码存储在调度中心里面，需要注意的是能定义的任务和需要用到的包，必须是执行器有的，也就是说这种任务类型本质还是以执行器作为调度环境，只是代码不用事先准备好，灵活度比较高

## 用户权限

xxl的权限很简单，只有两种用户，普通和管理员

管理员有全部权限，普通用户权限和执行器绑定，普通用户只能操作他绑定的执行器，去发布任务。没有执行器管理页面

## 开发流程

数据传输key采用固定的，project 配置中心配置能登录Apollo的账户，使用非管理员账户
执行器增加项目字段
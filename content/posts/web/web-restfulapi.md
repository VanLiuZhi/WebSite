---
weight: 6600
title: "RESTFulAPI"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "RESTFulAPI"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Web, Note]
categories: [Clear-Mind]

lightgallery: true

toc:
  auto: false
---

一种接口风格，使用一种通用的风格，在团队开发中，也便于沟通。

<!-- more -->

## 概念

REST风格5个约束：客户端-服务端，无状态，缓存，统一接口，分层系统

1. 使用名词表示资源，动词通过HTTP方法来实现，比如删除资源使用DELETE方法。

2. 关注请求头，比如请求头Accept要求返回application/xml，如果服务器只能放回json格式的，应该返回406错误。

3. 使用正确的请求方法和状态码

不能一味的使用POST和GET方法，HTTP方法表

| Name    | Description 
| ------- | :-----
| OPTIONS | 用于获取资源支持的所以HTTP方法 
| HRAD    | 用于只获取请求某个资源返回的头信息 
| GET     | 用于从服务器获取某个资源的信息：1.完成请求后，返回状态码200 OK 2.完成请求后，需要返回被请求的资源详细信息 
| POST    | 用于创建新资源：1.创建完成后，返回状态码201 Created 2.完成请求后，需要返回被创建的资源详细信息 
| PUT     | 用于完整的替换资源或者创建指定身份的资源：1.如果是创建了资源，则返回201 Created 2.如果是替换了资源，则返回200 OK 
| PATCH   | 用于局部更新资源：1.完成请求后，返回状态码200 OK 2.完成请求后，需要返回被修改的资源详细信息 3.完成请求后，需返回被修改的资源详细信息 
| DELETE  | 用于删除某个资源，完成请求后返回状态码204 No Content 

对输出结果不在封装：通过状态码来判断请求，不应该把信息写在响应体中
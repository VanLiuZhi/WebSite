---
weight: 3300
title: "分布式链路追踪系统-skywalking的运用"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "分布式链路追踪系统-skywalking的运用"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Linux, Docker, Note, Cloud]
categories: [Cloud-Native]

lightgallery: true

toc:
  auto: true
---

## 架构图

![integration](http://skywalking.apache.org/assets/img/integration.2e80f9dd.jpg)

整个系统分为三部分：

- agent：采集tracing（调用链数据）和metric（指标）信息并上报
- OAP：收集tracing和metric信息通过analysis core模块将数据放入持久化容器中（ES，H2（内存数据库），mysql等等），并进行二次统计和监控告警
- webapp(UI)：前后端分离，前端负责呈现，并将查询请求封装为graphQL提交给后端，后端通过ribbon做负载均衡转发给OAP集群，再将查询结果渲染展示

## 镜像版本选择

本次提供了8.0.0和6.6.0的部署文件，主要是3个镜像文件。由于官方提供的部署方式是Helm的，所以我们自己去Docker Hub上找到官方镜像，编写yaml文件来部署

选择ui和oap相匹配的版本，注意oap有es版本，因为官方默认也是用es来做存储的，所以我们也采用es的版本

1. apache/skywalking-oap-server:6.6.0-es7 or apache/skywalking-oap-server:8.0.0-es7
2. apache/skywalking-ui:6.6.0 or apache/skywalking-ui:8.0.0

由于agent官方是没有提供镜像的，需要我们去官方发行版中找到agent的文件，自己制作镜像

下载地址 http://skywalking.apache.org/downloads/ 选择对应的版本下载

关于镜像需要了解的点就这些

## 
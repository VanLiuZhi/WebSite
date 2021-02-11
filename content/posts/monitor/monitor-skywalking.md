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

cpm 全称 call per minutes，是吞吐量(Throughput)指标

SLA 全称 Service-Level Agreement，直译为 “服务等级协议”，用来表示提供服务的水平
在IT中，SLA可以衡量平台的可用性，下面是N个9的计算：

1年 = 365天 = 8760小时

99 = 8760 * 1% => 3.65天

99.9 = 8760 * 0.1% => 8.76小时

99.99 = 8760 * 0.01% => 52.6分钟

99.999 = 8760 * 0.001% => 5.26分钟

因此，全年只要发生一次较大规模宕机事故，4个9肯定没戏，一般平台3个9差不多。 但2个9就基本不可用了，相当于全年有87.6小时不可用，每周(一个月按4周算)有1.825小时不可用。 下图是服务、实例、接口的SLA，一般看年度、月度即可

Percent Response 百分位数统计
表示采集样本中某些值的占比，Skywalking 有 p50、p75、p90、p95、p99 一些列值。途中的 “p99:390” 表示 99% 请求的响应时间在390ms以内。而99%一般用于抛掉一些极端值，表示绝大多数请求。

Slow Endpoint 慢端点
Endpoint 表示具体的服务，例如一个接口。下面是全局Top N的数据，通过这个可以观测平台性能情况。

Heatmap 热力图
Heapmap 可译为热力图、热度图都可以，途中颜色越深，表示请求数越多，这和GitHub Contributions很像，commit越多，颜色越深。 横坐标是响应时间，鼠标放上去，可以看到具体的数量。 通过热力图，一方面可以直观感受平台的整体流量，另一方面也可以感受整体性能。

apdex
是一个衡量服务器性能的标准。 apdex有三个指标：

满意：请求响应时间小于等于T。

可容忍：请求响应时间大于T，小于等于4T。

失望：请求响应时间大于4T。

T：自定义的一个时间值，比如：500ms。 apdex = （满意数 + 可容忍数/2）/ 总数。 例如：服务A定义T=200ms，在100个采样中，有20个请求小于200ms，有60个请求在200ms到800ms之间，有20个请求大于800ms。计算apdex = (20 + 60/2)/100 = 0.5
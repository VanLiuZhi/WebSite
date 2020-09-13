---
weight: 1000
title: "python web 开发(四) 项目测试"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "python web 开发(四) 项目测试"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Python, Web, Technology]
categories: [Technology技术]

lightgallery: true

toc:
  auto: false
---

项目的前端到后端都准备好了，下面开始运行它，本来计划部署到阿里云服务器上的，但是200多的服务器还是太弱了，可以勉强跑起来，但是cup占用率过半后，导致远程连接卡到无法使用，只能在本地模拟了，由于是用的docker，其实和服务器上差异不大了。

<!-- more -->

## 准备工作

[后端代码地址](https://github.com/VanLiuZhi/flask_starlight.git)

[前端代码地址](https://github.com/VanLiuZhi/element_UI_Web.git)

由于前端代码我已经编译好了，要想跑起项目直接使用后端代码就行了。

在 `docker-compose.yaml` 修改你的文件路径，准备需要映射的数据目录（MySQL，Redis）

编辑Nginx配置文件，使用了一个虚拟域名 `test.liuzhi.com` 用以模拟域名访问。

## 运行

进入 `compose_docker_dev` 目录，先通过 `Dockerfile` 创建Python环境镜像，编辑 `docker-compose.yaml`，通过命令 `docker-compose up -d` 启动服务。默认是跑flask自带的服务器，改成 `gunicorn auto_app:app -c gunicorn.py`，使用gunicorn服务器来启动项目

启动服务后，进入Python容器，做数据库初始化，这样就可以访问首页了。

通过 python crawling 抓取别人的ress订阅的数据，展示在首页上。在此之前记得启动celery，celery负责处理后台任务。

运行后看起来应该是这样的：

![image](/images/Blog/python-advanced-04/2019-02-12-22.14.27.png)

## 总结

本项目是一个单机可用的开发环境，包含的python开发的完整服务，太偏业务的内容没有讲太多，看代码差不多就能了解了。

这样的架构还是比较普通的，如果要应对高并发，只是读取缓存的并发请求，添加多台Redis实例，通过nginx负载均衡调度应用服务器，可以抗下来，这方面的经验不多，是后续可以学习和改进的地方。

在后续的下一次总结中，计划做一个有实际业务需求的，其实我本来也搞过，不过在数据上玩脱了，没有数据写的东西不太有价值，当时是看了很多公司提供的API来作为数据源的，不过要不就是需要翻墙，要么就是接口变的不提供数据了，唯一可用的GitHub的API接口实在是过于繁琐了，使用了最新的 GraphQL 接口风格，是Facebook开发的用于 API 的查询语言，要完全运用还得花时间。所以最好的数据来源还是用爬虫吧，准备学习一下爬虫，做一个数据分析的项目，把所学的东西都调度起来。

搞IT就是要能折腾，看别人怎么写，看官方的文档，从中总结，把别人讲解的变成自己的，多去实践。
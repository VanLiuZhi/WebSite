---
weight: 1000
title: "gunicorn python 应用服务器"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "gunicorn python 应用服务器"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Python, Linux, Note]
categories: [Python编程]

lightgallery: true

toc:
  auto: false
---

Gunicorn 'Green Unicorn' is a Python WSGI HTTP Server for UNIX. It's a pre-fork worker model. The Gunicorn server is broadly compatible with various web frameworks, simply implemented, light on server resources, and fairly speedy.

<!-- more -->

## 运行

    gunicorn -w 2 --threads=4 application:app -b localhost:8080

    gunicorn -c HttpTest/gunicorn.py HttpTest.wsgi:application 指定配置文件

-w 指启动的进程数量如2

`ps -ef|grep python` 可以看到python共有3个进程，其中一个主进程和两个work进程（两个work进程是master的子进程）

注意，线程配置
threads = 100 

假如用sync模式，线程数不能不设置，否则就是1，即分发到每个工作进程的线程只能有1，一定要配，如果是其它模式，协程和io多路复用都只需要1个线程，配置无用

gunicorn 工作模式

- sync 默认使用同步阻塞的网络模型（-k sync）
- eventlet - Requires eventlet >= 0.9.7
- gevent - Requires gevent >= 0.13
- tornado - Requires tornado >= 0.2
- gthread - Python 2 requires the futures package to be installed
- gaiohttp - Requires Python 3.4 and aiohttp >= 0.21.5

使用 `-k` 指定工作模式

Linux进程有父进程和子进程之分，windows的进程是平等关系

gunicorn的同步模式，一次只处理一个请求，虽然python有GIL，但不是所有操作都是线程安全的。

## 指定配置参数

通过 `-c` 来指定配置文件来运行，就不需要把参数写入命令。

## 问题

遇到的问题汇总

### 启动django项目，找不到静态资源

gunicorn 来直接启动django项目，找不到静态资源，在url配置文件中加入

```python

from django.contrib.staticfiles.urls import staticfiles_urlpatterns
urlpatterns += staticfiles_urlpatterns()

```

问题解决来自 [Stack Overflow](https://stackoverflow.com/questions/12800862/how-to-make-django-serve-static-files-with-gunicorn)
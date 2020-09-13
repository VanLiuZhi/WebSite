---
weight: 1000
title: "Django的默认服务器"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Django的默认服务器"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Python, Web, Technology]
categories: [Technology技术]

lightgallery: true

toc:
  auto: false
---

很多web框架都会自带服务器，今天就来看看Django的默认服务器是怎么运行的，本次使用的Django版本为2.1.7

<!-- more -->

## WSGI

WSGI协议是Python为web服务器制定的标准，只要遵循WSGI协议，应用服务器就可以和Python程序实现通信，把http请求发送给Python程序，并返回响应。在Django中的 `wsgi.py` 就是一个 WSGI协议的实现，并返回了application。详情看[PEP333](https://www.python.org/dev/peps/pep-0333/)。

Django 通过 manage.py 来执行命令，原理就是收集命令行参数，去执行对应模块的方法，runserver就是启动默认服务器的命令，最终执行的代码如下

```py
def run(addr, port, wsgi_handler, ipv6=False, threading=False, server_cls=WSGIServer):
    server_address = (addr, port)
    if threading:
        httpd_cls = type('WSGIServer', (socketserver.ThreadingMixIn, server_cls), {})
    else:
        httpd_cls = server_cls
    httpd = httpd_cls(server_address, WSGIRequestHandler, ipv6=ipv6)
    if threading:
        # ThreadingMixIn.daemon_threads indicates how threads will behave on an
        # abrupt shutdown; like quitting the server by the user or restarting
        # by the auto-reloader. True means the server will not wait for thread
        # termination before it quits. This will make auto-reloader faster
        # and will prevent the need to kill the server manually if a thread
        # isn't terminating correctly.
        httpd.daemon_threads = True
    httpd.set_app(wsgi_handler)
    httpd.serve_forever()
```
文件路径：`python3.7/site-packages/django/core/servers/basehttp.py`

调用的是 `from wsgiref.simple_server import WSGIServer` WSGIServer 的 serve_forever 方法，这个wsgiref是标准库提供的。

所以要看Django默认服务是怎么运行了，分析server_forever就行了。

```pysu
def serve_forever(self, poll_interval=0.5):
    """Handle one request at a time until shutdown.

    Polls for shutdown every poll_interval seconds. Ignores
    self.timeout. If you need to do periodic tasks, do them in
    another thread.
    """
    self.__is_shut_down.clear()
    try:
        # XXX: Consider using another file descriptor or connecting to the
        # socket to wake this up instead of polling. Polling reduces our
        # responsiveness to a shutdown request and wastes cpu at all other
        # times.
        with _ServerSelector() as selector:
            selector.register(self, selectors.EVENT_READ)

            while not self.__shutdown_request:
                ready = selector.select(poll_interval)
                if ready:
                    self._handle_request_noblock()

                self.service_actions()
    finally:
        self.__shutdown_request = False
        self.__is_shut_down.set()
```

文件路径：`python3.7/socketserver.py`

注释描述了方法的作用，服务启动后一直运行下去，每次只处理一个请求，使用IO多路复用的形式。也提示如果要进行长时间任务，最好在另一个线程中进行。
     
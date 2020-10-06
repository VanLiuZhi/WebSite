---
weight: 5500
title: "Locust测试工具"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Locust测试工具"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Python, Web]
categories: [Python]

lightgallery: true

toc:
  auto: false
---

locust 是一个简单易用的测试工具。

官方文档：[https://docs.locust.io/en/stable/what-is-locust.html](https://docs.locust.io/en/stable/what-is-locust.html)

<!-- more -->

## 文件描述符限制

如果做高并发测试，操作系统会限制一个进程能够创建的文件描述符上限制，因为每个tcp连接都需要用到一个socket句柄。

`ulimit -n 可以查看系统允许当前用户打开的文件数限制` </br> `ulimit -n 65535 可以修改限制，本次有效果`

根据官方文档描述来看，模拟用户数超过能打开的文件数，会出现错误。

## 如何使用
安装locust后，通过编写脚本，然后启动服务，此时通过浏览器端，就可以进行测试了。
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Time    : 2018/10/7 下午10:46
# @Author  : liuzhi
# @File    : locustfile.py

from locust import HttpLocust, TaskSet, task

class UserBehavior(TaskSet):
    @task
    def sleep(self):
        self.client.get("/sleep")

    @task
    def woek(self):
        self.client.get("/work")


class WebsiteUser(HttpLocust):
    task_set = UserBehavior
    min_wait = 0  # 单位 ms 请求等待的最小时间
    max_wait = 1

# 测试server
# locust -f test_14.py --host=http://localhost:8888
这里配置的host是要去测试的服务地址，Locust服务的地址会在启动信息中打印出来

```
上述代码主要有两个类，UserBehavior类两个任务就是模拟当前用户的行为，sleep和work代表本次请求的用户将执行sleep和work操作，可以设置随机或同顺序，或者执行的比重。</br>`locust --host=http://localhost:8888`命令启动locust，8888代表测试服务的端口。

## UI界面描述

- Type：请求类型；
- Name：请求路径；
- requests：当前请求的数量；
- fails：当前请求失败的数量；
- Median：中间值，单位毫秒，一般服务器响应时间低于该值，而另一半高于该值；
- Average：所有请求的平均响应时间，毫秒；
- Min：请求的最小的服务器响应时间，毫秒；
- Max：请求的最大服务器响应时间，毫秒；
- Content Size：单个请求的大小，单位字节；10.reqs/sec：每秒钟请求的个数。
- RPS: 服务器每秒能处理的请求数
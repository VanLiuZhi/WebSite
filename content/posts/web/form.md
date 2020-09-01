---
weight: 1
title: "FormData 表单对象"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "FormData 表单对象"
resources:
- name: "base-image"
  src: "base-image.jpg"

tags: [Web]
categories: [Web]

lightgallery: true

toc:
  auto: false
---

## 基础

FormData 对象，可以创建表单对象，通过append方法添加字段，也可以传入表单DOM对象，获取页面已经输入的值（不过该方法在element-ui中无用），猜测
是因为DOM被Vue托管，属性已经发生改变，无法被原生对象FormData获取到数据，所以要注意使用FormData对象后，如何和后端交互的问题，可以参考代码。

ContentType指的是请求体的编码类型，常见的类型共有3种：

1. application/x-www-form-urlencoded
浏览器原生表单<form>默认的提交数据的方式（就是没有设置enctype属性），POST提交数据的默认方式。

application/x-www-form-urlencoded 方式提交数据

POST http://www.example.com HTTP/1.1
Content-Type: application/x-www-form-urlencoded;charset=utf-8

`下面是浏览器传递时数据的格式`
name=qwe&pwd=123

2. multipart/form-data，和第一种类似，但是如果是文件上传，那么要用这个enctype

Request URL:http://127.0.0.1:8000/index/
Content-Type:multipart/form-data; boundary=----WebKitFormBoundaryExT8avmSnrECoDbP

`下面是浏览器传递时数据的格式，可以看到加了很多内容`
------WebKitFormBoundaryExT8avmSnrECoDbP
Content-Disposition: form-data; name="name"

qwe
------WebKitFormBoundaryExT8avmSnrECoDbP
Content-Disposition: form-data; name="pwd"

123
------WebKitFormBoundaryExT8avmSnrECoDbP
Content-Disposition: form-data; name="icon"; filename="0fbc751ff63fa8cf3302b03889b9421e65d6592301"
Content-Type: application/octet-stream


------WebKitFormBoundaryExT8avmSnrECoDbP--

3. application/json
application/json 这个 Content-Type 常作为作为响应头，现在页经常使用它作为请求头，用来告诉服务端消息主题是序列化后的JSON字符串。

由于 JSON 规范的流行，除了低版本 IE 之外的各大浏览器都原生支持 JSON.stringify，服务端语言也都有处理 JSON 的函数，使用 JSON 不会遇上什么麻烦。
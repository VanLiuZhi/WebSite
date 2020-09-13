---
weight: 1000
title: "element-UI 前端 Vue UI 组件"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "element-UI 前端 Vue UI 组件"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [JavaScript, Framework, Note]
categories: [JavaScript]

lightgallery: true

toc:
  auto: false
---

饿了么前端团队出品的Vue UI框架

<!-- more -->

## from rules 

from rules 是表单组件中用于在表单验证规则的属性，把rules绑定给一个对象，对象的属性即为需要做验证的字段，属性值为验证规则

```js

rules: {
type: [{ required: true, message: 'type is required', trigger: 'change' }],
timestamp: [{ type: 'date', required: true, message: 'timestamp is required', trigger: 'change' }]
}

```

对表单的type字段做规则验证，该字段是必须的。

## Loading

关于 Loading  拥有指令调用，服务调用（在需要的地方手动触发），指令的话就是 v-loading='true'
即可。可以通过修饰符把效果的遮蔽罩覆盖到DOM的body上。
---
weight: 5500
title: "用 marshmallow 模块序列化数据"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "用 marshmallow 模块序列化数据"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Python, Note]
categories: [Python]

lightgallery: true

toc:
  auto: false
---

在接口开发中，从数据库查询得到的数据通常是需要转换的，或者从一个对象序列化需要的特定数据，可以为此写一个方法，为何不尝试使用marshmallow模块呢，它提供了非常全面的数据处理方案。

<!-- more -->

## 概述

一般用法是创建scheme的类，使用实例化的dump方法，方法接受待序列化的对象，即可返回一个被处理过的marshmallow对象，同时也可以反序列化。

## scheme 类

```py
from marshmallow import Schema, fields

import datetime

class BaseSchema(Schema):
    id = fields.Str()
    created_at = fields.Str()
    updated_at = fields.Str()
```

## 序列化数据

```py
def marshal(data, schema):
    if isinstance(data, (list, tuple)):
        return [marshal(d, schema) for d in data]

    result, errors = schema.dump(data)
    if errors:
        for item in errors.items():
            print('{}: {}'.format(*item))
    return result

class Item(object):
    created_at = datetime.datetime.now()
    updated_at = datetime.datetime.now()

instance = Item()

marshal(instance, BaseSchema())
```

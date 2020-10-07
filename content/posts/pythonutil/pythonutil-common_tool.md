---
weight: 5500
title: "python常用方法"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "python常用方法"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Python, Note]
categories: [Python]

lightgallery: true

toc:
  auto: false
---

记录python常用方法

<!-- more -->

## 获取文件所在目录

```python

import os

path = os.path.dirname(os.path.abspath(__file__))
print(path)
    
```

## dict扩展

```python

class AttrDict(object):
    """
    不继承dict，实现一个dict，实例化使用关键字参数的形式：AttrDict(a=1, b=2)
    __getitem__，__setitem__，__delitem__称为容器方法，就是_dict['a']这种操作的
    拦截，而__getattribute__（访问属性先执行这个，然后执行__getattr__），
    __setattr__这种才是属性拦截，通过把关键字参数赋值给__dict__，实现了可以通过
    _dict.a 或 _dict['a'] 的形式访问字典键值
    """

    def __init__(self, **kwargs):
        self.__dict__.update(**kwargs)

    def __getitem__(self, item):
        return self.__getattribute__(item)

    def __setitem__(self, key, value):
        return self.__setattr__(key, value)

    def __delitem__(self, key):
        return self.__delattr__(key)

    def __len__(self):
        return len(self.__dict__)


attr_dict = AttrDict(a=2, b=3)
print(attr_dict['a'])
print(attr_dict.a)


# print(attr_dict.get('a')) 报错的，这是字典的方法，这个类没有继承字典

class AttrDict2(dict):
    """
    继承dict，扩展链式操作
    """

    def __init__(self, *args, **kwargs):
        print('init a dict')
        # dict.__init__(self, *args, **kwargs) # 同下super的效果
        super().__init__(*args, **kwargs)
        # 继承dict的对象，对象的实例就是一个字典，print(_dict)可以看到，既然如此，把这个
        # 字典给到__dict__，那么对象的属性就有了
        self.__dict__ = self


_dict = AttrDict2()
_dict['a'] = 33
print(_dict)
print(_dict.__dict__)

# 如果再次执行_dict['b'] = 44，已经不执行__init__方法了，但是self.__dict__ = self已经关联了，
# dict做为可变变量类型，同样更新了__dict__
_dict['b'] = 44
print(_dict)
print(_dict.__dict__)
print(_dict.a, _dict.b)
    
```

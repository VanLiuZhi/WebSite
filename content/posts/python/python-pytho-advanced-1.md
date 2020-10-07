---
weight: 5500
title: "python 特殊方法"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "python 特殊方法"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Python, Note]
categories: [Python]

lightgallery: true

toc:
  auto: false
---

常见特殊方法总结

<!-- more -->

## 描述符

{% blockquote %}
property(fget=None, fset=None, fdel=None, doc=None) -> property attribute
  
fget is a function to be used for getting an attribute value, and likewise
fset is a function for setting, and fdel a function for del'ing, an
attribute.  Typical use is to define a managed attribute x
{% endblockquote %}

基本写法
```py
class C(object):
    def getx(self): return self._x
    def setx(self, value): self._x = value
    def delx(self): del self._x
    x = property(getx, setx, delx, "I'm the 'x' property.")
```

在这个类中，x属性被描述符托管，对x的修改，复制都会被拦截（需要描述符类实现了这些方法）

Decorators make defining new properties or modifying existing ones easy:

装饰器写法

```py
class C(object):
    @property
    def x(self):
        "I am the 'x' property."
        return self._x
    @x.setter
    def x(self, value):
        self._x = value
    @x.deleter
    def x(self):
        del self._x
```

要知道静态方法，类方法，property都是构建描述符的类，这种方法有一个特点，就是对当前对象的属性进行维护

就是将某种特殊类型的类的实例指派给另一个类的属性(注意：这里是类属性，而不是对象属性)。

描述符有两种情况，静态方法，类方法，property都可以归为一种情况，这种情况有弊端就是维护多个属性很困难，不利于代码重构

另一种情况就是使用描述符对象，能够实现在多个属性上重复利用同一个存储逻辑的方式，把一个类的操作托付与另一个类

使用描述符类

```py
class Age(object):
    # _value = '1'
    def __init__(self, age):
        print(age, id(self))
        self.age = age

    def __set_name__(self, owner, name):
        self.name = name

    def __get__(self, instance, owner):
        return self.age, id(self)

    def __set__(self, instance, value):
        self.age = value

    def __delete__(self, instance):
        del self.age


class XiaoMin:
    age_1 = Age('18')
    age_2 = Age('19')


a = XiaoMin()
b = XiaoMin()

print(b.age_1)
a.age_1 = '20'

print(a.age_1)
print(b.age_1)
```

需要注意py3.6使用了新的描述符协议，增加了__set_name__特殊方法，这里传递的name就会指向age_1，age_2
实例a, b的age_1属性都是同一个，描述符类中的instance指向实例a, 或b，owner指向类XiaoMin

a.age_1和b.age_1都是一样的，a.age_1修改也会影响b.age_1。只有instance的执行不一样
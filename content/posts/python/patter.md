---
weight: 1
title: "python-patter 相关设计模式"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "python-patter 相关设计模式"
resources:
- name: "base-image"
  src: "base-image.jpg"

tags: [Python, Note]
categories: [Python编程]

lightgallery: true

toc:
  auto: false
---

Python常见设计模式总结

<!-- more -->

## Borg 模式

这个模式创建的实例，属性都是共享的

```python

def borg(cls):
    cls._state = {}
    orig_init = cls.__init__

    def new_init(self, *args, **kwargs):
        self.__dict__ = cls._state
        orig_init(self, *args, **kwargs)

    cls.__init__ = new_init
    return cls


@borg
class A:
    ...


a = A()
a.b = '2'
b = A()
print(a.b, b.b)

```

## Pool 对象池模式

以下代码实现了维护一定线程池

```python

import time
import threading
from contextlib import contextmanager


class ObjectPool:

    def __init__(self, klass, *args, max_size=None, timeout=0.5, **kwargs):
        self._klass = klass
        self._max_size = max_size
        self._size = 0
        self._args = args
        self._kwargs = kwargs
        self._tiemout = timeout
        self._items = []
        self._mutex = threading.Lock()
        self._item_available = threading.Condition(self._mutex)

    def get(self):
        with self._mutex:
            if not self._items and (self._max_size is None or
                                    self._size < self._max_size):
                item = self._klass(*self._args, **self._kwargs)
                self._size += 1
            else:
                while not self._items:
                    self._item_available.wait(self._tiemout)
                item = self._items.pop()
        return item

    def put(self, item):
        with self._mutex:
            self._items.append(item)
            self._item_available.notify()

    @contextmanager
    def item(self):
        item = self.get()
        try:
            yield item
        finally:
            self.put(item)


class Test:
    def __init__(self, a, b=1):
        self.a = a
        self.b = b


pool = ObjectPool(Test, 1, b=2, max_size=3)


class MyThread(threading.Thread):
    def run(self):
        with pool.item() as item:
            print(f'<{item.__class__.__name__} at {id(item)}>')
            time.sleep(0.1)


def main():
    threads = []
    for i in range(10):
        t = MyThread()
        t.start()
        threads.append(t)
    for t in threads:
        t.join(True)


if __name__ == '__main__':
    main()

```
---
weight: 5500
title: "collections 模块使用"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "collections 模块使用"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Python, Note]
categories: [Python]

lightgallery: true

toc:
  auto: false
---

collections模块是python的内建模块，提供类很多实用的集合类。有时候很多数据结构其实标准库已经实现了，不妨来这里找找有没有你想要的。内容参考官方文档。

<!-- more -->

## nametuple(命名元组)
从名字可以看出，这是一个对tuple定义name的类，通过nametuple创建的tuple有了名字，最重要的是可以通过属性来访问`new tuple` 的元素。

{% blockquote %}
`collections.namedtuple(typename, field_names, *, rename=False, defaults=None, module=None)`
Returns a new tuple subclass named typename. The new subclass is used to create tuple-like objects that have fields accessible by attribute lookup as well as being indexable and iterable. Instances of the subclass also have a helpful docstring (with typename and field_names) and a helpful __repr__() method which lists the tuple contents in a name=value format.
{% endblockquote %}

示例：
```python
from collections import namedtuple
Point = namedtuple('Point', ['x', 'y'])
p = Point(11, 22)
x, y = p
print(x, y)
```
>输出 11 22

注意：field_name参数不能是关键字，也不能有重复。rename参数用于当field_name重名的时候，自动进行重命名。

```python
from collections import namedtuple
Point = namedtuple('Point', ['x', 'y', 'x'], rename=True)
p = Point(11, 22, 33)
print(p)
print(p._fields)
```
>输出<br/> Point(x=11, y=22, _2=33)<br/>('x', 'y', '_2')<br/>
重命名的时候使用了下划线 _ 加元素所在索引数的方式进行重命名

### classmethod somenamedtuple._make(iterable)

该方法用来给实例赋值，这通常配合一些有序数据处理，如csv
```py
from collections import namedtuple
Point = namedtuple('Point', ['x', 'y'], rename=True)
t = [11, 22]
p = Point._make(t)
print(Point, p)
```
>输出 <class '__main__.Point'> Point(x=11, y=22)

## deque(双向列表)

使用list存储数据时，按索引访问元素很快，但是插入和删除元素就很慢了，因为list是线性存储，数据量大的时候，插入和删除效率很低，deque是为了高效实现插入和删除操作的双向列表，适合用于队列和栈。

示例
```py
from collections import deque
raw_list = ['1', '2', '3', 'a', 'b', 'c']
new_list = deque(raw_list)
new_list_2 = deque(['11', '22'])
b = new_list_2 + new_list
print(b)
print(new_list, type(new_list), isinstance(new_list, list))
```
>OutPut
deque(['11', '22', '1', '2', '3', 'a', 'b', 'c'])
deque(['1', '2', '3', 'a', 'b', 'c']) <class 'collections.deque'> False

可以看到，deque创建到列表不是list的实例，只有deque的实例能相互进行运算。
deque支持的方法和list类似，多了像appendleft, popleft等方法，remove，reverse等通用方法，详情参考文档。

## defaultdict

在对字典操作的时候，如果没有值会key错误，通过defaultdict创建的字典，第一个参数传递一个callable对象或者None(None创建的字典和原字典没有区别)，如果key不存在则返回callable调用的值。
```py
from collections import defaultdict
a = [('a', 2), ('b', 3)]
_dict = defaultdict(int, a)
print(_dict, isinstance(_dict, dict))
print(_dict['aa'])
```
>OutPut
defaultdict(<class 'int'>, {'a': 2, 'b': 3}) True
0

使用int返回0，list返回[]，或者使用:lambda: 'The key doesn't exist'。
对于需要对字典key不存在的时候，返回统一值，就可以使用defaultdict。
使用get访问字典也是不错的编程习惯，在不想触发异常。

## OrderedDict(有序字典)

使用OrderedDict创建一个有序字典，在迭代的时候就按照创建的顺序来。
```py
from collections import OrderedDict
a = [('a', 1), ('b', 2), ('c', 3)]
_dict = OrderedDict(a)
print(_dict, isinstance(_dict, dict))
```

## Counter

用来统计字符出现的个数，结果返回一个dict
```py
from collections import Counter
a = [('a', 1), ('b', 2), ('c', 3)]
c = Counter('aasjaslajl')
c2 = Counter(['aasjaslajl', 'aa', 'bb', 'aa'])
c3 = Counter(a)
print(c, c2, c3)
print(sorted(c3.elements()))
print(isinstance(c, dict))
```
>OutPut 
Counter({'a': 4, 's': 2, 'j': 2, 'l': 2}) Counter({'aa': 2, 'aasjaslajl': 1, 'bb': 1}) Counter({('a', 1): 1, ('b', 2): 1, ('c', 3): 1})
[('a', 1), ('b', 2), ('c', 3)]
True
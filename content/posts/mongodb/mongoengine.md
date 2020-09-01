---
weight: 1
title: "MongoEngine 官方文档学习笔记"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "MongoEngine 官方文档学习笔记"
resources:
- name: "base-image"
  src: "base-image.jpg"

tags: [DataBase, Note, Python]
categories: [DataBase]

lightgallery: true

toc:
  auto: false
---

文档[http://docs.mongoengine.org/index.html](http://docs.mongoengine.org/index.html)
MongoEngine是Python操作MongoDB的ORM封装，可以看到很多ORM框架的影子，比如Django的ORM。底层调用了pymongo。

基本用法都是创建Document的class，得到class的instance，然后去操作instance。

MongoDB数据库是文档型的，在一个集合中，每一个文档都可以是不同的结构，不过使用了ORM，查询写入都会受到ORM的限制，当然你应该规范文档的结构。

ORM提供的功能比较有限，基础操作满足不了的，查pymongo，框架底层是基于pymongo。

<!-- more -->

# Document instances

## clean
实例方法，用来做save前操作。

## Cascading Saves
If your document contains ReferenceField or GenericReferenceField objects, then by default the save() method will not save any changes to those objects. 
如果文档包含ReferenceField或者GenericReferenceField字段，save方法不会保存他们的修改，需要在save(cascade=True)设置，save方法描述：
>:param cascade: Sets the flag for cascading saves.  You can set a default by setting "cascade" in the document __meta__
暂时没有做过测试。

## delete
执行delete，需要有id字段。

## Document IDs

文档要保存了才能访问id，通过情况不需要声明id字段
```s
>>> page = Page(title="Test Page")
>>> page.id
>>> page.save()
>>> page.id
ObjectId('123456789abcdef000000000')
```

通过设置字段的关键字来创建id，这里把email作为id，事实上id是主键的别名，pk == id 是等价的
? 修改了默认主键，是不是就不存在唯一表示了，既没有了ObjectId('123456789abcdef000000000')
```s
>>> class User(Document):
...     email = StringField(primary_key=True)
...     name = StringField()
...
>>> bob = User(email='bob@example.com', name='Bob')
>>> bob.save()
>>> bob.id == bob.email == 'bob@example.com'
True
```

# Querying the database
QuerySetManager QuerySet 的概念在MongoEngine中也是适用的。
查询集使用本地缓存，如果想返回新的结果，使用no_cache方法。

## Filtering queries
user = User.objects(name='liu zhi')

## Query operators
```
ne – not equal to 不等于
lt – less than 小于
lte – less than or equal to 小于等于
gt – greater than 大于
gte – greater than or equal to 大于等于
not – negate a standard check, may be used before other operators (e.g. Q(age__not__mod=5)) 否定其它条件，比如查询所有age不在[20, 30]中的 age__not__in=[20, 30]
in – value is in list (a list of values should be provided)
nin – value is not in list (a list of values should be provided)
mod – value % x == y, where x and y are two provided values
all – every item in list of values provided is in array
size – the size of the array is
exists – value for field exists
```

### String queries
The following operators are available as shortcuts to querying with regular expressions:

```
exact – string field exactly matches value
iexact – string field exactly matches value (case insensitive)
contains – string field contains value
icontains – string field contains value (case insensitive)
startswith – string field starts with value
istartswith – string field starts with value (case insensitive)
endswith – string field ends with value
iendswith – string field ends with value (case insensitive)
match – performs an $elemMatch so you can match an entire document within an array
```

### Geo queries(特定字段扩展的查询)
PointField, LineStringField and PolygonField字段增加了特殊的查询方法，详情看文档。

### Querying lists(查询list字段的扩展)

### Raw queries(pymongo查询)
使用pyMongo的原生查询，document.objects(__raw__={'name': 'liuzhi'})

### Limiting and skipping results
使用切片实现原生db.document.find().limit().skip()，get，first方法，get是检索唯一结果，如果有多个结果匹配，会触发MultipleObjectsReturned异常。get_or_create()已经弃用，最好不要使用，由于没有事务的原因，它不是安全的。

### Default Document queries(扩展模型管理器)
相当于对objects重写，使用特定的装饰器，方法名字可以自定义，这样可以做到使用原始的查询原始数据，使用自定义，查询自定义数据，比如自定义的只查询状态是正常的。
```
class BlogPost(Document):
    title = StringField()
    date = DateTimeField()

    @queryset_manager
    def objects(doc_cls, queryset):
        # This may actually also be done by defining a default ordering for
        # the document, but this illustrates the use of manager methods
        return queryset.order_by('-date')
```

### Custom QuerySets(封装查询集方法)
把某些特定查询条件组合，通过新的方法获取查询集，可以给多个文档模型使用。
```
class AwesomerQuerySet(QuerySet):

    def get_awesome(self):
        return self.filter(awesome=True)

class Page(Document):
    meta = {'queryset_class': AwesomerQuerySet}

# To call:
Page.objects.get_awesome()
```

### Aggregation
MongoDB的聚合方法
objects 方法
- count() 返回QuerySet() 数目
- sun('quantitu') 求和
- average() 求平均



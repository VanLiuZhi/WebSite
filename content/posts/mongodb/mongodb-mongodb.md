---
weight: 9900
title: "MongoDB NoSQL 数据库"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "MongoDB NoSQL 数据库"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [DataBase, Note]
categories: [Middleware(中间件)]

lightgallery: true

toc:
  auto: false
---

MongoDB是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。它支持的数据结构非常松散，是类似json的bson格式，因此可以存储比较复杂的数据类型。Mongo最大的特点是它支持的查询语言非常强大，其语法有点类似于面向对象的查询语言，几乎可以实现类似关系数据库单表查询的绝大部分功能，而且还支持对数据建立索引。

<!-- more -->

# MongoDB

![image](/images/mongodb/MongoDB.png)

## ObjectID字段

`ObjectId`构成我们使用`MySQL`等关系型数据库时，主键都是设置成自增的。但在分布式环境下，这种方法就不可行了，会产生冲突。为此，`MongoDB`采用了一个称之为`ObjectId`的类型来做主键。`ObjectId`是一个12字节的`BSON`类型字符串。按照字节顺序，依次代表：

- 4字节：UNIX时间戳
- 3字节：表示运行MongoDB的机器
- 2字节：表示生成此_id的进程
- 3字节：由一个随机数开始的计数器生成的值

`MongoDB`对`ObjectId`对象提供了`getTimestamp()`方法来获取`ObjectId`的时间。

这里不使用自增长`id`主要是因为`MongoDB`是分布式数据库，在并发插入的时候需要维护全局的唯一ID，传统的像MySQL是单机版的，使用自增长ID可以了，处理多条数据插入加锁就行了，虽然MySQL也可以部署集群，但是这种情况ID就没用了，需要自定义全局唯一字段。

`ObjectID`是字段类型，通常来说它是按照默认规则来生产的，文档中的其它字段也可以使用这种字段类型。


## 查询

- 查询全部：`db.document.find({})`
- 查询字段是对象的：`db.document.find({'id_card.idcard_type': '身份证'})`

### projection
该参数指明要显示的字段或者要隐藏的字段
- `db.document.find({}, {'name':1})`  返回结果只显示name字段
- `db.document.find({}, {'name':0})`  返回结果把name字段隐藏了，其它展示出来

## 内嵌文档

见文档

## 数据库引用

分为手动引用和DBRefs，手动引用就是自己建立关系，然后查多次或关联查询。

使用方法：
- $ref：集合名称
- $id：引用的id
- $db:数据库名称，可选参数

产品document中引用attr_data，attr_data是产品属性document

```py
from collections import namedtuple
Point = namedtuple('Point', ['x', 'y'])
p = Point(11, 22)
x, y = p
print(x, y)
```

```js
{   
    "_id":ObjectId("53402597d852426020000002"),
    "product_name": "卫龙辣条",
    "attr_data": {
        "$ref": "product_attr",
        "$id": ObjectId("534009e4d852427820000002"),
        "$db": "testdata"
    }
}
```

product_attr document

```json
{
   "_id" : ObjectId("534009e4d852427820000002"),
   "size": "大",
   "weight": "100g"
}
```

查询
```sh
var product = db.products.findOne({"product_name":"卫龙辣条"})
var dbRef = product.attr_data
db[dbRef.$ref].findOne({"_id":(dbRef.$id)})
结果：
{
   "_id" : ObjectId("534009e4d852427820000002"),
   "size": "大",
   "weight": "100g"
}
```

## 原子操作

mongodb不支持事务，所以，在你的项目中应用时，要注意这点。无论什么设计，都不要要求mongodb保证数据的完整性。

但是mongodb提供了许多原子操作，比如文档的保存，修改，删除等，都是原子操作。

所谓原子操作就是要么这个文档保存到Mongodb，要么没有保存到Mongodb，不会出现查询到的文档没有保存完整的情况。

### findAndModify方法
该方法将查询一些结果，如果查询到，执行更新。这些语句都是写在一个查询中的，并且使用对应的原子操作方法，让整个findAndModify实现原子性操作。
如果分开操作，就是先查询，再修改，在这两个操作之间，如果有人购买了产品，导致库存不足，那么修改操作就会导致数据库数据一致性问题。
```
book = {
          _id: 123456789,
          title: "MongoDB: The Definitive Guide",
          author: [ "Kristina Chodorow", "Mike Dirolf" ],
          published_date: ISODate("2010-09-24"),
          pages: 216,
          language: "English",
          publisher_id: "oreilly",
          available: 3,
          checkout: [ { by: "joe", date: ISODate("2012-10-15") } ]
        }
```
对于上述的book模型，available是我们的判别标志，当它大于0的时候，说明是可以操作的(比如借书)，这部分就是查询，查询成功了，就可以去执行要更新的操作。
db.document.findAndModify({
    query: {},
    update: {}
})
更新操作应该使用原子操作命令

 $set:用来指定一个键并更新键值，若键不存在并创建。


{ $set : { field : value } }
  $unset:用来删除一个键


{ $unset : { field : 1} }
  $inc:可以对文档的某个值为数字型的键进行增减操作 


{ $inc : { field : value } }
  $push：把value追加到field里面去，field一定要是数组类型才行，如果field不存在，会新增一个数据类型加进去。


{ $push : { field : value } }
  $pushAll:同$push,只是一次可以追加到多个值到一个数组字段内。


{ $pushAll : { field : value_array } }
  $pull:从数组field内删除一个等于value值


{ $pull : { field : _value } }
  $addToSet：增加一个值到数组内，而且只有当这个值不在数组内才增加。
  $pop：删除数组的第一个或最后一个元素
{ $pop : { field : 1 } }
  $rename：修改字段名称
{ $rename : { old_field_name : new_field_name } }
  $bit：位操作，integer类型
{$bit : { field : {and : 5}}}
  偏移操作符
> t.find() { "_id" : ObjectId("4b97e62bf1d8c7152c9ccb74"), "title" : "ABC", "comments" : [ { "by" : "joe", "votes" : 3 }, { "by" : "jane", "votes" : 7 } ] }
 
> t.update( {'comments.by':'joe'}, {$inc:{'comments.$.votes':1}}, false, true )
 
> t.find() { "_id" : ObjectId("4b97e62bf1d8c7152c9ccb74"), "title" : "ABC", "comments" : [ { "by" : "joe", "votes" : 4 }, { "by" : "jane", "votes" : 7 } ] }


## 索引

MongoDB通过索引加快查询速度，索引运行在内存中，数据库的操作也需要对索引进行操作，索引超过内存的情况，将会删除一些索引。最好只对大数据文档创建索引。

索引不能被以下的查询使用：
- 正则表达式及非操作符，如 $nin, $not, 等
- 算术运算符，如 $mod, 等
- $where 子句

所以，检测你的语句是否使用索引是一个好的习惯，可以用explain来查看。

### 最大范围

- 集合中索引不能超过64个
- 索引名的长度不能超过128个字符
- 一个复合索引最多可以有31个字段



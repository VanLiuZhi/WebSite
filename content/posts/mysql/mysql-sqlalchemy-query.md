---
weight: 1000
title: "sqlAlchemy-query 常见操作总结"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "sqlAlchemy-query 常见操作总结"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [DataBase, Note, Python]
categories: [DataBase]

lightgallery: true

toc:
  auto: false
---

MySQL 的 Python 版 ORM

<!-- more -->

# sqlAlchemy 数据库操作

sqlAlchemy提供了数据库的很多封装

## 模型创建

模型需要继承 `db.Model` ，db.Colum类描述table的字段，`__tablename__` 描述table在数据库中的名称，不写默认为类名。
`__table_args__` 创建数据库是时附加语句，有点像sql脚本的语句，下面的 `db.Index('idx_commit_item', target_id, target_kind, user_id)` 翻译成sql语句就是 `create index idx_commit_item on collect_items (target_id, target_kind, user_id);`

```python
class CommentItem(db.Model):
    __tablename__ = 'comment_items'
    user_id = db.Column(db.Integer)
    target_id = db.Column(db.Integer)
    target_kind = db.Column(db.Integer)

    __table_args__ = (
        db.Index('idx_commit_item', target_id, target_kind, user_id),
    )
```

`__abstract__ = True` 模型设置了该值，说明该模型是抽象基类，不会被创建数据库
`extend_existing = True` 重定义表结构

## 基本用法

filter 和 filter_by 的区别：

`q = Moedl.query.filter_by(id=1)`
`q = Model.query.filter(Model.id == 1)`

一般来说，查询单体结果，使用 `filter_by`，而 `filter` 可以配合复杂的查询来使用

`q = Model.query.with_entities(Model.id, Model.name, Model.age).filter(Model.state == '1')`

为了连表查询，使用 `db.session.query(Model, User)` 的形式，和 `Model.query` 的形式接近，就是需要把查询模型写入query中，其它关于filter或with_entities的方法都是类似的。

{% blockquote %}
连表查询可能会伴随重复的数据，需要注意
{% endblockquote %}

## 查询集的操作

对于 `q = Model.query.filter(Model.id == 1)` 是还没有操作数据库的，执行 `q.all()` 拿到所有的查询结果，`q.first()` 获取第一个，类似的方法可以参考文档。

## CRUD

C是指create新增，R是指retrieve检索，U是指update更改，D是指delete删除，一般用的最多的就是查询操作，其它操作基本使用如下：

C(create新增)：
```python
db.session.add(self) # self 为模型对象的实例
db.session.commit() # 修改数据库操作记得提交事务
```

U(update更改)：
```python
kwargs = {} # 需要修改的数据字典
for attr, value in kwargs.items():
        setattr(self, attr, value) # 将值修改到查询实例 self 上
    db.session.add(self) # self 为模型对象的实例
    db.session.commit() # 修改数据库操作记得提交事务
```

D(delete删除)：
```python
db.session.delete(self)
db.session.commit()
```

## get

查询操作，返回查询结果，参数必须是主键
`model = Model.query.get(2)`

## count

对查询集使用，返回查询数目合计

## filter 查询条件

1. notin_ 和 in_
q = Model.query.filter(Model.id.in_([1, 2, 3]))

2. or_ 和 and_
q = Model.query.filter(or_(Model.id == 1, Model.id == 3))

3. like
q = Model.query.filter(Model.name.like('%' + 'python' + '%'))

4. between
q = Model.query.filter(Model.age.between(10, 18))

## order_by

对查询集来使用，`q = Model.query.filter(Model.id.in_([1, 2, 3])).order_by(Post.id.desc())`
配合参数的形式
```python
eval('db.%s(func.length(MemberIntegralSetting.%s))' % (sortOrder, sort)),
eval('MemberIntegralSetting.%s.%s()' % (sort, sortOrder))
```
转换字符串类型，或者用func函数
```python
upgrade_value_order_by = '(MemberCardSetting.upgrade_value + 0)'
eval('%s.%s()' % (upgrade_value_order_by, sortOrder))
```

## offset 和 limit

对查询集取范围，offset决定了起始位，比如User有10条数据，`User.query.offset(2).all()` 返回 id是3到10的数据。
limit截取多少，`User.query.offset(2).limit(3).all()` 表示从id是3的数6条数据，即返回3到8的数据。

对于查询集可以使用切片的方式，但是这将查询所有的数据利用切片来获取，和原生语句有效率上的区别的，需要注意。

## with_entities

用于直接获取查询字段的值（不然会查出所有字段，浪费资源），很方便，配合fun函数等，可以直接转换特定对象

需要用到的函数先导入
`from sqlalchemy import or_, func, extract, and_, cast, FLOAT, CHAR`

例子：

先定义一个集合
```python
with_entities_field = {
    func.count(User.id).label('sum_count'), # 计算User的总数，并添加新的标签sum_count
    func.sum(User.recharge_value).label('sum_recharge_value'), # 求和
    User.type # 通常情况
    }
``` 

```python
user = User.query.with_entities(with_entities_field).filter(**kwargs).all()
```

除了func函数，还有cast函数，用来转换类型

1. 整形数据转换成字符串
`cast(FoodOrderInfo.product_count, CHAR).label('product_count')`
2. 时间对象转换成字符串
`cast(FoodOrderInfo.create_time, CHAR).label('create_time')` 

## subquery

```python
merchant_info_subquery = db.session.query(
            MerchantInfo.merchant_code,
            MerchantInfo.merchant_name
        ).join(
            SysOrg, and_(
                SysOrg.org_code.like(org_code + '%'),
                SysOrg.org_id == MerchantInfo.org_id
            )
        ).group_by(MerchantInfo.merchant_code).subquery()
```

## 批量插入（非orm方式）

```py
# MemberCouponRecord 为要操作的表名
db.session.execute(
    MemberCouponRecord.__table__.insert(),
    [{'coupon_id': 'NO0001', 'member_id': 0002, 'update_time': get_current_time(), 'create_by': get_op_user_name()} for item in item_data]
)
db.session.commit()
```
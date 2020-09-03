---
weight: 1
title: "Django 框架学习笔记"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Django 框架学习笔记"
resources:
- name: "base-image"
  src: "base-image.jpg"

tags: [Python, Framework, Note]
categories: [Python编程]

lightgallery: true

toc:
  auto: false
---

django学习笔记 {% github_emoji tada %} {% github_emoji tada %}

对文档阅读的大致补充，框架拥有功能的概述，详细内容查阅文档。

在使用前后端分离的项目中，模版相关的内容基本没什么用了，对于Django如果不是快速开发的应用，基本也不用它提供的功能（认证，表单等），使用较多的有中间件，URL到视图响应，ORM，个人认为扩展ORM开发一套自己的框架很有实战意义，你可以做一个自己风格的ORM，实现像只完成模型定义，就拥有对模型进行增加(Create)、读取查询(Retrieve)、更新(Update)和删除(Delete)的能力。

如此，你便迭代出一个非常适合自己的框架，相信大公司都会有自己的一套迭代框架，用于快速开发产品。

<!-- more -->

# Django 相关

![image](/images/Python/Django.png)

## session

session是一种常用的web技术，在Django框架中很容易去使用它。

### session 概念 

大多数的应用都是用 `Cookie` 来实现 `Session` 跟踪的，第一次创建 `Session` 的时候，服务端会在HTTP协议中告诉客户端，需要在 Cookie里面记录一个 `Session ID`，以后每次请求把这个会话ID发送到服务器，我就知道你是谁了。如果客户端的浏览器禁用了Cookie怎么办？一般这种情况下，会使用一种叫做 `URL重写` 的技术来进行会话跟踪，即每次HTTP交互，URL后面都会被附加上一个诸如 `sid=xxxxx` 这样的参数，服务端据此来识别用户。

网站保存登录账号和密码是由本地的Cookie来实现的。

{% blockquote %}
关于缓存：为了实现性能，缓存还是有必要的，不过先做到数据库的实现。session还可以基于文件来实现
{% endblockquote %}

### 工作流程

session是要浏览器这边配合Cookie来实现的，所以浏览器不能禁用cookie：

1. 当用户来访问服务端时,服务端生成一个随机字符串；
2. 当用户登录成功后 把 {sessionID :随机字符串} 组织成键值对 加到 cookie里发送给用户；
3. 服务器以发送给客户端 cookie中的随机字符串做键，用户信息做值，保存用户信息；

### 代码流程（在默认配置下）

在代码上，我们直接 `request.session['name'] = "my name"` 这一步执行了，就是使用随机字符串，创建了session保存到数据库，然后把 `session_id`（随机字符串）放在cookie里面给到浏览器，浏览器就设置了cookie，下次浏览器就会在请求里面cookie带上这个id，框架流程上中间件会拿出请求体的cookie查询数据库，并将session对象赋值给 `request.session`。

### session序列化(框架文档有讲解)

session的数据会被序列化保存在数据库中，默认是json，一般不需要改，由于是json，所以数据创建的键最好是字符串，数据要能被json编码，你不能直接把一个对象设置在session的键值对当中。

如果想保存更高级的格式，就需要自己实现序列化程序。（从数据库的数据来看，存储的并不是是序列字符串，是一定规则化的字母，猜测是为了压缩数据，s.get_decoded()可以得到解码的结果）

### 会话对象准则

使用普通的Python字符串作为字典键 `request.session`。这是一个比硬性规定更重要的惯例。
以下划线开头的会话字典键由Django保留供内部使用。不要request.session用新对象覆盖，也不要访问或设置其属性。像Python字典一样使用它。

### 扩展

session 就是用来在后端，为了给无状态的HTTP协议提供识别（用户识别），扩展它是很重要的。比如我们不依赖cookie，而是每次都传递一个id，后端用这个id自己创建session，然后前端每次请求都带这个ID，这样后端中间件每次都通过ID查询数据库，赋值request.session

扩展依赖于 `SessionStore`

`from django.contrib.sessions.backends.db import SessionStore`

如果 `SESSION_ENGINE` 不是数据库，需要从对应的引擎来引入，可以这样：

```python
from importlib import import_module
from django.conf import settings
SessionStore = import_module(settings.SESSION_ENGINE).SessionStore
```

### 安全

如果您在cookie中设置了 `HttpOnly` 性，那么通过js脚本将无法读取到cookie信息，这样能有效的防止 `XSS` 攻击，具体一点的介绍请google进行搜索

遇到这个问题时：

{% blockquote %}
The request's session was deleted before the request completed. The user may have logged out in a concurrent request, for example.
{% endblockquote %}

该地址有讲解，不过这个问题应该是出现在开发阶段的调试中，如果出现了问题可以清除浏览器数据来解决，一劳永逸的方案（还没看，链接在下面）不一定需要。
[https://stackoverflow.com/questions/42211065/django-memcached-error-the-requests-session-was-deleted-before-the-request-c](https://stackoverflow.com/questions/42211065/django-memcached-error-the-requests-session-was-deleted-before-the-request-c)

## 密码加密

框架提供了密码加密功能，该部分讲解了密码如何存储，密码升级，密码验证，管理密码。

密码存储暂时没看。

密码升级有从下一版本升级到新版本的时候，使用新的算法，和对所有需要升级的一次处理，具体参考文档。

除了框架的自己提供的，还可以使用自己编写的算法进行加密

手动管理密码：包括几个函数，对密码进行加密得到加密的结果，这个用来保存在数据库，验证密码，把明文密码和数据库存储的加密密码进行验证，返回布尔值。

密码验证：控制用户输入的密码，避免太简单，例如用户的密码输入6为，验证规则是9位那么验证不通过。

配置文件：

```python
# 密码加密使用的算法
# 列表的第一个元素 (即settings.PASSWORD_HASHERS[0]) 会用于储存密码，
# 所有其它元素都是用于验证的哈希值，它们可以用于检查现有的密码。
PASSWORD_HASHERS = [
    'django.contrib.auth.hashers.PBKDF2PasswordHasher',
    'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',
    'django.contrib.auth.hashers.Argon2PasswordHasher',
    'django.contrib.auth.hashers.BCryptSHA256PasswordHasher',
    'django.contrib.auth.hashers.BCryptPasswordHasher',
    'django.contrib.auth.hashers.SHA1PasswordHasher',
    'django.contrib.auth.hashers.MD5PasswordHasher',
    'django.contrib.auth.hashers.UnsaltedSHA1PasswordHasher',
    'django.contrib.auth.hashers.UnsaltedMD5PasswordHasher',
    'django.contrib.auth.hashers.CryptPasswordHasher',
]
```

## 中间件
高版本需要继承MiddlewareMixin，低版本不需要。

各种架构中都会用到的技术（有些框架也称为管道，httphandle）。用户发起的请求会依次经过所有的中间件。由于中间件也用来处理django的内部的东西，所以自己添加的中间件一般写在系统中间件后面，除非你对流程很了解，想在框架某个流程时插入某些东西。

## 模型

- 每个模型都是django.db.models.Model 的一个Python 子类。模型的每个属性都表示为数据库中的一个字段。
- 每个字段都被指定成一个类属性，例如Field类。每个属性映射到一个数据库的列。
- 查阅或看源代码，可以从模型类上获取到很多内容。

### 字段参数

**question_text = models.CharField(max_length=200,blank=true,null=true)**

- `blank，null` 都是该字段可以为空，blank是在admin中可以为空，要在表中可以为空，设置null
- `null = Ture` 指定空，blank = Ture 允许填
- `choices` 这个字段参数设置该字段内容是选择列表
- `primary_key` 该参数为Ture  指定其为主键字段，不写，自动添加一个IntegerField字段为主键
- `db_index` 为此字段创建索引
- `editable` 设置为False  这个字段将不会出现在 admin 或者其他 ModelForm.，会跳过模型验证
- `unique` 为Ture，这个字段在表中必须有唯一值，注意当设置 unique 为True 时，你不需要再指定 db_index，因unique 本身就意味着一个索引的创建

verbose_name（字段的自述名）

`Field.verbose_name` 一个字段的可读性更高的名称。如果用户没有设定冗余名称字段，Django会自动将该字段属性名中的下划线转换为空格，并用它来创建冗余名称。可以参照 `Verbose field names`。

`元参数` permissions 是用来这种模型的权限的，default_permissions 是设置默认权限的，比如add，change，即模型是否能添加，改变。

{% blockquote %}
由于Django 查询语法的工作方式，字段名称中连续的下划线不能超过一个。
django不允许重写字段。
{% endblockquote %}

### 模型的属性

`objects` 模型最重要的属性是Manager。它是Django 模型进行数据库查询操作的接口，并用于从数据库获取实例。如果没有自定义Manager，则默认的名称为objects。Managers 只能通过模型类访问，而不能通过模型实例访问。

Manager 称为管理器 ，它是一个类，该类下面有多个方法，比如get，all，filter 方法，它们都属于这个管理器，你也可以重写一个新的管理器，实现新的查询方法。通常的 Objects.get()，就是指向该类下的方法。

### 类的扩展

创建一个可公共使用的模型，他的字段将被其他模型包含，这个模型称为：`抽象基类`。

当你想将一些共有信息放进其它一些model的时候，抽象化类是十分有用的。你编写完基类之后，在 `Meta` 类中设   `abstract=True`，这个模型就不会被用来创建任何数据表。取而代之的是，当它被用来作为一个其他model的基类时，它的字段将被加入那些子类中。如果抽象基类和它的子类有相同的字段名，那么将会出现 `error（并且Django将抛出一个exception）`。

一个例子：

```python
from django.db import models

class CommonInfo(models.Model):
    name = models.CharField(max_length=100)
    age = models.PositiveIntegerField()

    class Meta:
        abstract = True

class Student(CommonInfo):
    home_group = models.CharField(max_length=5)

```

Student 模型将有三个项：name，age 和 home_group。CommonInfo 模型无法像一般的Django模型一样使用，因为它是一个抽象基类。它无法生成一张数据表或者拥有一个管理器，并且不能实例化或者直接储存。

许多应用场景下, 这种类型的模型继承恰好是你想要的。它提供一种在 Python 语言层级上提取公共信息的方式，同时在数据库层级上，每个子类各自仍然只创建一个数据库表。

`多表继承`

这是 Django 支持的第二种继承方式。使用这种继承方式时，每一个层级下的每个 model 都是一个真正意义上完整的 model 。每个 model 都有专属的数据表，都可以查询和创建数据表。继承关系在子 model 和它的每个父类之间都添加一个链接 (通过一个自动创建的 OneToOneField来实现)。 

例如：

```python
from django.db import models

class Place(models.Model):
    name = models.CharField(max_length=50)
    address = models.CharField(max_length=80)

class Restaurant(Place):
    serves_hot_dogs = models.BooleanField(default=False)
    serves_pizza = models.BooleanField(default=False)

```

Place里面的所有字段在 Restaurant中也是有效的，只不过数据保存在另外一张数据表当中。所以下面两个语句都是可以运行的：

```sh
>>> Place.objects.filter(name="Bob's Cafe")
>>> Restaurant.objects.filter(name="Bob's Cafe")
```

`代理继承`

使用多表继承时，model 的每个子类都会创建一张新数据表，通常情况下，这正是我们想要的操作。这是因为子类需要一个空间来存储不包含在基类中的字段数据。但有时，你可能只想更改 model 在 Python 层的行为实现。比如：更改默认的 manager ，或是添加一个新方法。

```python
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)

class MyPerson(Person):
    class Meta:
        proxy = True

    def do_something(self):
        # ...
        pass

```

它们都操作同一个数据表，设置 proxy = Ture 实现代理，为Person 模型添加了一个方法
你可以在代理中重写方法，或者改变某一字段的排序，不会对原始模型产生影响。比如你用原模型
查询a字段是无序的，你在代理中对a字段元属性进行了排序，用代理模型去查询就是有序的。

## 模型对应关系

框架实现了大部分字段了，文件，图片都有，关于文件如果用到可以仔细看看，另外有模型API，方便获取对象的信息，这对于开发很有用。

### 一对一和一对多

一对多比较了解了，被to的模型，通常会被多个表to。如果字段是外键，并且这个外键只被这个字段to那么这个时候就可以用一对一了，就是这个被to的模型，一张表只被另一张表to（不知道再被另一张表to的时候会发生什么）。通常用来做详情或扩展，比如用户的扩展，那么扩展内容可以是一个新的模型，因为这些内容是和用户相关的，所以应该一条详情对应一个用户，就可以用一对一。

使用一对一模型，在查询的时候，这个被关联的字段可以互相取到双方的值。

如 `user` 有 `detail` 对到模型 `detail`：

```python
u = user.objects.filter().first()
u.detail.msg
# 反向查询
d = detail.objects.filter().first()
d.user.name
# 一对多，上面的详情变成了class
u = user.objects.filter().first()
u.class.msg
# 反向查询
c = class.objects.filter().first()
c.user_set.all()  # 返回查询集
```

反向查询如果设置了 `related_name`，比如 `class_user`，可以这样 `c.class_user`，默认用 `表名_set`。

### 多对多

一般通过外键，新建一个表，把两张表联系起来，实现多对多。或者在其中一个模型里面建立多对多字段。
此时进行正向查询，不会像一对多那样得到外键id或对象，应该 `.all()` 得到查询集（一对多方向才得到查询集），反向查询也是得到查询集，反向查询也是可以用 `related_name`。

### 什么时候用多对多

对于一对多来说，被to的可以说是独立的，它可以不依赖其它模型，比如学生和班级，班级就是班级，而学生要和班级建立关系就用一对多，班级被to了，才会通过反向查询得到所有的学生。

对于上诉情况也会有特殊情况，上诉情况比较实用于中学，中学就是一个学生属于一个班级，如果在大学，我们进行选课，那么课程会是一个模型，一个学生会选择多个课程，这个时候用多对多就比较合适了，当然也可以通过第三张表将课程和学生建立关系，用一对多来实现。

被用做多对多的字段，创建对象的时候，如何赋值，add, set, remove, clear。

## 查询与数据库

框架ORM查询补充

### 数据迁移与修改模型

迁移功能非常强大，可以让你在开发过程中不断修改你的模型而不用删除数据库或者表然后再重新生成一个新的 —— 它专注于升级你的数据库且不丢失数据。实现模型变更的三个步骤：

1. 修改你的模型（在models.py文件中）
2. 运行python manage.py makemigrations ，为这些修改创建迁移文件
3. 运行python manage.py migrate ，将这些改变更新到数据库中

将生成和应用迁移文件的命令分成几个命令来执行，是因为你可能需要将迁移文件提交到你的版本控制系统中并跟随你的应用一起变化；这样做不仅可以使开发变得更加简单，而且对其他开发者以及上线生产非常有用。

### 跨关联关系的查询

可以像这样，只需使用关联的模型字段的名称，并使用双下划线分隔，直至你想要的字段

`dquery = DrawMoneyRecord.objects.filter(drawuser__district__name__icontains='西山片区')`

发现当关联的字段有null的时候，查询不到结果。比如模型DMR有三张表，三张都有drawuser，到了drawuser外键对应的模型User的时候，有两张User的表，其中一张district字段为null，此时查询失效，需保证有关联的都不能为null。

它还可以反向工作。若要引用一个“反向”的关系，只需要使用该模型的小写的名称。

比如 Order 和  OrderItem ，它们的对应关系为：

`order = models.ForeignKey(to=Order, verbose_name=_('订单'), editable=False)`

想要从Order来查：`Order.objects.filter(orderitem__name__icontains='小')`，便可以利用orderitem来查出order

反向查询，关系是从右看到左：

`AgencyOrder.objects.filter(agencyorderitem__orderitem__order_id__in=order_ids)`

上面这个order_ids可以得到order，向上得到orderitem...

order__bigorder__date

### SQL 优化

prefetch_related  select_related 

都是针对表有关联的，如果不用，则拿到的只是外键的id，如果使用，则一次就把外键对象拿到。

### 打印 SQL 语句

1. 确保 `django.core.context_processors.debug` 在 `CONTEXT_PROCESSORS` 中
2. DEBUG = True

代码如下：

```python
from django.db import connection
# 这里是查询
MyModel.objects.filter(name="my name")
print connection.queries
# 或者
from django.db import connection
print MyModel.objects.filter(name="my name").query
```

### save 方法

一般在得到查询集后，只有是去创建的情况才使用 `queryset.save()`，对于数据的修改，使用 `quseryset.save(update_fields=['fields'])`。

### 查询对象 F，Q

F用来做运算，加减乘除。Q做复杂查询。

### 查询对象Case

```py
equipments = equipments.order_by(
    Case(When(id__in=[item.get('eqid', 0) for item in equipment_ids], then=0), default=1), '-ismonitor', 'id')
```
equipments中，先把id为特定条件的数据放到最前面，然后后面的数据按照`'-ismonitor', 'id'`的规则进行排序

### 查询集

查询集，就是查询结果的集合。

Blog.objects.all() 返回包含数据库中所有 Blog 对象的一个查询集。

- filter(**kwargs) 返回一个新的查询集，它包含满足查询参数的对象。
- exclude(**kwargs) 返回一个新的查询集，它包含不满足查询参数的对象。

通过 get 获取一个单一的对象 `one_entry = Entry.objects.get(pk=1)`

queryset是查询集，就是传到服务器上的url里面的查询内容。Django会对查询返回的结果集QuerySet进行缓存，这是为了提高查询效率。也就是说，在你创建一个QuerySet对象的时候，Django并不会立即向数据库发出查询命令，只有在你需要用到这个QuerySet的时候才会这样做。

缓存和查询集：

每个查询集都包含一个缓存来最小化对数据库的访问。理解它是如何工作的将让你编写最高效的代码。
在一个新创建的查询集中，缓存为空。首次对查询集进行求值 —— 同时发生数据库查询 ——Django 将保存查询的结果到查询集的缓存中并返回明确请求的结果（例如，如果正在迭代查询集，则返回下一个结果）。接下来对该查询集 的求值将重用缓存的结果。
请牢记这个缓存行为，因为对查询集使用不当的话，它会坑你的。例如，下面的语句创建两个查询集，对它们求值，然后扔掉它们：

```sh
>>> print([e.headline for e in Entry.objects.all()])
>>> print([e.pub_date for e in Entry.objects.all()])
```

这意味着相同的数据库查询将执行两次，显然倍增了你的数据库负载。同时，还有可能两个结果列表并不包含相同的数据库记录，因为在两次请求期间有可能有Entry被添加进来或删除掉。为了避免这个问题，只需保存查询集并重新使用它：

```sh
>>> queryset = Entry.objects.all()
>>> print([p.headline for p in queryset]) # Evaluate the query set.
>>> print([p.pub_date for p in queryset]) # Re-use the cache from the evaluation.
```

### 查询集方法

- order_by 排序
- pk__in  当需要取多个结果的时候，比如id=1,2,3这三条数据，`models.object.filter(pk__in = ids)`，ids=[1,2,3]
- select_related()函数 在一对一和外键中使用，目的：减少查询次数

关联查询例子：

Person(人)，字段中有一个居住地living，living外键到City，City 中有 province(省)，如何直接得到省的数据？

```py
p=Person.object.select_related('living').get(name="小明")
p.living.province
```

如果省是其它的外键，比如 province外键到 Province类，查询集要用 `' __ '`，`p=Person.object.select_related('living__province').get(name="小明")`。

常用方法：

| Command         	            |  Description                     
| ----------------------------- |:-------------------------------: 
| __exact		                | 精确等于 like 'aaa'   
| __iexact                      |精确等于 忽略大小写 ilike 'aaa'
| __contains                    |包含 like '%aaa%'
| __icontains                   |包含 忽略大小写 ilike '%aaa%'，但是对于sqlite来说，contains的作用效果等同于icontains。
| __gt                          |大于
| __gte                         |大于等于
| __lt                          |小于
| __lte                         |小于等于
| __in                          |存在于一个list范围内
| __startswith                  |以...开头
| __istartswith                 |以...开头 忽略大小写
| __endswith                    |以...结尾
| __iendswith                   |以...结尾，忽略大小写
| __range                       |在...范围内
| __year                        |日期字段的年份
| __month                       |日期字段的月份
| __day                         | 日期字段的日
| __isnull=True/False           |
| __isnull=True                 |与 __exact=None的区别

例子：
类是 Author 字段 username，password 

```py
Author.objects.filter(username__exact=username)   #精准查询
k=Author.objects.filter(username__exact=username)
obj=Author.objects.get(username__exact=username)   #get 取得的是字段为username
```

匹配的对象：在使用update的时候，需要注意，查询集才有这个方法，查询集实例没有，查询集有5个，update可以一次更新5张表的数据。

### 分页实现

两个参数 page，limit 每次都传这两个参数，决定了数据的截取位置和截取多少。

`query[ (page-1)*limit : page*limit ]`

### 其它

- 对于模型的新创建的实例，直接save()就行了，不能用 `save(update_fields=[])`，本来就没有字段，所以不能用。如果是QS的元素，考虑使用 `save(update_fields=[])`，它只更新特定的字段。 
- 对于QS，可以使用QS.update(field=value)，批量跟新。

`django queryset` 的 `values` 和 `values_list(values_list('id', flat=true))`

这两个函数可以得到特定字段的值，有些字段是外键，我们在表示的时候，需要的是外键所对应的对象，利用这个外键id,查到数据后，将查到的对象添加到刚才的列表中去。

- 去除重复：distinct

distinct()可以对查询集进行去重复，比如querset.distinct(),  得到的查询集元素都是唯一的对象。（有些时候，我们用id__in去得到的数据会有好几个，比如一个模型的两张表字段外键都是对到同一个，这个时候id__in 就会得到两个查询集对象，但是这个去重一般是在想处理相同字段的时候使用）当我们想去除字段相同的数据的时候，`querset.values(fields).distinct().order_by('fields')`, 这个操作会返回dict, 这里需要排序的原因是如果不排序。在执行distinct的时候，使用默认排序id, 两列数据id是不一样的，只有我们想去重的字段是一样的，所以要以想去重的字段为准，执行一次排序。

- 在做查询的时候，filter的参数如果是None，就是把模型中该字段为null的查出来

- sql：EXISTS用于检查子查询是否至少会返回一行数据，该子查询实际上并不返回任何数据，而是返回值True或False。EXISTS 指定一个子查询，检测 `行` 的存在。django也可以用这个方法，对于queryset.exists()即可

- aggregate和annotate方法使用详解与示例

https://blog.csdn.net/weixin_42134789/article/details/84567365

## 事务

默认情况下，django的sql执行都是在事务中进行的，MySQL一般都是开启默认事务提交的，正常情况下，sql语句必须要开启事务，如何才能提交事务。而django的sql的执行会默认在事务中，避免了脏读。

在Django中使用事务：`@transaction.atomic` 该装饰器将装饰内容由事务来处理，也就是把多个sql执行封装在一个事务里面。

## 命令

项目目录下的 `manage.py` 接受输入参数，执行对应命令。

- python manage.py validate 验证模型的有效性
- python manage.py migrate  创建数据库表
- python manage.py makemigrations polls  关联应用，激活模型（现在Django知道要包含polls应用。 可以运行这个命令）
- sqlmigrate    展示迁移的SQL语句

如果在后面的开发中需要更改模型，先在代码中加入新的字段，然后执行 `迁移命令`，再执行数据库创建命令。
- python manage.py makemigrations （根据模型给应用生成迁移脚本）
- python manage.py migrate
- python mange.py shell 进入交互式shell

各种修改表后，可能会导致表无法创建（历史原因，可能），执行命令来同步表（遇到场景，提示字段名字重复，创建了字段后，又修改，删除migrations，创建），可以执行以下命令：

```sh
python manage.py migrate myapp --fake
```

### 创建管理员

首先，我们需要创建一个能够登录管理站点的用户。 

运行如下命令：

1. `$ python manage.py createsuperuser` 键入你想要使用的用户名，然后按下回车键

2. `Username: admin` 然后提示你输入想要使用的邮件地址

3. `Email address: admin@example.com` 你将被要求输入你的密码两次，第二次输入是确认密码

```
Password: **********
Password (again): *********
Superuser created successfully.
```

## 配置

配置修改setting.py文件

### Django设置TIME_ZONE

Django默认的timezone是 `TIME_ZONE = 'America/Chicago'`，现在要改成我们中国的时区 只需编辑settings.py文件，把time_zone的值改成TIME_ZONE=即可。

## 模版

模版语法可以快速开发页面，**不过对于前后端分离的项目，基本没什么用了**。

- 使用模板，实现动态的生成HTML。模板包含所需HTML输出的静态部分，以及一些特殊的语法，描述如何将动态内容插入。
- Django 为加载和渲染模板定义了一套标准的API，与具体的后台无关。
- 加载包括根据给定的标识找到模板然后预处理，通常会将它编译好放在内存中。
- 渲染表示使用Context 数据对模板插值并返回生成的字符串。

注意在django工程的设置中，模板设置里面，APP_DIRS设置为True，这样就允许查找应用下的模板。这里的应用包括自己创建和系统的，可以查看设置文件看看该项目有哪些应用。只要在该项目下有过Template文件夹，django就可以找到模板。

模板语言：

变量 {% raw %}{{ }}{% endraw %}，变量的值来自于context中的输出

句点查找，就是  .  查找，比如 {% raw %}{{ f.b }}{% endraw %}，将按照一定顺序的查找规则。
字典，属性，方法调用，列表类型索引。  

系统使用所找到的第一个有效类型，这是一种短路逻辑它可以嵌套，比如 {% raw %}{ { f.a.bv(&nbsp;) }}{% endraw %}，按顺序来，如果a没有找到，就去找bv()。

### Django 模板中的HTML自动转义

当用户输入的信息是一个JS脚本的时候，这个时候浏览器会执行脚本，难免会有漏洞。所以不能进行转义。

可以使用模板过滤器：safe

例子： 

```
hello {{name|safe}}
```

### 自定义标签

我们可以创建一个模板库，里面包含我们自己写的标签和过滤器。在应用目录下创建一个templates文件夹，该文件创建一个文件
`__init__.py`，说明它是一个模块，如何就可以写自定义的 `.py` 标签过滤器。

在模板中 `{% raw %}{% load poll_extras %}{% endraw %}` 便可以将标签载入，load后面的是写的.py。

`{% raw %}{% load %}{% endraw %}` 标签检查 INSTALLED_APPS 中的设置，仅允许加载已安装的Django应用程序中的
模板库。这是一个安全特性。它可以让你在一台电脑上部署很多的模板库的代码，而又不用把它们
暴露给每一个Django安装。

### url 标签

```py
urlpatterns = patterns('',    
    (r'^article$','news_index' ),
)
```

```html
<a href="/article">资讯</a> 
```

通常我们的URL都是硬编码的，在模板里面可能会有多个标签都是一个url，如果你要改变这个url，那么模板里面所有的url都要改变，这时候我们可以使用url标签。

```py
urlpatterns = patterns('',    
    url(r'^article$','news_index' ，name="news_index"),
)
```

```html
<a href="{%url 'news_index'%}">资讯</a>
```

增加一个nema，当你改变原来的url时，模板的地址也会随之改变，在view中使用 `HttpResponseRedirect("/article")`。

使用 `reverse()` 函数 `HttpResponseRedirect(reverse("news_index"))`。

带参数的url 

```py
url(r'^(?P<year>\d{4})/(?P<month>\d{1,2})/$','news_list',name="news_archive" )
```

```html
<a href="{%url 'news_archive' 2010  02%}">2010年02月</a> 
```

或者这样:

```html
<a href="{%url 'news_archive' year=2010  month=02%}">2010年02月</a> 
```

### 模板继承

创建一个基本的骨架模板，将里面的部分内容用其它模板来替换。

```html
{% block title %}
    <ul>
        <li><a href="/">Home</a></li>
        <li><a href="/blog/">Blog</a></li>
    </ul>
{% endblock %}
```

在这个模板中，使用block标签， block 告诉模版引擎：子模版可能会覆盖掉模版中的这些位置。

只是可能替换，不一定非要替换，如果没被替换，输出是原样：

```html
<ul>
    <li><a href="/">Home</a></li>
    <li><a href="/blog/">Blog</a></li>
</ul>
```

在子模板中，同样是block标签。比如

```html
{% block title %}My amazing blog{% endblock %}
```

则原模板在使用的时候，它的title包围的块被替换成 `My amazing blog`，子模板在开头加上extend标签，说明继承关系：

```html
{% extends "base.html" %}
```

如果需要获得父模板中代码块的内容，可以使用 
{% blockquote %}
{{ block.super }} 
{% endblockquote %}
如果只想在上级代码块基础上添加内容，而
不是全部重载，该变量就显得非常有用了。

过滤器可以自己定义，记得在设置中添加路径
自定义的过滤器添加了才有用，继承中没有添加过的过滤器不能使用

### 标签

Django自带了大约24个内置的模版标签。你可以在内置标签参考手册中阅读全部关于它们的内容。

标签在渲染的过程中提供任意的逻辑。

这个定义是刻意模糊。例如，一个标签可以输出内容，作为控制结构，例如“if”语句或“for”循环从数据库中提取内容，甚至可以访问其他的模板标签。Tags是由 `{% raw %}{%   %}{% endraw %}` 来定义的。

### 过滤器

更改变量或标签参数的值

`{% raw %}{{ django|title }}{% endraw %}` 把django变量的内容开头是小写的变成大写

过滤器能够被“串联”。一个过滤器的输出将被应用到下一个。`{% raw %}{{ text|escape|linebreaks }}{% endraw %}` 就是一个常用的过滤器链，它编码文本内容，然后把行打破转成 `<p>` 标签。

一些过滤器带有参数。过滤器的参数看起来像是这样：`{% raw %}{{ bio|truncatewords:30 }}{% endraw %}`。这将显示 bio 变量的前30个词。
过滤器参数包含空格的话，必须被引号包起来；例如，使用逗号和空格去连接一个列表中的元素，你需要使用 `{% raw %}{{ list|join:", " }}{% endraw %}`。

Django提供了大约六十个内置的模版过滤器。你可以在 内置过滤器参考手册中阅读全部关于它们的信息。

### 注释

`{# This is a comment #}` 这样完成类注释，注释的内容不会在模板渲染时输出。

要注意的是，注释不会跨多行比如：

```
This is a {# this is not
            a comment #}
test.
```

这个注释是无效的。

### 其它

ifequal：输出第一个判断为True的值

## 其它功能

- 发送信号：在特点操作完成前后都可以发送信息，比如数据save()
- 聚合内容（RSS ATOM）
- 静态文件收集：方便部署
- 验证器：用在模型字段参数里面，对字段的值进行验证
froms:表单模型，表单相关，验证器可以在表单模型字段里面使用（表单模型不是我常用的东西）
- 日志记录模块

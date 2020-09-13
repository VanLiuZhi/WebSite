---
weight: 1000
title: "python web 开发(一) 后端架构"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "python web 开发(一) 后端架构"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Python, Web, Technology]
categories: [Technology技术]

lightgallery: true

toc:
  auto: false
---

Python web 开发总结篇，主要分析web开发中后端架构的设计，前端工程化，服务部署，测试等和web开发相关的整个流程。做了很多准备工作，编写了一个可以用的项目，主要为演示所用，后面会上传到GitHub上，整体功能就是通过ress获得数据用于展示，也可以当场一个快速上手的开发环境，它将包括很多常见的服务。

在之前差不多就构思好了，这次一次整理发出来，开篇从后端开始。

<!-- more -->

## python 做 web 开发

web 开发在互联网的浪潮中经历了数次变革，各种技术层出不穷，做为一名后辈也许体会不到10年前的程序开发是什么样的，但是总结历史，才能放眼未来，开遍写对 web 开发的发展做个概述吧（就不套用资料了，用自己的话）

初期阶段，设计出了浏览器，当时的浏览器只能查看静态页面，后来为了能让浏览器有一定的交互性，设计了早期的JavaScript语言，我没记错的话，当时应该是叫其它名字。

即使是这样，和现今的浏览器还是有很大的差距的，首先它的页面也是事先写好的，想要动态的数据就需要由程序来生成，各种语言都来搞这个事情，于是提出了CGI这样的接口标准来实现CGI程序与WEB服务器之间的通信，当然现在也不用CGI这么原始的方式(在Python核心编程一书中有将如何用CGI来生成动态页面)，Python 的服务器网关接口称为 WSGI，很多web框架都实现了这个协议，到此解决了动态数据和各个程序之间的问题。

该聊聊前端了，在不断的开发和总结中，提出了一些设计模式，就是MVC，MVP等，个人认为不需要太多的去纠结你的架构属于什么模式，也许你复用了多个模式之间的东西，像Django也不好归属，一般说它是MTV，模型，模板，视图的架构。这些架构模式，不仅适应后端框架，还适应于前端框架，GUI软件等，所以模式真的太多了，有很多公司都不好把自己归在其中，其实好用的架构才是好的。对于前端我比较推崇的是MVVM这样的框架，也就是Vue，React，Angularjs。

python 在 web 开发中虽然占据一席之地，不过由于国内环境，Java是一家独大，这里有很多因素就不展开了。另外必须要承认的是Python做为动态语言，速度上是没法和静态语言相比的，这也Python经常被吐槽的原因，但是我认为这并不是你web应用的主要瓶颈，你需要的是合理的架构，因为在web开发中，速度不是决定一切的关键，IO经常需要等待，速度再快也没用；和开发者也有关，一个普通的开发者用静态语言写了同样的服务，他占着速度的优势也许差的程序跑起来也不会太差，而Python作为静态语言本身在速度上就有欠缺，当然更需要简练的代码。好在Python作为动态语言，有着开发速度快，语言简洁的优点，仍被众多开发者青睐。

## 概述

在文章开始前，先对整个项目做下简介。

后端采用flask框架，用过Django后，又用flask，还是觉得falsk好，方便扩展，精简功能，舍弃不需要的，如果是想定制一套自己的开发框架，推荐用flask。

总结嘛，就是学之所用，整体的技术栈如下：
```m
1. 数据库
    1. MySQL(主数据库)，使用SQLALchemy
    2. Redis(缓存，副数据库)

2. 任务队列
    1. Celery
    2. RabbitMQ
    3. Redis
    
3. 搜索服务
    Elasticsearch
    
4. 容器技术 Docker
    1. docker-compose
    2. Dockerfilea

5. 前端
    1. Vue 生态构建前端工程
    2. elemet-UI 作为主要UI组件
```

前端采用了Vue的生态，也就是说整个项目都是走的接口来和后端交互。前端的项目和后端是分开的。

后面的文章将逐个分析项目内容，从后端到前端，最后到部署，总结一下在此过程中，为什么要选用它，应该怎么去用，以及一些可能会出现的坑。

GitHub地址在最后测试部分。

项目实现的功能为前端展示数据，数据库存储ress抓取到的数据，Redis也会用来存储部分数据。搜索功能使用了Elasticsearch，索引的创建由celery来执行，部署和开发阶段都用到了docker。

## 项目分析

该项目的工程构建，层级不是很多，各个模块归属一个文件夹，其中flask_app为核心部分，包含了flask相关的基础部分，base包含了核心服务的代码，模型都在models中，接口是api_views，celery任务在handler中，其它文件基本也是这样。

对于工程构建，有很多的模式，有些复杂的项目层级关系就很复杂，无论你怎么组织，对于flask来说，你应该注意：

1. 重复引用问题
这个问题在flask中尤为突出，flask就是创建 `app = Flask('flask_app')` ，全局围绕这个app展开，如果你不组织好，很容易因为相互导入app引发问题。

2. app实例的参数，Flask类需要传递一个文件名来创建实例，框架将根据它来组织文件搜索路径，官方的例子实在是过于简单了，我这里用的参数是flask_app，就是所处文件的名称，这里也可以配合 `__file__` 来动态处理。这个路径的错误将导致最大的问题就是静态文件搜索错误，可以通过DEBUG，分析app实例的path是否是正确的路径。

3. 数据库管理工具Migrate，用这个迁移工具的时候，需要把对应的模型先加载，同样因为官方的例子过于简单，在组织工程的时候很容易忽视这个问题。查看源码就知道Migrate能够找到模型是在app的实例创建的命名空间去查找的，开始我一度认为是通过静态文件检查来实现的。本项目在创建app的时候，通过load_models()方法来加载模型。

4. 最后再总结一点，flask整体就是围绕着app来的，报错了，从app实例创建一步一步的去分析，看看源码，很快就能定位问题了。另外像request这样的模块好像没和flask交互就拿到了请求参数，这里是使用了一个本地线程的功能，request通过本地线程做桥梁和app实例来沟通，有很多地方都用到这个本地线程。

### app.py

该文件位于flask_app中，主要是创建app做准备，包括加载配置，注册命令，初始化扩展模块，并创建app实例，以提供外部使用，这是最简单的单例模式了，你的app一定要保证全局唯一，除非你需要新的app实例。

### setting.py

配置文件内容，使用类的模式来组织，Config是基类，包含基本配置参数，日志配置也在此，DevConfig是开发配置，ProdConfig是生产配置，都继承了Config。

在需要用配置的地方，通过 `get_config` 来加载对应环境的配置，通过当前环境变量判断，环境变量设置在 `.evn` 中，通过 `load_env_value` 来加载，如果使用pipenv，或者autoenv这样的虚拟环境，会自己加载 `.env` 的东西。

### auto_app.py

最外层的启动文件，主要就是实现migrate，返回app实例。

## Redis 服务

Redis 在 web 开发中是必备，也可以用 Memcached，通常用做缓存，缓存可以说是web开发中最廉价的解决性能问题的方案了。

一定要记住，无论什么时候，都不要让用户轻易的访问到数据库，能用缓存的尽量用缓存，因为缓存是从内存读取，内存的寻址要快多了，而数据库，要连接服务，然后访问存储设备，然后通过索引等查询技术在文件中找到数据。

本项目的 Redis 用在了数据库和缓存两个地方，数据库用在了模型的字段上，通过一个描述符来处理：

```py
class PropsItem(object):

    def __init__(self, name, default=None, output_filter=None, pre_set=None):
        self.name = name
        self.default = default
        self.output_filter = output_filter
        self.pre_set = pre_set

    def __get__(self, obj, objtype):
        r = obj.get_props_item(self.name, None)
        if r is None:
            return copy.deepcopy(self.default)
        elif self.output_filter:
            return self.output_filter(r)
        else:
            return r

    def __set__(self, obj, value):
        if self.pre_set:
            value = self.pre_set(value)
        obj.set_props_item(self.name, value)

    def __delete__(self, obj):
        obj.delete_props_item(self.name)
```

可以这么用 content = PropsItem('content')，set_props_item方法是Redis操作的封装，在base模块的redis_db中，机制都是自动更新的，对应的键更新。

另外在sqlalchemy的扩展部分，对属于Redis处理的字段做了标注

```py
class BindDBPropertyMixin(object):
    def __init__(cls, name, bases, d):
        super(BindDBPropertyMixin, cls).__init__(name, bases, d)
        db_columns = []
        # 这个d是每个模型类的属性字典，应用启动初始化数据就会执行到这里，这样实现了对每个模型类的定制
        # 下面代码从模型类的属性中取出键值对，v就是类属性的值，
        # 这里判断使用PropsItem实例作为字段的模型将添加_db_columns属性
        for k, v in d.items():
            if isinstance(v, PropsItem):
                # k 为字段名称，v是实例对象
                db_columns.append((k, v.default))
        setattr(cls, '_db_columns', db_columns)
```

通过对象的_db_columns就可以获取当前模型需要被Redis处理的字段，sqlalchemy的初始化在base模块的，base_extend.py 中。

## 缓存服务

缓存要单独说，本项目用到了Redis缓存，数据库缓存，本地缓存。

1. Redis 缓存

Redis缓存比较简单，就是用装饰器，这是很通用的做法，在Python中，有@property，通过改造它，我们实行一个逻辑，把被装饰对象注入到当前实例 `__dict__` 中，这样每次的服务都不会再走代码，Django 提供的属性缓存就是这么做的。

这里实现的逻辑是去Redis中找，能找到就不计算属性，通过键来查找。

```py
def cache(key_pattern, expire=None):
    def deco(f):
        arg_names, varargs, varkw, defaults = inspect.getargspec(f)
        if varargs or varkw:
            raise Exception("do not support varargs")
        gen_key = gen_key_factory(key_pattern, arg_names, defaults)

        @wraps(f)
        def _(*a, **kw):
            key, args = gen_key(*a, **kw)
            if not key:
                return f(*a, **kw)
            force = kw.pop('force', False)
            r = rdb.get(key) if not force else None

            if r is None:
                r = f(*a, **kw)
                if r is not None:
                    if not isinstance(r, BUILTIN_TYPES):
                        r = dumps(r)
                    rdb.set(key, r, expire)
                else:
                    r = dumps(empty)
                    rdb.set(key, r, expire)

            try:
                r = loads(r)
            except (TypeError, UnpicklingError):
                pass
            if isinstance(r, Empty):
                r = None
            return r

        _.original_function = f
        return _

    return deco
```

这都是很标准的写法，利用该装饰器就可以实现基于Redis的缓存了，缓存的使用要注意何时清除缓存，数据被更新了就应该清除缓存。

2. 本地缓存

除了使用Redis缓存，还可以使用本地缓存的办法，这是一种思想的体现，本地缓存就是把数据缓存在当前进程了，这样都不需要通过Redis，速度就更快了，这个的实现也很简单，写一个类把数据都缓存在这个类中。

当然这不是万能的，这相当于自己造轮子了，如果要用集群，相信Redis能提供更好的服务。

3. 数据库缓存

这也是一种缓存方法，相当于对Redis的封装，通过特定的查询接口就可以获取被缓存的查询，不必每次去数据库找，和Redis装饰器相比更加灵活。

这部分的代码比较多，用法就是调用cache方法

```py
@declared_attr
    def cache(cls):
        return Cache(cls, cls.cache_regions, cls.cache_label)
```

## 任务队列

任务队列也是web中常用的技术，一般的web程序都是同步阻塞的IO模型，这表示一个请求如果耗时间很长，将会影响其它用户，所以我们不能把这种耗时长的任务直接在本次请求中执行，应该交给像celery这样的异步任务队列。

虽然有了异步编程，协程等方式不会一直阻塞进程，但是它们都是最大化利用cup的，该耗时的任务还是耗时，使用celery就可以解耦这些耗时的业务，让程序更加清晰，明确。

本项目中，使用RabbitMQ做为消息代理，Redis作为结果存储，配置对应的参数后就可以使用celery了，具体的用法参考[Celery 在 web 开发中的运用](http://www.liuzhidream.com/2018/10/22/Web/Celery/)

## 搜索服务

以前我在写web的时候，对于搜索都是直接从数据库查询的，当数据需要从多张表中获取的时候，比较头疼了，需要把每张表都查一遍，当时也找过一些全局搜索的方案，只是没在项目中用起来。

本项目用了Elasticsearch，简单来说就是查询都是去查由Elasticsearch提供的数据，那这些数据怎么来的？就是事先创建的，有点像数据库的索引，使用Elasticsearch，相当于和数据库分割了，我们不会再去查询数据。

Elasticsearch其实也是一个数据库，它由Java编写，但是它的接口是走的http服务，这使得其它语言来使用它变得非常简单。你可能会疑问不是访问了另一个数据吗？对于速度上会有什么优势吗？其实像MySQL这样的数据库只要使用特定的引擎，查询效率会上升很多，但是为了兼顾存储，一般使用innodb，所以只要针对查询做特别的优化，就可以有不错的速度。

用法参考[elasticsearch 全文搜索解引擎在 web 开发中的运用](http://www.liuzhidream.com/2019/01/11/Web/elasticsearch/)

通过创建类来存储数据

```py
class Item(Document):
    id = Integer()
    title = Text()
    kind = Integer()
    content = Text()
    n_likes = Integer()
    n_collects = Integer()
    n_comments = Integer()
    can_show = Boolean()
    created_at = Date()
    tags = Text(fields={'raw': Keyword()})

    class Index:
        name = 'test'
```

通过add方法，去插入新的索引，这个操作由celery来完成

## API 设计

api设计也是老生常谈的话题，很多公司会遵循通用的方案，比如RESTFulAPI，通过使用规范的方案，便于程序阅读。这里没有使用RESTFulAPI，使用的我比较喜欢的一种风格，就是接口名称写在接口类中，每个接口对应一个方法，通过动态路由来匹配方法。

这样设计了一个可用的API方案，如果你要想像RESTFulAPI，那么规范一下请求，做多层路由。

这里的视图就应该使用类视图了，不要用函数视图，然后对类视图注册，把请求拦截后通过匹配对应的方法来执行。

大概是这样的：

```py
class APIHandelView(MethodView):

    def dispatch_request(self, *args, **kwargs):
        """
        重载 dispatch_request 实现动态接口路由
        """
        api_name = kwargs.get('api_name')
        api_method = getattr(self, api_name, None)
        if api_method is None:
            raise ApiException(errors.api_not_found)
        return api_method()
```

这样的API风格还可以进行后续的扩展，比如请求方法的限制，权限控制等，都是需求上的东西了，这也是使用类视图的好处。

## 总结

基本就用到了这些，都是一个web架构常见的服务，下面讲前端部分，配合后端的接口部分。





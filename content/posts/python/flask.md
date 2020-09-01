---
weight: 1
title: "Flask 框架学习笔记"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Flask 框架学习笔记"
resources:
- name: "base-image"
  src: "base-image.jpg"

tags: [Python, Framework, Note]
categories: [Python编程]

lightgallery: true

toc:
  auto: false
---

Flask框架学习笔记

Flask 也是Python web 开发中很重要的框架，相比Django来说，它体现在一个“微型”上，只提供核心功能，其余功能由模块来扩展。拥有很强大的灵活性，通过核心来不断的扩展实现需求。

<!-- more -->

# Flask

![image](/images/Python/Flask.png)

## 关于扩展

通过安装对应的扩展包，可以扩展框架的很多功能（这些扩展和框架会有结合，有点想开启框架的功能一样），Flask的扩展都暴露在flask.ext命名空间下，你可以在环境中通过pip安装好相应扩展，然后再在程序中导入相应的包即可使用扩展。

在新的版本中，引入flask扩展不能再从flask.ext导入了，直接从安装模块导入，比如 `from flask_sqlalchemy import SQLAlchemy`

## 自定义url转换器

在路由中，使用<>来获取动态参数，默认是字符串类型的，如果想要指定参数类型，需要标记成`<converter:variable_name>` 这样的格式，类似 `<int:quantity>`，使用any可以指定多种路径，类似`<any(a, b:page_name)>`。像any，int做为类型可以自定义，比如定义list类型，代码如下：

```python

from urllib import parse

from flask import Flask
from werkzeug.routing import BaseConverter

app = Flask(__name__)


class ListConverter(BaseConverter):

    def __init__(self, url_map, separator='+'):
        super(ListConverter, self).__init__(url_map)
        self.separator = parse.unquote(separator)  # unquote 对url进行解码

    def to_python(self, value):
        return value.split(self.separator)

    def to_url(self, values):
        return self.separator.join(BaseConverter.to_url(self, value)
                                for value in values)


app.url_map.converters['list'] = ListConverter


@app.route('/list1/<list:page_names>/')
def list1(page_names):
    print(page_names)
    return 'Separator: {} {}'.format('+', page_names)


@app.route('/list2/<list(separator=u"|"):page_names>/')
def list2(page_names):
    return 'Separator: {} {}'.format('|', page_names)


if __name__ == '__main__':
    app.run(host='0.0.0.0', port=9000)

```

## 唯一url

在路由的装饰器中，如果指定了结尾的反斜杠，类似这样的路径 `\page\`，那么浏览器访问地址以反斜杠结尾，或者没有反斜杠都可以访问(访问一个不以反斜杠结尾的url会被重定向到到反斜杠的url上)，如果路由是`\page`，那么浏览器访问以反斜杠结尾会报错。

## 扩展响应

这是一个很常见的需求，对返回对象进行包装，比如返回一个ORM的查询实例，这个是不能被json序列化的，通过扩展返回数据，可以对特定对象进行处理。另一种情况，是重写响应类，`app.response = JSONResponse`

## 静态文件管理

在创建flask实例的时候，通过static_folder修改默认静态文件路径，`Flask(__name__, static_folder='/tmp')`

## 关于视图

通常使用函数视图，但是这样就发挥不出类的作用了，比如继承一个基类，定义一些基础的东西，flask也可以使用基于类的视图

### 标准类视图

标准类视图是继承自 `flask.views.View`，并且在子类中必须实现 `dispatch_request` 方法，这个方法类似于视图函数，也要返回一个基于Response或者其子类的对象。通过 `app.add_url_rule(url_rule, view_func)` 来进行注册

```python

from flask.views import View
class PersonalView(View):
    def dispatch_request(self):
        return "hello"

app.add_url_rule('/users/',view_func=PersonalView.as_view('personalview'))

```


### 基于调度方法的视图

继承自 `flask.views.MethodView`，可以对不同的HTTP方法执行对应的函数，使用方法的小写名。在类视图中定义一个属性叫做decorators，然后存储装饰器。以后每次调用这个类视图的时候，就会执行这个装饰器

```python

from flask import Flask, jsonify
from flask.views import MethodView
from flask import session

app = Flask(__name__)


def login_required(func):
    def wrapper(*args, **kwargs):
        if not session.get("user_id"):
            return 'auth failure'
        return func(*args, **kwargs)

    return wrapper


class UserAPI(MethodView):
    decorators = [login_required]

    def get(self):
        return jsonify({
            'username': 'fake',
            'avatar': 'http://lorempixel.com/100/100/nature/'
        })

    def post(self):
        return 'UNSUPPORTED!'


app.add_url_rule('/user', view_func=UserAPI.as_view('userview'))

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=9000)
        
```

## 使用命令行接口

和Django一样，flask框架也提供了很多命令，flask命令需要添加都环境变量，这个一般在安装后就有了。然后需要设置flask应用的环境变量，可以使用 `export FLASK_APP='app.py path'` 。

也可以自定义命令，比如 `flask run_test` 执行一个测试脚本，代码如下：

```python

@app.cli.command()
def run_test():
    print('this is a test script')

```

## Context

理解请求上下文和应用上下文。

## werkzeug

WSGI 协议工具集

## 配置参数讲解

创建app需要传递配置参数，以供各个模块使用

SECRET_KEY：密码加盐的参数，推荐设置成系统变量

## 扩展模块

扩展模块很多是基于现有的模块做扩展，封装成flask的扩展，比如把实例加入请求上下文中；模块的更多用法可以参考原模块。

需要分析用法的模块，列出项目地址。

### flask_script

用来自定义命令的，不过模块没有维护了，官方也推荐不要再使用它了，推荐使用 `@app.cli.command` 的形式来添加命令。然后配置环境变量，通过flask command的形式来执行命令。

可以使用装饰器，和类来添加命令。装饰器：`@manager.command`；类：继承Command类，实现run方法。

由于是弃用的模块，不再赘述。

### flask_migrate

Flask-Migrate是用于处理SQLAlchemy 数据库迁移的扩展工具。当Model出现变更的时候，通过migrate去管理数据库变更。依赖alembic模块。

用法：

```python

from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///app.db'

db = SQLAlchemy(app)
migrate = Migrate(app, db)

```

命令：

1. 初始化
flask db init
这个命令会在当前目录下生成一个migrations文件夹。这个文件夹也需要和其他源文件一起，添加到版本控制。

2. 生成最初的迁移
flask db migrate
此命令会在migrations下生成一个version文件夹，下面包含了对应版本的数据库操作py脚本。

3. 数据库升级
flask db upgrade
最后一步。此命令相当于执行了version文件夹下的相应py版本，对数据库进行变更操作。此后，对model有变更，只要重复migrate和upgrade操作即可。

由于migrate并不一定全部发现你对model的所有改动，因此生成的py脚本需要review, 有错的话则需要edit。

例如目前知道的，表名称表更，列名称变更，或给constraints命名等，migreate都不能发现的。更多限制细节见此：Alembic autogenerate documentation。

在Alembic 中，数据库迁移用迁移脚本表示。脚本中有两个函数，分别是upgrade() 和downgrade()。upgrade() 函数把迁移中的改动应用到数据库中，downgrade() 函数则将改动删除。Alembic 具有添加和删除改动的能力，因此数据库可重设到修改历史的任意一点。

我们可以使用revision 命令手动创建Alembic 迁移，也可使用migrate 命令自动创建。手动创建的迁移只是一个骨架，upgrade() 和downgrade() 函数都是空的，开发者要使用Alembic 提供的Operations 对象指令实现具体操作。自动创建的迁移会根据模型定义和数据库当前状态之间的差异生成upgrade() 和downgrade() 函数的内容。自动创建的迁移不一定总是正确的，有可能会漏掉一些细节。自动生成迁移脚本后一定要进行检查。

查看帮助文档：flask db --help

运行flask db init后，提示配置日志输出，默认输出到终端，应该配置日志到文件，方便以后排查问题（看看是谁动了数据库）

### flask_logging

[GitHub地址](https://github.com/dgilland/flask-logconfig)

用来快速实现登录验证。在User模型中，通过继承UserMixin实现以下属性：

1. is_authenticated 属性，用来判断是否是已经授权了，如果通过授权就会返回true

2. is_active 属性，判断是否已经激活

3. is_anonymous 属性，判断是否是匿名用户

4. get_id() 方法，返回用户的唯一标识

模块提供了login_required, login_user, logout_user等方法，验证成功后，通过login_user设置session。

### flask_sqlalchemy

配置参数：

- `SQLALCHEMY_DATABASE_URI`：配置数据库连接

`sqlite:////tmp/test.db 或 mysql://username:password@server/db`
- SQLALCHEMY_BINDS：用来绑定多个数据库，例如：
```
SQLALCHEMY_DATABASE_URI = 'postgres://localhost/main'
SQLALCHEMY_BINDS = {
    'users':        'mysqldb://localhost/users',
    'appmeta':      'sqlite:////path/to/appmeta.db'
}
```
多个数据库在创建数据库或在模型中声名使用哪个，比如在模型中 `__bind_key__ = 'users'`

- `SQLALCHEMY_ECHO`：值为Boolean（默认为False），设置为True，会把查询语句输出到stderr

- `SQLALCHEMY_RECORD_QUERIES`：可以用于显式地禁用或者启用查询记录。查询记录 在调试或者测试模式下自动启用。
一般我们不设置。

- `QLALCHEMY_NATIVE_UNICODE`：可以用于显式地禁用支持原生的unicode。

- `SQLALCHEMY_POOL_SIZE`：数据库连接池的大小。默认是数据库引擎的默认值（通常是 5）。当用户需要访问数据库时，并非建立一个新的连接，而是从连接池中取出一个已建立的空闲连接对象。使用完毕后，用户也并非将连接关闭，而是将连接放回连接池中，以供下一个请求访问使用。而连接的建立、断开都由连接池自身来管理。

- `SQLALCHEMY_POOL_TIMEOUT`：指定数据库连接池的超时时间。默认是10。

- `SQLALCHEMY_POOL_RECYCLE`：自动回收连接的秒数（1200即为2小时）。这对MySQL是必须的，默认情况下MySQL会自动移除闲置8小时或者以上的连接，Flask-SQLAlchemy会自动地设置这个值为2小时。也就是说如果连接池中有连接2个小时被闲置，那么其会被断开和抛弃。

- `SQLALCHEMY_MAX_OVERFLOW`：控制在连接池达到最大值后可以创建的连接数。当这些额外的连接使用后回收到连接池后将会被断开和抛弃。保证连接池只有设置的大小。

- `SQLALCHEMY_TRACK_MODIFICATIONS`：如果设置成 True (默认情况)，Flask-SQLAlchemy 将会追踪对象的修改并且发送信号。这需要额外的内存，如果不必要的可以禁用它。

常用字段

| Command      | Description 
| ------------ | :----------
| Integer      | int普通整数，一般是32位 
| SmallInteger | int取值范围小的整数，一般是16位 
| BigInteger   | int或long不限制精度的整数 
| Float        | float浮点数 
| Numeric      | decimal.Decimal普通整数，一般是32位 
| String       | str变长字符串 
| Text         | str变长字符串，对较长或不限长度的字符串做了优化 
| Unicode      | unicode变长Unicode字符串 
| UnicodeText  | unicode变长Unicode字符串，对较长或不限长度的字符串做了优化 
| Boolean      | bool布尔值 
| Date         | datetime.date时间 
| Time         | datetime.datetime日期和时间 
| LargeBinary  | str二进制文件 
| Enum         | enum枚举类型 

字段选项

| Command      | Description 
| ------------ | :---------------
| primary_key  | 如果为True，代表表的主键 
| unique       | 如果为True，代表这列不允许出现重复的值 
| index        | 如果为True，为这列创建索引，提高查询效率 
| nullable     | 如果为True，允许有空值，如果为False，不允许有空值 
| default      | 为这列定义默认值，如default=1 

flask_sqlalchemy的db实例提供了创建数据库的方法，不过这个是把当前上下文的模型创建数据库，如果是大型项目，那么就是把所有模型创建数据库，它的使用也有局限性，一般做为外部使用（一个测试工具，在这个上下文中创建db实例，对指定的模型来创建表）。所以应该使用迁移工具来管理数据库。

常用的过滤器

| Command     | Description 
| ----------- | :---------
| filter()    | 把过滤器添加到原查询上，返回一个新查询 
| filter_by() | 把等值过滤器添加到原查询上，返回一个新查询 
| limit()     | 使用指定的值限定原查询返回的结果 
| offset()    | 偏移原查询返回的结果，返回一个新查询 
| order_by()  | 根据指定条件对原查询结果进行排序，返回一个新查询 
| group_by()  | 根据指定条件对原查询结果进行分组，返回一个新查询 

执行器

| Command        | Description 
| -------------- | :------------
| all()          | 以列表形式返回查询的所有结果 
| first()        | 返回查询的第一个结果，如果未查到，返回None 
| first_or_404() | 返回查询的第一个结果，如果未查到，返回404 
| get()          | 返回指定主键对应的行，如不存在，返回None 
| get_or_404()   | 返回指定主键对应的行，如不存在，返回404 
| count()        | 返回查询结果的数量 
| paginate()     | 返回一个Paginate对象，它包含指定范围内的结果 

## 应用调度

`werkzeug.wsgi.DispatcherMiddleware` 可以把特定的请求分配到对应的一个新的flask应用上，这可以用来组织多个flask应用。需要注意的是新的flask应用和之前的是相互独立的，比如登录，使用的secret_key(可以使用不同的配置)等，所以要解耦项目，需要考虑这个问题，例如把接口的请求用新的app来处理，如果这些接口是需要登录才能使用的，那么在用户登录后，请求接口仍然需要登录

```python
from werkzeug.wsgi import DispatcherMiddleware
from frontend_app import application as frontend
from backend_app import application as backend

application = DispatcherMiddleware(frontend, {
    '/backend':     backend
})
```

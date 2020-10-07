---
weight: 9900
title: "Celery 在 web 开发中的运用"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Celery 在 web 开发中的运用"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Python, Note]
categories: [Middleware(中间件)]

lightgallery: true

toc:
  auto: false
---

Celery - Distributed Task Queue。要理解 Celery 本身不是任务队列，它是管理分布式任务队列的工具，或者换一种说法，它封装好了操作常见任务队列的各种操作，我们用它可以快速进行任务队列的使用与管理，当然你也可以自己看 rabbitmq 等队列的文档然后自己实现相关操作都是没有问题的。

<!-- more -->

## 安装

通过Python的包管理工具来安装，在我查到的一些资料中，celery和docker的配合不是很好，建议不要在单一容器中使用celery了。

## 架构

- 生产者(Celery client)。生产者(Celery client)发送消息。在Flask上工作时，生产者(Celery client)在Flask应用内运行。
- 消费者(Celery workers)。消费者用于处理后台任务。消费者(Celery client)可以是本地的也可以是远程的。我们可以在运行Flask的server上运行一个单一的消费者(Celery workers)，当业务量上涨之后再去添加更多消费者(Celery workers)。
- 消息传递者(message broker)。生产者(Celery client)和消费者(Celery workers)的信息的交互使用的是消息队列(message queue)。Celery支持若干方式的消息队列，其中最常用的是RabbitMQ和Redis.

以上是最基本的架构，完整的组件还包括：

1. Celery Beat：任务调度器
2. Celery Worker：执行任务的消费者
3. Broker：消息代理
4. Product：任务生产者（通过API，装饰器等产生任务并交个任务队列处理）
5. Result Backend：任务处理完成后，保存状态信息，以供查询

1，4都是任务的生产者，只是方式不一样，1的方式是Beat进程读取配置文件，周期性的将到期的任务发给任务队列执行，就是定时任务。

## 在flask中使用celery

如何集成，并且很好的解耦模块是celery运用的关键。你总不能把代码都堆叠在一起吧。

### 注意事项

- flask_celery不能支持celery4.0，所以弃用扩展模块，直接使用celery模块。使用扩展的好处是在扩展模块在一个文件初始化，并且全局保持一个实例对象，所以你的celery需要在app创建后才能创建，需要考虑是否使用了全局的celery对象
- 创建celery的实例对象的名字使用flask应用程序app的名字，通过 `app.name` 获取，如果你使用扩展插件，建议不要修改此名称，否则创建失败，不使用扩展插件也不建议修改
- 当有多个装饰器的时候，celery.task一定要在最外层

init代码

```python
app = create_app(CONFIG)
celery = CeleryApp(app.name)
celery.conf.update(app.config)
```

首先创建Flask app的实例app，然后创建Celery的实例celery，这里需要传递一个名称，这个名称会作为celery task的前缀，例如 `flask_app.celery_app.task.long_task`，long_task是我们定义的任务，你要改创建实例的参数也可以，建议不要修改。这里的CeleryApp是自己编写的，继承Celery的一个类，目的是实现单实例，让其它模块通过CeleryApp创建的实例保持一样，然后调用celery.conf.update更新参数，`flask app.config` 是继承dict的Config类，这样就把需要的参数配置通过flask配置，作用于celery。

flask大多数的插件做的事情就是在单独的扩展文件中，先不传递参数实例化扩展，然后在创建app的时候初始化它，估计flask_celery也是做差不多的事情，不过实例化Celery必须要先传递参数，现在插件没有在更新了(有一些其他名称的扩展可以支持)，主要还是为了工程化。

这里记录一下扩展包的情况吧，在py3中，有：

Flask-Celery：这个其实不是扩展，而是装这个就把Celery相关的给安装了，这个是Celery的作者写的，他也说在4.0版本不再需要了，我也没看出来这个有什么用

Flask-Celery-Helper：这个就是扩展了，导入用flask_celery，不支持4.0

Flask-Celery-py3，Flask-Celery3：好像都是不支持4.0的

Flask-CeleryExt：在文档中写明可以支持4.0了，和大部分扩展使用方法一样，用懒加载的方式实例化


### 数据序列化

| Command          | Description 
| ---------------- | :--------------: 
| pickle:          | 二进制序列化方式；是标准库的一个模块，支持Python的内置数据结构，但是他是Python的专有协议，在celery3.2开始，出于安全考虑，不再采用此方案；
| json:            | json支持多种语言，可用于跨语言方案，但好像不支持自定义的类对象； 
| XML:             | 类似标签语言； 
| msgpack:         | 二进制的类json序列化方案，但比json的数据结构更小，更快； 
| yaml:            | yaml表达能力更强，支持的数据类型较json多，但是python客户端的性能不如json; 

在自定义对象上，序列化方案我也出现过问题，pickle用了不行，可能就是不支持了，有待解决


### 使用步骤

1. `celery = Celery(app.name)` 创建celery实例
2. `celery.conf.update(app.config)` 更新配置
3. 在需要后台运行的任务使用@celery.task

```python
@celery.task
def hello_world():
    return "hello_world"
```

需要注意的是，被装饰的任务需要调用才会加到任务队列，也就是通过hello_world.delay()调用，在官方的例子中，通过继承的方式，增加 `__call__` 方法，内部调用run，这样装饰器@celery.task()便会直接加入任务队列了，不过这样的功能应该是不需要的。

通过 `r = hello_world.delay()` 方法，返回的对象拥有以下方法：

| Command          | Description 
| ---------------- | :--------------: 
| r.ready()        | #查看任务状态，返回布尔值，任务执行完成，返回True，否则返回False. 
| r.wait()         | #等待任务完成,返回任务执行结果，很少使用； 
| r.get(timeout=1) | #获取任务执行结果，可以设置等待时间 
| r.result         | #任务执行结果. 
| r.state          | #PENDING,START,SUCCESS，任务当前的状态 
| r.status         | #PENDING,START,SUCCESS，任务当前的状态 
| r.successful     | #任务成功返回true 
| r.traceback      | #如果任务抛出了一个异常，你也可以获取原始的回溯信息

### 装饰器参数

`@celery.task()`

1. name：可以显示指定任务的名字；

2. serializer：指定序列化的方法；

3. bind：一个bool值，设置是否绑定一个task的实例，如果把绑定，task实例会作为参数传递到任务方法中，可以访问task实例的所有的属性，具体属性可参照 `celery--app--task.py` 中的Task类，通过`self.request.__dict__`打印相关属性；

4. base：指定任务的基类，可以定义一个类，继承celery.Task，利用重写或扩展的类接口技术制定需求，例如on_success方法，默认是没有返回值的，就是提供这个钩子让开发者自定义的；

### 调用任务

任务被装饰器装饰后，通过task.delay()，task.apply_async()把任务加入到队列中，send_task()，可以发送未被注册的异步任务，即没有被celery.task装饰的任务

### apply_async的参数

| Command      | Description 
| ------------ | :----------
| countdown    | 设置该任务等待一段时间再执行，单位为s； 
| eta          | 定义任务的开始时间；eta=time.time()+10; 
| expires      | 设置任务时间，任务在过期时间后还没有执行则被丢弃； 
| retry        | 如果任务失败后,是否重试;使用true或false，默认为true 
| shadow       | 重新指定任务的名字str，覆盖其在日志中使用的任务名称； 
| retry_policy | 重试策略，为一个字典，各个键值配置：`max_retries`-最大重试次数，默认为 3 次. `interval_start`-重试等待的时间间隔秒数，默认为 0 ，表示直接重试不等待. `interval_step`-每次重试让重试间隔增加的秒数，可以是数字或浮点数，默认为 0.2. `interval_max`-重试间隔最大的秒数,即通过 interval_step 增大到多少秒之后，就不在增加了，可以是数字或者浮点数，默认为 0.2 .
| routing_key  | 自定义路由键； 
| queue        | 指定发送到哪个队列； 
| exchang      | 指定发送到哪个交换机； 
| priority     | 任务队列的优先级，0-9之间； 
| serializer   | 任务序列化方法；通常不设置； 
| compression  | 压缩方案，通常有zlib,bzip2 
| headers      | 为任务添加额外的消息； 
| link         | 任务成功执行后的回调方法；是一个signature对象；可以用作关联任务； 
| link_error   | 任务失败后的回调方法，是一个signature对象；

自定义发布者,交换机,路由键, 队列, 优先级,序列方案和压缩方法:

```python
task.apply_async((2,2), 
    compression='zlib',
    serialize='json',
    queue='priority.high',
    routing_key='web.add',
    priority=0,
    exchange='web_exchange')
```

### 基本配置

```python
# 注意，celery4版本后，CELERY_BROKER_URL改为BROKER_URL
BROKER_URL = 'amqp://username:passwd@host:port/虚拟主机名'
# 指定结果的接受地址
CELERY_RESULT_BACKEND = 'redis://username:passwd@host:port/db'
# 指定任务序列化方式
CELERY_TASK_SERIALIZER = 'msgpack' 
# 指定结果序列化方式
CELERY_RESULT_SERIALIZER = 'msgpack'
# 任务过期时间,celery任务执行结果的超时时间
CELERY_TASK_RESULT_EXPIRES = 60 * 20   
# 指定任务接受的序列化类型.
CELERY_ACCEPT_CONTENT = ["msgpack"]   
# 任务发送完成是否需要确认，这一项对性能有一点影响     
CELERY_ACKS_LATE = True  
# 压缩方案选择，可以是zlib, bzip2，默认是发送没有压缩的数据
CELERY_MESSAGE_COMPRESSION = 'zlib' 
# 规定完成任务的时间
CELERYD_TASK_TIME_LIMIT = 5  # 在5s内完成任务，否则执行该任务的worker将被杀死，任务移交给父进程
# celery worker的并发数，默认是服务器的内核数目,也是命令行-c参数指定的数目
CELERYD_CONCURRENCY = 4 
# celery worker 每次去rabbitmq预取任务的数量
CELERYD_PREFETCH_MULTIPLIER = 4 
# 每个worker执行了多少任务就会死掉，默认是无限的
CELERYD_MAX_TASKS_PER_CHILD = 40 
# 设置默认的队列名称，如果一个消息不符合其他的队列就会放在默认队列里面，如果什么都不设置的话，数据都会发送到默认的队列中
CELERY_DEFAULT_QUEUE = "default" 
# 设置详细的队列
CELERY_QUEUES = {
    "default": { # 这是上面指定的默认队列
        "exchange": "default",
        "exchange_type": "direct",
        "routing_key": "default"
    },
    "topicqueue": { # 这是一个topic队列 凡是topictest开头的routing key都会被放到这个队列
        "routing_key": "topic.#",
        "exchange": "topic_exchange",
        "exchange_type": "topic",
    },
    "task_eeg": { # 设置扇形交换机
        "exchange": "tasks",
        "exchange_type": "fanout",
        "binding_key": "tasks",
    },
}
```

## 命令

`celery worker -A auto_app.celery --loglevel=info` 启动Worker

## 任务状态

| Command | Description 
| ------- | :-----: 
| PENDING | 任务等待中 
| STARTED | 任务已开始 
| SUCCESS | 任务执行成功 
| FAILURE | 任务执行失败 
| RETRY   | 任务将被重试 
| REVOKED | 任务取消 

通过 `r.get('status') == 'PENDING'` 获取状态

## 设置任务调度器

配置文件:

```python
from datetime import timedelta
from celery.schedules import crontab

Config = dict(
    CELERYBEAT_SCHEDULE={
        'ptask': {
            'task': 'flask_app.celery_app.task.period_task',
            'schedule': timedelta(seconds=5),
        },
    },
    CELERY_TIMEZONE='Asia/Shanghai'
)
```

配置中 schedule 就是间隔执行的时间，这里可以用 datetime.timedelta 或者 crontab，如果定时任务涉及到 datetime 需要在配置中加入时区信息，否则默认是以 utc 为准。例如中国可以加上：

`CELERY_TIMEZONE = 'Asia/Shanghai'`

task的任务路径不能出错，在启动Worker进程的时候，可以看到task列表，这里指的的定时任务和其对应即可。

启动命令：

需要执行两个进程，一个是Worker进程，用来处理生成的任务，一个就是beat进程，启动任务调度器进程，定时生成任务

- `celery beat -A auto_app.celery --loglevel=info`
- `celery worker -A auto_app.celery --loglevel=info`

{% blockquote %}
任务调度会有需要动态添加任务，管理任务的情况，Django框架通过djang-celery实现在管理后台创建，删除，更新任务，它通过自定义调度类来实现，如果有类似的需求，可以参考源码实现
{% endblockquote %}

## 工作流

Signature 对象，把任务通过签名的方法传递给其它任务，成为一个子任务

```
In [6]: task = signature('flask_app.celery_app.task.add', args=(2, 2), countdown=5)
In [7]: task
Out[7]: flask_app.celery_app.task.add(2, 2)
In [8]: task.apply_async()
Out[8]: <AsyncResult: 0cbe319e-c3f6-48b9-b1e4-6a034711cf3a>
```

`from celery import signature` 导入signature，可以看到，传递的第一个参数是已经存在的任务，也可以先把add导入，通过 `add.subtask((2, 2), countdown=5)`，或使用subtask的缩写s，add.s()。

子任务能支持偏函数的方式，利用它实现工作流。

支持原语实现工作流，原语表示由若干条指令组成的，用于完成一定功能的过程

1.chain - 调用链，任务的链式执行，前面的执行结果作为参数传递给后面，直到任务完成

chain 函数接受一个任务的列表，Celery 保证一个 chain 里的子任务会依次执行，在 AsynResult 上执行 get 会得到最后一个任务的返回值。和 link 功能类似，每一个任务执行结果会当作参数传入下一个任务，所以如果你不需要这种特性，采用 immutable signature 来取消。

```python
def subtask():
    from celery import chain
    part = add.s(1, 2) | add.s(3) | add.s(5)
    # or part = (add.s(1, 2), add.s(3), add.s(5))
    res = chain(part)()
    print(res.get())
```

2.group - 任务的并发执行

```python
def subtask():
    from celery import group
    res = group([add.s(i, i) for i in range(1, 10)])()
    print(res.get())
```

group 函数也接受一个任务列表，这些任务会同时加入到任务队列中，且执行顺序没有任何保证。在 AsynResult 上执行 get 会得到一个包含了所有返回值的列表。`意参数必须是list对象`

3. chord - 带回调的 group

chord 基本功能和 group 类似，只是有一个额外的回调函数。回调函数会在前面的任务全部结束时执行，其参数是一个包含了所有任务返回值的列表。在 AsynResult 上执行 get 会得到回调函数的返回值。

4. map/starmap - 每个参数都作为任务的参数执行一遍

5. chunks - 将任务分块

## 总结

在启动Worker进程后，可以看到被装饰的任务已经被列出来了，这说明Celery有读取文件的机制(你可以在任务模块的最外层使用print测试)，被装饰的函数应该要在最外层，而且，创建实例后，再去修改配置，似乎没有生效（在我的测试中是这样的），其实这也符合逻辑，在进程被创建了，却又动态的去修改配置，与之对应的风险也很高。

celery是队列管理工具，真正的队列是Broker，更深入一点要了解RabbitMQ，AMQP协议，一般在celery上关注Worker，可以使用多个Worker，任务的生成使用定时器或触发的机制，任务本身就要由Python来编写，也包括对执行结果的处理。

任务生成，处理有了，还有队列的管理，默认使用名为celery的队列，可以配置队列，比如队列A，队列B，进入A队列的任务优先级要高，会被先处理。可以在启动worker进程的时候指明队列(通过-Q指定队)，这样这个Worker只会处理指定的队列。

后续扩展内容：celery信号，分析任务执行情况。Worker管理，监控和管理celery。


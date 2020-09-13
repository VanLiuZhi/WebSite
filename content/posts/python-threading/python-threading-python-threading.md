---
weight: 1000
title: "Python-threading 多线程"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Python-threading 多线程"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Python, Note]
categories: [Python编程]

lightgallery: true

toc:
  auto: false
---

python多线程相关笔记

<!-- more -->

## 线程

使用标准库threading来创建线程。threading 库可以在单独的线程中执行任何的在 Python 中可以调用的对象。你可 以创建一个 Thread 对象并将你要执行的对象以 target 参数的形式提供给该对象。虽然python GIL 的存在，导致多线程同一时刻只能有一个线程获得解释器（在py2中，大概执行1000行字节码后，会释放解释器，当线程被阻塞的时候，会让出解释器，释放GIL）

可以通过time.sleep(3)来阻塞线程

一个简单例子：

```python

import threading

# 计算密集型任务
def func():
    a = [i for i in range(1111)]
    print('hello world')


t = threading.Thread(target=func)
t.start()
print('sleep')
# 此时创建列表a占用了解释器，先hello world 再 sleep


# 计算密集型任务
def func():
    a = [i for i in range(11111111111)]
    print('hello world')


t = threading.Thread(target=func)
t.start()
print('sleep')
# 这种情况，先打印sleep再是hello world（执行一定的字节码后，释放了解释器）

```

## threading的属性和方法

- current_thread()  # 返回当前线程对象.
- main_thread()  # 返回主线程对象.
- active_count()  # 当前处于alive状态的线程个数.
- enumerate()  # 返回所有活着的线程的列表，不包括已经终止的线程和未开始的线程.
- get_ident()  # 返回当前线程ID，非0整数.

看一个例子：

```python

import threading

def func():
    # a = [i for i in range(1111)]
    print('current thread = {}'.format(threading.current_thread()))
    print('main thread = {}'.format(threading.main_thread()), '"主线程对象"')
    print('active count = {}'.format(threading.active_count()), '"alive"')
    print('hello world')


t = threading.Thread(target=func)
t.start()
print('sleep')
print('current thread = {}'.format(threading.current_thread()))
print('main thread = {}'.format(threading.main_thread()), '"主线程对象"')
print('active count = {}'.format(threading.active_count()), '"alive"')

```

运行以上代码，每次的执行结果是不一样的，而且是print是线程不安全的。要解释这个问题，需要再了解一些线程相关的概念。

## thread实例的属性和方法

- name: 只是一个名称标识，可以重名，getName()、setName()来获取、设置这个名词。
- ident: 线程ID，它是非0整数。线程启动后才会有ID，否则为None。线程退出，此ID依旧可以访问。此ID可以重复使用。
- is_alive(): 返回线程是否活着。

通过threading.Thread()  我们创建了线程类的实例，像面向对象一样，可以有对应的方法，属性

`t = threading.Thread(target=func, name='my_thread', args=('1', ), kwargs={'a': 2})`

- start(): 启动线程。每一个线程必须且只能执行该方法一次。

开始线程活动。

对每一个线程对象来说它只能被调用一次，它安排对象在一个另外的单独线程中调用run()方法（而非当前所处线程）。
当该方法在同一个线程对象中被调用超过一次时，会引发RuntimeError(运行时错误)。

- run(): 运行线程函数。

代表了线程活动的方法。

你可以在子类中重写此方法。标准run()方法调用了传递给对象的构造函数的可调对象作为目标参数，如果有这样的参数的话，顺序和关键字参数分别从args和kargs取得。

start() 后，还会执行run。如果你重写线程类，在调用start和run的时候，加入打印代码，start执行的线程，会派生出子线程，在子线程中去执行run，配合threading.current_thread()可以看到整个过程。

而run只在当前线程中执行。

## 多线程情况

继承Thread类，使用Extender的形式扩展start和run方法，观察执行情况。我们开启两个线程，然后start他们，利用threading.current_thread()获取当前线程，main_thread()返回主线程对象

```python

import threading
import time
import logging

logging.basicConfig(level=logging.NOTSET)


def worker():
    count = 0
    while True:
        if count > 5:
            break
        time.sleep(1)
        count += 1
        # print("worker running")
        logging.info("{} {} 主线程：{}".format(threading.current_thread().name, threading.current_thread().ident,
                                        threading.main_thread()))
        # print(threading.current_thread().name, threading.current_thread().ident)


class MyThread(threading.Thread):
    def start(self):
        print('start~~~~~~~~~~~~~')
        super().start()

    def run(self):
        print('run~~~~~~~~~~~~~~~')
        super().run()


print(threading.main_thread())

t = MyThread(name='worker', target=worker)
t2 = MyThread(name='not worker', target=worker)
t.start()
t2.start()
t.join()
t2.join()

# 输出结果
# <_MainThread(MainThread, started 4587271616)>
# start~~~~~~~~~~~~~
# run~~~~~~~~~~~~~~~
# start~~~~~~~~~~~~~
# run~~~~~~~~~~~~~~~
# INFO:root:worker 123145369858048 主线程：<_MainThread(MainThread, started 4587271616)>
# INFO:root:not worker 123145375113216 主线程：<_MainThread(MainThread, started 4587271616)>
# INFO:root:worker 123145369858048 主线程：<_MainThread(MainThread, started 4587271616)>
# INFO:root:not worker 123145375113216 主线程：<_MainThread(MainThread, started 4587271616)>
# INFO:root:worker 123145369858048 主线程：<_MainThread(MainThread, started 4587271616)>
# INFO:root:not worker 123145375113216 主线程：<_MainThread(MainThread, started 4587271616)>
# INFO:root:worker 123145369858048 主线程：<_MainThread(MainThread, started 4587271616)>
# INFO:root:not worker 123145375113216 主线程：<_MainThread(MainThread, started 4587271616)>
# INFO:root:worker 123145369858048 主线程：<_MainThread(MainThread, started 4587271616)>
# INFO:root:not worker 123145375113216 主线程：<_MainThread(MainThread, started 4587271616)>
# INFO:root:worker 123145369858048 主线程：<_MainThread(MainThread, started 4587271616)>
# INFO:root:not worker 123145375113216 主线程：<_MainThread(MainThread, started 4587271616)>

```

可以看到两个线程交替运行，如果使用print，你跑多次这个结果是不一样的。

打印前可以加入threading.main_thread()，这样可以看到俩个线程都是主线程派生出来的子线程。

换成run()方法后，结果如下：

```python

import threading
import time
import logging

logging.basicConfig(level=logging.NOTSET)


def worker():
    count = 0
    while True:
        if count > 5:
            break
        time.sleep(1)
        count += 1
        # print("worker running")
        # print(threading.main_thread().name, threading.current_thread().name, threading.current_thread().ident)
        logging.info("{} {} 主线程：{}".format(threading.current_thread().name, threading.current_thread().ident,
                                        threading.main_thread()))


class MyThread(threading.Thread):
    def start(self):
        print('start~~~~~~~~~~~~~')
        super().start()

    def run(self):
        print('run~~~~~~~~~~~~~~~')
        super().run()


t = MyThread(name='worker', target=worker)
t2 = MyThread(name='not worker', target=worker)
t.run()
t2.run()

# run~~~~~~~~~~~~~~~
# INFO:root:MainThread 4705641920 主线程：<_MainThread(MainThread, started 4705641920)>
# INFO:root:MainThread 4705641920 主线程：<_MainThread(MainThread, started 4705641920)>
# INFO:root:MainThread 4705641920 主线程：<_MainThread(MainThread, started 4705641920)>
# INFO:root:MainThread 4705641920 主线程：<_MainThread(MainThread, started 4705641920)>
# INFO:root:MainThread 4705641920 主线程：<_MainThread(MainThread, started 4705641920)>
# run~~~~~~~~~~~~~~~
# INFO:root:MainThread 4705641920 主线程：<_MainThread(MainThread, started 4705641920)>
# INFO:root:MainThread 4705641920 主线程：<_MainThread(MainThread, started 4705641920)>
# INFO:root:MainThread 4705641920 主线程：<_MainThread(MainThread, started 4705641920)>
# INFO:root:MainThread 4705641920 主线程：<_MainThread(MainThread, started 4705641920)>
# INFO:root:MainThread 4705641920 主线程：<_MainThread(MainThread, started 4705641920)>
# INFO:root:MainThread 4705641920 主线程：<_MainThread(MainThread, started 4705641920)>
# INFO:root:MainThread 4705641920 主线程：<_MainThread(MainThread, started 4705641920)>

```

可以看到，run就是去调用函数，谁来调用呢？当然是`当前线程`了，可以看到 `print(threading.main_thread().name, threading.current_thread().name, threading.current_thread().ident)` 打印出来的都是主线程。

**没有开新的线程，这就是普通函数调用，所以执行完t1.run()，然后执行t2.run()，这里就不是多线程。**

当使用start方法启动线程后，进程内有多个活动的线程并行的工作，就是多线程。

一个进程中至少有一个线程，并作为程序的入口，这个线程就是主线程。一个进程至少有一个`主线程`。其他线程称为`工作线程`。

## 线程安全

使用print来运行上面的两个例子，本应该是一行行打印，但很多字符串打印在了一起，这说明print函数被打断了，被线程切换打断了。

print函数分两步，第一步打印字符串，第二部换行，就在这之间，发生了线程的切换。

说明print函数不是线程安全函数。

print函数还没执行换行符，就被其它线程打断了，在python3中：

`def print(self, *args, sep=' ', end='\n', file=None)`

print变成了函数，结尾默认加‘\n’，你可以去改变这个参数，比如改成'', 打印结果就是一行的一串字符

线程安全: 线程执行一段代码，不会产生不确定的结果，那这段代码就是线程安全的。在开发中，我们会使用标准库的logging来，打印信息，这个是线程安全的。

## 线程daemon

线程可以被标识为"Daemon线程"，**Daemon线程表明整个Python主程序只有在Daemon子线程运行时可以退出**。该属性值继承自父线程，可通过setDaemon()函数设定该值。

daemon线程和non-daemon线程(注：这里的daemon不是Linux中的守护进程)：

- 进程靠线程执行代码，至少有一个主线程，其他线程是工作线程。
- 主线程是第一个启动的线程。
- 父线程：如果线程A中启动了一个线程B，A就是B的父线程。
- 子线程：B就是A的子线程。
- python中构造线程的时候可以设置daemon属性，这个属性必须在start方法之前设置好。

daemon属性：表示线程是否是daemon线程，这个值必须在start()之前设置，否则引发RuntimeError异常。

{% blockquote %}
daemon=False 运行发现子线程依然执行，主线程已经执行完，但是主线程会一直等着子线程执行完
daemon=True 运行发现主线程执行完程序立即结束了
{% endblockquote %}

实例方法：

isDaemon()：是否是daemon线程。
setDaemon()：设置为daemon线程，必须在start方法之前设置。
 
总结:

线程具有一个daemon属性，可以显式设置为True或False，也可以不设置，不设置则取默认值None。

如果不设置daemon，就取当前线程的daemon来设置它。子线程继承父线程的daemon值，作用和设置None一样。

主线程是non-daemon线程，即daemon=False。

从主线程创建的所有线程不设置daemon属性，则默认都是daemon=False，也就是non-daemon线程。

python程序在没有活着的non-daemon线程运行时退出，也就是剩下的只能是daemon线程，主线程才能退出，否则主线程就只能等待。
- 如果有non-daemon线程的时候，主线程退出时，也不会杀掉所有daemon线程，直到所有non-daemon线程全部结束
- 如果还有daemon线程，主线程需要退出，会结束所有 daemon线程，退出。

线程创建的时候`t = threading.Thread(target=func, daemon=False)`这个daemon不设置就是False

子线程也是non-daemon，只要有线程是non-daemon，python程序就不会退出，如果还未执行完成的线程是daemon的，主线程执行完，就会退出，并杀掉所有daemon线程。

Daemon线程会被粗鲁的直接结束，它所使用的资源（已打开文件、数据库事务等）无法被合理的释放。

```python

import time
import threading


def foo(n):
    for i in range(n):
        print(i)
        time.sleep(1)


t1 = threading.Thread(target=foo, args=(10,), daemon=True)
t1.start()
# t1.join()  # 设置join.
print('Main Thread Exiting')

```

在这个例子中，子线程开始执行，然后主线程执行了打印，由于主线程执行完成了，而剩下的线程是daemon的，所以程序退出。把daemon = False或者不设置，结果就是打印了Main Thread Exiting后，子线程继续，打印1，2，3.....

## join

- 使用了join方法后，daemon线程执行完了，主线程才退出。
- join(timeout=None)，是线程的标准方法之一。
- 一个线程中调用另一个线程的join方法，调用者将被阻塞，直到被调用线程终止。
- 一个线程可以被join多次。
- timeout参数指定调用者等待多久，没有设置超时，就一直等待被调用线程结束。
- 调用谁的join方法，就是join谁，就要等谁。

把上面例子的 `t1.join()  # 设置join.` 放开，`print('Main Thread Exiting')` 要等子线程执行完成才执行。

join ()方法：主线程A中，创建了子线程B，并且在主线程A中调用了B.join()，那么，主线程A会在调用的地方等待，直到子线程B完成操作后，才可以接着往下执行，那么在调用这个线程时可以使用被调用线程的join方法。

总结：

主要理解daemon join，不做处理的多线程，线程是并发的，daemon控制了主线程是否等待子线程执行完成，join控制了线程是否要组赛，主线程被阻塞了，就不会因为还剩daemon线程退出，因为主线程被阻塞了，他还没有执行完，所以这两个概念是互不冲突的（你可以设置超时时间，超时到了，主线程不再阻塞，就会杀掉daemon线程）。

在主线程中创建了3个线程，3个线程执行了join，就是说主线程要等着3个线程完成才执行，3个线程中的A线程创建了线程a，那么a就是A的子线程，a中join A就要等a执行完成，主线程也被阻塞，在等A，即主线程等A，A等a。

## 原子性

python的大部分操作是原子性的，比如你对列表执行反向，排序，它不会被其它线程打断。

```python
import dis
dis.dis(foo)
```

利用标准库的dis可以看python代码的字节码实现，一般操作由一条指令来完成，那么就是原子性，如果一个操作（对应python的一行或几行代码）需要多个指令（入栈，出栈，调用寄存器等），可能在入栈等某个指令的时候被其它线程打断，出现和预期不一样的效果。

## 队列

标准库queue提供了队列支持，在py2中，通过import Queue来使用队列，在py3中，通过from queue import Queue，py3中，除了Queue类，还增加了queue.LifoQueue（LIFO后进先出队列），queue.PriorityQueue（优先级队列）

### 实例方法

`q = queue.Queue(3) # 创建队列，队列最大元素3个，默认为0，此时队列长度没有限制`

- queue.qsize() 返回队列的大小
- queue.empty() 如果队列为空，返回True，反之False
- queue.full() 如果队列满了，返回True，反之False
- queue.full 与 maxsize 大小对应
- queue.get([block[, timeout]])获取队列，timeout等待时间
- queue.get_nowait() 相当queue.get(False)
- queue.put(item) 写入队列，timeout等待时间
- queue.put_nowait(item) 相当queue.put(item, False)
- queue.task_done() 在完成一项工作之后，queue.task_done()函数向任务已经完成的队列发送一个信号
- queue.join() 实际上意味着等到队列为空，再执行别的操作

## 本地线程

不同的线程对内容的修改只在线程内发挥作用，线程之间互相不影响，在flask框架中有使用到

```python

import threading

my_data = threading.local()
my_data.number = 42
print(my_data.number)
log = []

def f():
    my_data.number = 11
    log.append(my_data.number)
    print(id(my_data.number))

thread = threading.Thread(target=f)
thread.start()
thread.join()
print(log)
print(my_data.number)
print(id(my_data.number))

# 42
# 4559721904
# [11]
# 42
# 4559722896

```

## 同步原语

控制多线程同时访问资源，包括互斥锁，信号量，条件变量，事件

以房间为例子举例：

有些房间最多只能容纳一个人。里面有人的时候，其他人就不能进去了。这代表一个线程使用某些共享内存时，其他线程必须等它结束，才能使用这一块内存。

一个防止他人进入的简单方法，就是门口加一把锁。先到的人锁上门，后到的人看到上锁，就在门口排队，等锁打开再进去。这就叫互斥锁（Mutual exclusion，缩写 Mutex），防止多个线程同时读写某一块内存区域。

还有些房间，可以同时容纳n个人。也就是说，如果人数大于n，多出来的人只能在外面等着。这好比某些内存区域，只能供给固定数目的线程使用。

这时的解决方法，就是在门口挂n把钥匙。进去的人就取一把钥匙，出来时再把钥匙挂回原处。后到的人发现钥匙架空了，就知道必须在门口排队等着了。这种做法叫做信号量（Semaphore），用来保证多个线程不会互相冲突。

不难看出，mutex是semaphore的一种特殊情况（n=1时）。也就是说，完全可以用后者替代前者。但是，因为mutex较为简单，且效率高，所以在必须保证资源独占的情况下，还是采用这种设计。

### 信号量

使用信号量做为同步机制，使用with进入上下文管理器，省略了acquire和release，信号量通过计数器来管理，这里计数器初始是3，获取acquire操作，计数器减1，release操作，计数器加1，当计数器为0的时候，阻塞其它线程的操作。

通过执行结果可以看到，创建了5个线程，前3个线程 0，1，2 执行了 acquire操作，使得信号量为0，阻塞了其它线程，通过sleep模拟
线程阻塞，等到线程 2 release的时候，线程 3 才执行 acquire 操作，4 线程也是等待 3 线程release后才执行 acquire。

通过使用信号量，实现了只能有3个线程并发，而锁其实就是信号量为1的情况。

```python

import time
from random import random
from threading import Thread, Semaphore

sema = Semaphore(3) # 创建信号量


def foo(tid):
    with sema:
        print(f'{tid} acquire sema')
        time.sleep(random() * 2)
    print(f'{tid} release sema')


threads = []

for i in range(5):
    t = Thread(target=foo, args=(i,))
    threads.append(t)
    t.start()

for i in threads:
    i.join()

# 0 acquire sema
# 1 acquire sema
# 2 acquire sema
# 2 release sema
# 3 acquire sema
# 3 release sema
# 4 acquire sema
# 1 release sema
# 0 release sema
# 4 release sema

```

## 总结

所以线程的执行结果是有很多因素影响的，在你用默认操作的时候，如果进行了IO密集任务或是CPU密集任务，IO密集在等待时会释放GIL，CPU密集也会执行一定数量的字节码后释放一下GIL，由于线程并发的切换是操作系统控制的，所以有这样的编程需求的时候，务必配合join，daemon等控制程序，不然什么时候切换，这是说不准的。

线程何时切换？一个线程无论何时开始睡眠或等待网络 I/O，其他线程总有机会获取 GIL 执行 Python 代码。这是协同式多任务处理。CPython 也还有抢占式多任务处理。如果一个线程不间断地在 Python 2 中运行 1000 字节码指令，或者不间断地在 Python 3 运行15 毫秒，那么它便会放弃 GIL，而其他线程可以运行。把这想象成旧日有多个线程但只有一个 CPU 时的时间片。

1. 协同式多任务处理
当一项任务比如网络 I/O启动，而在长的或不确定的时间，没有运行任何 Python 代码的需要，一个线程便会让出GIL，从而其他线程可以获取 GIL 而运行 Python。这种礼貌行为称为协同式多任务处理，它允许并发；多个线程同时等待不同事件。

2. 抢占式多任务处理
Python线程可以主动释放 GIL，也可以先发制人抓取 GIL 。

让我们回顾下 Python 是如何运行的。你的程序分两个阶段运行。首先，Python文本被编译成一个名为字节码的简单二进制格式。第二，Python解释器的主回路，一个名叫 pyeval_evalframeex() 的函数，流畅地读取字节码，逐个执行其中的指令。当解释器通过字节码时，它会定期放弃GIL，而不需要经过正在执行代码的线程允许，这样其他线程便能运行。默认情况下，检测间隔是1000 字节码。所有线程都运行相同的代码，并以相同的方式定期从他们的锁中抽出。在 Python 3 GIL 的实施更加复杂，检测间隔不是一个固定数目的字节码，而是15 毫秒。然而，对于你的代码，这些差异并不显著。


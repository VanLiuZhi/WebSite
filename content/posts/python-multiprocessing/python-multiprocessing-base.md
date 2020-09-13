---
weight: 1000
title: "Python-multiprocessing 多进程"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Python-multiprocessing 多进程"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Python, Note]
categories: [Python编程]

lightgallery: true

toc:
  auto: false
---

python多进程学习笔记

<!-- more -->

# 进程

Python标准库为我们提供了 `threading` 和 `multiprocessing` 模块编写相应的多线程/多进程代码。从Python3.2开始，标准库为我们提供了 `concurrent.futures` 模块，它提供了 `ThreadPoolExecutor` 和 `ProcessPoolExecutor` 两个类，实现了对 `threading` 和 `multiprocessing` 的更高级的抽象，对编写线程池/进程池提供了直接的支持。 

concurrent.futures基础模块是executor和future。future是concurrent.futures模块和asyncio模块的重要组件

## 概念

### fork()

pid=os.fork()
1. 只用在Unix系统中有效，Windows系统中无效
2. fork函数调用一次，返回两次：在父进程中返回值为子进程id，在子进程中返回值为0

### 和线程的区别

进程是操作系统资源分配的基本单位，而线程是任务调度和执行的基本单位

### 进程池

python中，进程池内部会维护一个进程序列。当需要时，程序会去进程池中获取一个进程。如果进程池序列中没有可供使用的进程，那么程序就会等待，直到进程池中有可用进程为止。

进程池支持同步，异步，映射方式添加任务到进程池。

### 预创建

预先创建一组子进程，当有新任务来时，系统通过 调配 该组进程中的某个 子进程 完成此任务。

为什么需要？
1. 进程创建，销毁需要消耗cpu时间
2. 预先创建，以空间换时间，提升性能。时间是唯一稀缺资源，空间不足加内存，能够用钱解决的问题，都不是问题，钱都解决不了的问题才是问题
3. 通过系统合理分配任务，提高性能。尽量实现真正的并行处理，提升系统处理效率

### 关于GIL

多进程，一个进程就有一个GIL

### 内核态和用户态

我们知道现在操作系统都是采用虚拟存储器，那么对32位操作系统而言，它的寻址空间（虚拟存储空间）为4G（2的32次方）。操心系统的核心是内核，独立于普通的应用程序，可以访问受保护的内存空间，也有访问底层硬件设备的所有权限。为了保证用户进程不能直接操作内核，保证内核的安全，操心系统将虚拟空间划分为两部分，一部分为内核空间，一部分为用户空间。针对linux操作系统而言，将最高的1G字节（从虚拟地址0xC0000000到0xFFFFFFFF），供内核使用，称为内核空间，而将较低的3G字节（从虚拟地址0x00000000到0xBFFFFFFF），供各个进程使用，称为用户空间。每个进程可以通过系统调用进入内核，因此，Linux内核由系统内的所有进程共享。于是，从具体进程的角度来看，**每个进程可以拥有4G字节的虚拟空间**。我们常说的陷入内核态，就是当前进程进入内核，去访问更高权限的东西。

需要注意的细节问题：
- 内核空间中存放的是内核代码和数据，而进程的用户空间中存放的是用户程序的代码和数据。不管是内核空间还是用户空间，它们都处于虚拟空间中。 
- Linux使用两级保护机制：0级供内核使用，3级供用户程序使用。
- 当一个任务（进程）执行系统调用而陷入内核代码中执行时，称进程处于内核运行态（内核态）。此时处理器处于特权级最高的（0级）内核代码中执行。当进程处于内核态时，执行的内核代码会使用当前进程的内核栈。每个进程都有自己的内核栈。
- 当进程在执行用户自己的代码时，则称其处于用户运行态（用户态）。此时处理器在特权级最低的（3级）用户代码中运行。当正在执行用户程序而突然被中断程序中断时，此时用户程序也可以象征性地称为处于进程的内核态。因为中断处理程序将使用当前进程的内核栈。

### windows中创建进程

在windows中创建进程，相当于导入模块，所以要把进行放在 `if __name__ == '__main__':` 后面来执行，而且子进程会把代码再跑一次，如果你在代码块最外层有打印等执行语句，也会被执行（所以一般外层的代码都封装在函数里面，在linux上不会这样，由于外层代码不一定全是函数，只要是会被执行的都要执行，这和想像的情况相差太多，所以我建议多进程只在linux上使用，在windos上的多进程，只有演示学习的意义，过于复杂的程序受到太多的限制）

这是 Windows 上多进程的实现问题。在 Windows 上，子进程会自动 import 启动它的这个文件，而在 import 的时候是会执行这些语句的。如果你这么写的话就会无限递归创建子进程报错。但是在multiprocessing.Process的源码中是对子进程再次产生子进程是做了限制的，是不允许的，于是出现如上的错误提示。所以必须把创建子进程的部分用那个 if 判断保护起来，import 的时候 name 不是 main ，就不会递归运行了。

### 操作系统的设计

1. 以多进程形式，允许多个任务同时运行；
2. 以多线程形式，允许单个任务分成不同的部分运行；
3. 提供协调机制，一方面防止进程之间和线程之间产生冲突，另一方面允许进程之间和线程之间共享资源。

## 队列

标准库提供了队列模块queue，在py2中，通过 `import Queue` 使用队列，在py3中通过 `from queue import Queue` 来使用队列。
在multiprocessing模块中，通过了Queue类来实现队列。queue.Queue是进程内非阻塞队列，multiprocess.Queue是跨进程通信队列。


1. from queue import Queue 
这个是普通的队列模式，类似于普通列表，先进先出模式，get方法会阻塞请求，直到有数据get出来为止

2. from multiprocessing.Queue import Queue（各子进程共有）
这个是多进程并发的Queue队列，用于解决多进程间的通信问题。普通Queue实现不了

### multiprocessing.Queue

queue = Queue(5) 初始化Queue()对象时（ 例如：q=Queue() ），若括号中没有指定最大可接收的消息数量，或数量为负值，那么就代表可接受的消息数量没有上限（直到内存的尽头）

Queue.qsize()：返回当前队列包含的消息数量，结果不可靠，理由同q.empty()和q.full()一样

Queue.empty()：如果队列为空，返回True，反之False。该结果不可靠，比如在返回True的过程中，如果队列中又加入了项目。

Queue.full()：如果队列满了，返回True，反之False。该结果不可靠，比如在返回True的过程中，如果队列中的项目被取走。

Queue.get([block[, timeout]])：获取队列中的一条消息，然后将其从列队中移除，block默认值为True；这里的block代表阻塞。

- 如果block使用默认值，且没有设置timeout（单位秒），消息列队如果为空，此时程序将被阻塞（停在读取状态），直到从消息列队读到消息为止，如果设置了timeout，则会等待timeout秒，若还没读取到任何消息，则抛出"Queue.Empty"异常

- 如果block值为False，消息列队如果为空，则会立刻抛出"Queue.Empty"异常

Queue.get_nowait()：相当Queue.get(False)

Queue.put(item,[block[, timeout]])：将item消息写入队列，block默认值为True

- 如果block使用默认值，且没有设置timeout（单位秒），消息列队如果已经没有空间可写入，此时程序将被阻塞（停在写入状态），直到从消息列队腾出空间为止，如果设置了timeout，则会等待timeout秒，若还没空间，则抛出"Queue.Full"异常

- 如果block值为False，消息列队如果没有空间可写入，则会立刻抛出"Queue.Full"异常

Queue.put_nowait(item)：相当Queue.put(item, False)

## 管道

进程间通信（IPC）方式二：管道

创建管道的类：`Pipe([duplex])`，在进程之间创建一条管道，并返回元组（conn1,conn2），其中conn1，conn2表示管道两端的连接对象，强调一点：**必须在产生Process对象之前产生管道**

参数介绍：

duplex:默认值是True，管道是全双工的，如果将duplex改成False，conn1只能用于接收，conn2只能用于发送。

方法介绍：（详情参考文档：multiprocessing.connection.Connection类）

- conn1.recv()：接收conn2.send(obj)发送的对象。如果没有消息可接收，recv方法会一直阻塞。如果连接的另外一端已经关闭，那么recv方法会抛出EOFError
- conn1.send(obj)：将一个对象发送到应该使用recv()读取的连接的另一端。obj必须是与序列化兼容的任意对象。非常大的pickle(大约32 MiB+，尽管这取决于操作系统)可能会引发ValueError异常
- conn1.close()：关闭连接。如果conn1被垃圾回收，将自动调用此方法
- conn1.fileno()：返回连接使用的整数文件描述符
- conn1.poll([timeout])：返回是否有任何数据可供读取(如果连接上的数据可用，返回True)。timeout指定等待的最长时限。如果省略此参数，方法将立即返回结果。如果将timeout射成None，操作将无限期地等待数据到达
- conn1.recv_bytes([maxlength])：接收c.send_bytes()方法发送的一条完整的字节消息。maxlength指定要接收的最大字节数。如果进入的消息，超过了这个最大值，将引发IOError异常，并且在连接上无法进行进一步读取。如果连接的另外一端已经关闭，再也不存在任何数据，将引发EOFError异常
- conn.send_bytes(buffer [, offset [, size]])：通过连接发送字节数据缓冲区，buffer是支持缓冲区接口的任意对象，offset是缓冲区中的字节偏移量，而size是要发送字节数。结果数据以单条消息的形式发出，然后调用c.recv_bytes()函数进行接收    
- conn1.recv_bytes_into(buffer [, offset])：接收一条完整的字节消息，并把它保存在buffer对象中，该对象支持可写入的缓冲区接口（即bytearray对象或类似的对象）。offset指定缓冲区中放置消息处的字节位移。返回值是收到的字节数。如果消息长度大于可用的缓冲区空间，将引发BufferTooShort异常。

代码举例：

```python

from multiprocessing import Process, Pipe

import time

def consumer(p, name):
    left_conn, right_conn = p
    left_conn.close()
    while True:
        try:
            data = right_conn.recv()
            print('%s 消费产品:%s' % (name, data))
        except EOFError:
            right_conn.close()
            break


def producer(sequence, p):
    left_conn, right_conn = p
    right_conn.close()
    for i in sequence:
        left_conn.send(i)
        print('%s 生产产品:%s' % ('c2', i))
        time.sleep(1)
    else:
        left_conn.close()


if __name__ == '__main__':
    left, right = Pipe()

    c1 = Process(target=consumer, args=((left, right), 'c1'))
    c1.start()

    seq = (i for i in range(10))
    producer(seq, (left, right))

    right.close()
    left.close()

    c1.join()
    print('主进程')

```

注意：生产者和消费者都没有使用管道的某个端点，就应该将其关闭，如在生产者中关闭管道的右端，在消费者中关闭管道的左端。如果忘记执行这些步骤，程序可能再消费者中的recv()操作上挂起。管道是由操作系统进行引用计数的，必须在所有进程中关闭管道后才能生产EOFError异常。因此在生产者中关闭管道不会有任何效果，付费消费者中也关闭了相同的管道端点。

管道可以用于双向通信，利用通常在客户端/服务器中使用的请求／响应模型或远程过程调用，就可以使用管道编写与进程交互的程序（代码略）

## Process

```python

from multiprocessing import Process

p = Process()

```

创建实例的参数：

- group参数未使用，值始终为None

- target表示调用对象，即子进程要执行的任务

- args表示调用对象的位置参数元组，args=(1,2,'hexin',)

- kwargs表示调用对象的字典，kwargs={'name':'hexin','age':18}

- name为子进程的名称

Process()由该类实例化得到的对象，表示一个子进程中的任务（尚未启动）

方法介绍：

- p.start()：启动进程，并调用该子进程中的p.run() 
- p.run()：进程启动时运行的方法，正是它去调用target指定的函数，我们自定义类的类中一定要实现该方法  
- p.terminate()：强制终止进程p，不会进行任何清理操作，如果p创建了子进程，该子进程就成了僵尸进程，使用该方法需要特别小心这种情况。如果p还保存了一个锁那么也将不会被释放，进而导致死锁
- p.is_alive()：如果p仍然运行，返回True
- p.join([timeout])：主线程等待p终止（强调：是主线程处于等的状态，而p是处于运行的状态）。timeout是可选的超时时间，需要强调的是，p.join只能join住start开启的进程，而不能join住run开启的进程

实例属性：

- p.daemon：默认值为False，如果设为True，代表p为后台运行的守护进程，当p的父进程终止时，p也随之终止，并且设定为True后，p不能创建自己的新进程，必须在p.start()之前设置
— p.name：进程的名称
- p.pid：进程的pid
- p.exitcode：进程在运行时为None、如果为–N，表示被信号N结束(了解即可)
- p.authkey：进程的身份验证键，默认是由os.urandom()随机生成的32字符的字符串。这个键的用途是为涉及网络连接的底层进程间通信提供安全性，这类连接只有在具有相同的身份验证键时才能成功（了解即可）



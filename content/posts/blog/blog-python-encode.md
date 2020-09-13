---
weight: 1000
title: "python 2/3 str unicode bytes 区别详解"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "python 2/3 str unicode bytes 区别详解"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Python, Web, Technology]
categories: [Technology技术]

lightgallery: true

toc:
  auto: false
---

str unicode bytes 在 2/3 版本中概念是不同的，需要详细理解它

<!-- more -->

## 在2/3中的输出表现

在Python2中
```py
s = 'abc' # str
u = u'abc' # unicode
b = b'abc' # bytes
print s, u, b
print type(s), type(u), type(b)
# s u b 输出
# abc abc abc
# <type 'str'> <type 'unicode'> <type 'str'>
```
输出看着是一样的，可以通过type判断


在Python3中
```py
s = 'abc' # str
u = u'abc' # unicode
b = b'abc' # bytes
print(s, u, b)
# s u b 输出
# abc abc b'anc'
# str和Unicode是一样的
```
输出可以看到有`b`前缀

Python2中，普通字符串的类型就是str，这个字符是Python3中的bytes，也就是字节

Python3中的str是Python2中的Unicode

## 意义

Python3中做出的改变是有意义的，编码和解码都发生了变化。中文字符长度判断更加清晰，方法调用更加合理(编码解码的错误信息更加清晰)

比如在要求返回bytes的地方，使用Python2可以直接运行，而Python3需要把str encode('utf-8') 成bytes

str 和 Unicode 是两种字符串类型，在2中，str存储用的是字节序列，计算机在读取的时候使用ascii解码规则，从二进制得到对应ascii码，在不对系统编码设置的情况下，默认是使用ascii编码。

编码和解码：

Python2中，字符串是字节序列，可以解码成Unicode

Python3中，统一了字符和Unicode，字符串是以字符为单位进行处理的，bytes类型是以字节为单位处理的，bytes因为是二进制，所以需要使用对应的编码才能解析，而str已经在存储的时候使用了Unicode

{% blockquote %}
2中str和bytes是一样的，Unicode是3中的str
3中的str和Unicode是一样的，bytes是2中的str
{% endblockquote %}

{% blockquote %}
python3 中 str和uncode 调用 encode('utf-8') 可以编码成 bytes
bytes 调用 decode('utf-8') 可以解码成 str

python2 中 str调用 decode('utf-8') 可以解码成 Unicode
Unicode 调用 encode('utf-8') 可以编码成 str 也就是 bytes
{% endblockquote %}

{% blockquote %}
在Python3中，对bytes类型调用str函数，会把b前缀也加上，`"b'12aa'"` 这显然不是你想要的，需要注意
{% endblockquote %}

字节的shell输出看起来是这样的 `\xae\x9e\xe6\x88\x98\` 使用十六进制表示，通过`\`分割，这里表示了5个字节，Unicode的源码表示差不多也是这样 `u'\u6492\u5a07'`，记得utf-8是变长编码，使用1到4个字节表示汉字或其它语言的字符。

## 例子

下面的例子模拟了一个web服务，使用socket编程来处理浏览器发送的请求，并返回，在Python2中是可以正常运行的
```py
# coding:utf-8

import socket

EOL1 = '\n\n'
EOL2 = '\n\r\n'
body = '''Hello, world! <h1> from the5fire 《Django企业开发实战》</h1>'''
response_params = [
    'HTTP/1.0 200 OK',
    'Date: Sat, 10 jun 2017 01:01:01 GMT',
    'Content-Type: text/plain; charset=utf-8',
    'Content-Length: {}\r\n'.format(len(body)),
    body,
]
response = b'\r\n'.join(response_params)


def handle_connection(conn, addr):
    request = ""
    while EOL1 not in request and EOL2 not in request:
        request += conn.recv(1024)
    print(request)
    conn.send(response)
    conn.close()


def main():
    # socket.AF_INET    用于服务器与服务器之间的网络通信
    # socket.SOCK_STREAM    基于TCP的流式socket通信
    serversocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    # 设置端口可复用，保证我们每次Ctrl C之后，快速再次重启
    serversocket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    serversocket.bind(('127.0.0.1', 8080))
    # 可参考：https://stackoverflow.com/questions/2444459/python-sock-listen
    serversocket.listen(1)
    print('http://127.0.0.1:8080')

    try:
        while True:
            conn, address = serversocket.accept()
            handle_connection(conn, address)
    finally:
        serversocket.close()


if __name__ == '__main__':
    main()
```

**那么问题来了，如果要迁移到Python3上要做哪些改动呢？**

1. 首先print等语法是要改的，但是这不是本次聊的重点，通过上文的总结，我们知道了Python2中str就是存储字节序列，而socket也是要求发送字节序列的，所以在Python3上，str默认是Unicode就不适用了，需要编码成bytes。

2. 在handle_connection中，把需要用到的str都编码，如果文本都是ASCII，加一个`b`前缀即可，如果包含像中文的文本，需要使用encode进行编码。调整好后程序可以运行了。

3. 细心观察，你会发现返回的文本被截断了，或者出现了乱码，这就涉及到Python2/3 `len` 函数的问题，关键在这一句 `'Content-Length: {}\r\n'.format(len(body))` 。

4. 在2中已经是字节序列了，如果用3计算的是Unicode的长度。现在中文文本是 `Django企业开发实战` 2中的bytes能计算得到长度是 24 源码为 `'Django\xe4\xbc\x81\xe4\xb8\x9a\xe5\xbc\x80\xe5\x8f\x91\xe5\xae\x9e\xe6\x88\x98'`，汉子占用18个字节，英文占用6个字节。

5. 如果是在3中，使用Unicode一个中文就占用一个长度，就没有24这么长，然后response是编码成bytes返回的，自然长度不够，出现解析错误。所以在设置Content-Length的时候要以bytes计算的长度为准。

Python3完整修改版
```py
import socket

EOL1 = b'\n\n'
EOL2 = b'\n\r\n'
body = '''Hello, world! <h1> from the5fire 《Django企业开发实战》</h1>'''
response_params = [
    'HTTP/1.0 200 OK',
    'Date: Sat, 10 jun 2017 01:01:01 GMT',
    'Content-Type: text/plain; charset=utf-8',
    'Content-Length: {}\r\n'.format(len(body.encode('utf-8'))),
    body,
]
response = '\r\n'.join(response_params)
response = response.encode('utf-8')


def handle_connection(conn, addr):
    request = b""
    while EOL1 not in request and EOL2 not in request:
        request += conn.recv(1024)
    print(request)
    conn.send(response)
    conn.close()


def main():
    # socket.AF_INET    用于服务器与服务器之间的网络通信
    # socket.SOCK_STREAM    基于TCP的流式socket通信
    serversocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    # 设置端口可复用，保证我们每次Ctrl C之后，快速再次重启
    serversocket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    serversocket.bind(('127.0.0.1', 8080))
    # 可参考：https://stackoverflow.com/questions/2444459/python-sock-listen
    serversocket.listen(1)
    print('http://127.0.0.1:8080')

    try:
        while True:
            conn, address = serversocket.accept()
            handle_connection(conn, address)
    finally:
        serversocket.close()


if __name__ == '__main__':
    main()
```
---
weight: 5500
title: "Python-socket"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Python-socket"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Python, Note]
categories: [Python]

lightgallery: true

toc:
  auto: false
---

python网络编程笔记

<!-- more -->

## EAGAIN

当客户通过Socket提供的send函数发送大的数据包时，就可能返回一个EAGAIN的错误。该错误产生的原因是由于 send 函数中的size变量大小超过了tcp_sendspace的值。tcp_sendspace定义了应用在调用send之前能够在kernel中缓存的数据量。当应用程序在socket中设置了O_NDELAY或者O_NONBLOCK属性后，如果发送缓存被占满，send就会返回EAGAIN的错误。 
## accept

accept()是在一个套接口接受的一个连接。accept（）是c语言中网络编程的重要的函数，本函数从s的等待连接队列中抽取第一个连接，创建一个与s同类的新的套接口并返回句柄

## socket.listen(backlog)

开始监听传入连接。backlog指定在拒绝连接之前，可以挂起的最大连接数量。

backlog等于5，表示内核已经接到了连接请求，但服务器还没有调用accept进行处理的连接个数最大为5
这个值不能无限大，因为要在内核中维护连接队列。

## close

close方法可以释放一个连接的资源，但是不是立即释放，如果想立即释放，那么在close之前使用shutdown方法shut_rd() -------关闭接受消息通道shut_wr()--------关闭发送消息通道shut_rdwr()-------连个通道都关闭使用：在close()之前加上shutdown(num)即可  

[shut_rd(), shut_wr(), shut_rdwr()分别代表num 为0  1  2 ]（但是测试过close()关闭，发现如果关闭后，那么accept()得到的connection就马上不能用了[提示不能在非套接字上]）
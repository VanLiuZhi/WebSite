---
weight: 1
title: "http-protocol 协议"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "http-protocol 协议"
resources:
- name: "base-image"
  src: "base-image.jpg"

tags: [Http, Note]
categories: [Web]

lightgallery: true

toc:
  auto: false
---

超文本传输协议（HTTP，HyperText Transfer Protocol)是互联网上应用最为广泛的一种网络协议。所有的WWW文件都必须遵守这个标准。

<!-- more -->

## http action

HTTP协议中GET、POST和HEAD的介绍  2008-05-10 14:15 

| Name         	                |  Description                     
| ----------------------------- |:-------------------------------: 
| GET                           | 请求指定的页面信息，并返回实体主体。 
| HEAD                          | 只请求页面的首部。 
| POST                          | 请求服务器接受所指定的文档作为对所标识的URI的新的从属实体。 
| PUT                           | 从客户端向服务器传送的数据取代指定的文档的内容。 
| DELETE                        | 请求服务器删除指定的页面。 
| OPTIONS                       | 允许客户端查看服务器的性能。 
| TRACE                         | 请求服务器在响应中的实体主体部分返回所得到的内容。 
| PATCH                         | 实体中包含一个表，表中说明与该URI所表示的原内容的区别。 
| MOVE                          | 请求服务器将指定的页面移至另一个网络地址。 
| COPY                          | 请求服务器将指定的页面拷贝至另一个网络地址。 
| LINK                          | 请求服务器建立链接关系。 
| UNLINK                        | 断开链接关系。 
| WRAPPED                       | 允许客户端发送经过封装的请求。 
| Extension-mothed              | 在不改动协议的前提下，可增加另外的方法。 

## 三次握手，四次挥手

三次握手：A向B发起连接，B收到，回一个给A，A也收到，连接确定

1. 第一次握手：建立连接时，客户端发送syn包（syn=j）到服务器，并进入SYN_SENT状态，等待服务器确认；SYN：同步序列编号（Synchronize Sequence Numbers）。
2. 第二次握手：服务器收到syn包，必须确认客户的SYN（ack=j+1），同时自己也发送一个SYN包（syn=k），即SYN+ACK包，此时服务器进入SYN_RECV状态；
3. 第三次握手：客户端收到服务器的SYN+ACK包，向服务器发送确认包ACK(ack=k+1），此包发送完毕，客户端和服务器进入ESTABLISHED（TCP连接成功）状态，完成三次握手。

完成三次握手，客户端与服务器开始传送数据。

建立连接是三次握手，释放连接是四次挥手（关闭连接）

1. 第一步，当主机A的应用程序通知TCP数据已经发送完毕时，TCP向主机B发送一个带有FIN附加标记的报文段（FIN表示英文finish）。
2. 第二步，主机B收到这个FIN报文段之后，并不立即用FIN报文段回复主机A，而是先向主机A发送一个确认序号ACK，同时通知自己相应的应用程序：对方要求关闭连接（先发送ACK的目的是为了防止在这段时间内，对方重传FIN报文段）。
3. 第三步，主机B的应用程序告诉TCP：我要彻底的关闭连接，TCP向主机A送一个FIN报文段。
4. 第四步，主机A收到这个FIN报文段后，向主机B发送一个ACK表示连接彻底释放。

## 状态码

| Name         	                |  Description                     
| ----------------------------- |:-------------------------------: 
| 2xx                           | 成功 
| 3xx                           | 重定向 
| 4xx                           | 客户端问题
| 5xx                           | 服务端问题

## get 和 post

1. get是从服务器上获取数据，post是向服务器传送数据。
2. get是把参数数据队列加到提交表单的ACTION属性所指的URL中，值和表单内各个字段一一对应，在URL中可以看到。post是通过HTTP post机制，将表单内各个字段与其内容放置在HTML HEADER内一起传送到ACTION属性所指的URL地址。用户看不到这个过程。
3. 对于get方式，服务器端用Request.QueryString获取变量的值，对于post方式，服务器端用Request.Form获取提交的数据。
4. get传送的数据量较小，不能大于2KB。post传送的数据量较大，一般被默认为不受限制。但理论上，IIS4中最大量为80KB，IIS5中为100KB。
5. get安全性非常低，post安全性较高。但是执行效率却比Post方法好。 

建议：
1、get方式的安全性较Post方式要差些，包含机密信息的话，建议用Post数据提交方式；
2、在做数据查询时，建议用Get方式；而在做数据添加、修改或删除时，建议用Post方式；

## 302响应

当ajax请求响应302的时候，得到的响应会先由浏览器去请求302响应体中headers里面的location的地址，这个地址的响应才会是ajax请求得到的东西

就是接口302的时候，浏览器最终反馈是去请求重定向地址的响应
---
weight: 1
title: "Distribute Storage"
subtitle: "Distribute Storage"
date: 2020-10-22T09:37:45+08:00
lastmod: 2020-10-22T09:37:45+08:00
draft: true
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Distribute Storage"
# featuredImagePreview: "images/base-image.jpg"
# featuredImage: "/images/base-image.jpg"
# resources:
# - name: "featured-image"
#   src: "images/base-image.jpg"

tags: 
categories: 

lightgallery: true

toc:
  # 自动展开目录
  auto: false
---

云计算--分布式存储运用

<!--more-->

## 分布式存储技术

1. GlusterFS 

GlusterFS系统是一个可扩展的网络文件系统，相比其他分布式文件系统，GlusterFS具有高扩展性、高可用性、高性能、可横向扩展等特点，并且其没有元数据服务器的设计，让整个服务没有单点故障的隐患

2. Ceph

提供块存储，对象存储，网络存储等功能，比较复杂，高性能

ceph storage class

https://www.cnblogs.com/xzkzzz/p/9848930.html

https://www.cnblogs.com/passzhang/p/12179816.html

https://www.cnblogs.com/kuku0223/p/9232858.html

https://www.cnblogs.com/happy1983/p/9246379.html

https://blog.csdn.net/liukuan73/article/details/79228180

https://blog.csdn.net/weixin_33757911/article/details/91820211

https://blog.csdn.net/mofiu/article/details/77529851?utm_medium=distribute.pc_relevant.none-task-blog-OPENSEARCH-1.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-OPENSEARCH-1.channel_param

https://blog.csdn.net/weixin_42551719/article/details/81386909?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param


ceph-deploy new
ceph-deploy install
ceph-deploy mon create
ceph-deploy gatherkeys
ceph-deploy admin

rbd 只支持 ReadWriteOnce 和 ReadOnlyMany

pv,pvc

https://blog.csdn.net/dkfajsldfsdfsd/article/details/81319735


3. NFS

4. iSCSI 计算机系统接口


https://blog.csdn.net/mingongge/article/details/100788388
https://www.jianshu.com/p/25163032f57f


安装

https://www.cnblogs.com/happy1983/p/9246379.html


博客

https://www.cnblogs.com/panchanggui/category/1238317.html
https://www.jianshu.com/p/24e1179ef563
https://blog.csdn.net/ywq935

## 网络NAT

从定义上讲，SNAT是原地址转换，DNAT是目标地址转换。区分这两个功能可以简单的由服务的发起者是谁来区分，内部地址要访问公网上的服务时，内部地址会主动发起连接，将内部地址转换成公有ip。网关这个地址转换称为SNAT. 当内部需要对外提供服务时，外部发起主动连接，路由器或着防火墙的网关接收到这个连接，然后把连接转换到内部，此过程是由带公有ip的网关代替内部服务来接收外部的连接，然后在内部做地址转换，此转换称为DNAT.主要用于内部服务对外发布。

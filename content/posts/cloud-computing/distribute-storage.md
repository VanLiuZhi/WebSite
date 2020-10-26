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


ceph-deploy new
ceph-deploy install
ceph-deploy mon create
ceph-deploy gatherkeys
ceph-deploy admin

rbd 只支持 ReadWriteOnce 和 ReadOnlyMany


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
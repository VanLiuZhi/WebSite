---
weight: 1
title: "Network"
subtitle: "Network"
date: 2020-12-23T16:09:44+08:00
lastmod: 2020-12-23T16:09:44+08:00
draft: true
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Network"
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



<!--more-->

## SSL/TLS，HTTOPS

为了保证网络通信的安全性，需要对网络上传递的数据进行加密。现在主流的加密方法就是SSL (Secure Socket Layer 安全套接字协议)，TLS (Transport Layer Security 安全传输层协议)。后者比前者要新一些，不过在很多场合还是用SSL指代SSL和TLS

HTTPS全称是HTTP over SSL，也就是通过SSL/TLS加密HTTP数据，这或许是SSL最广泛的应用

https://zhuanlan.zhihu.com/p/36981565


kong中可以配置ssl_cipher_suite的值，默认是intermediate

定义Nginx服务的TLS密码。可接受的值是modern， intermediate，old，或custom
注: 在2.x.x 版本之前的版本默认值是modern，这个级别比较高，最好还是用intermediate

字段参考 https://wiki.mozilla.org/Security/Server_Side_TLS。描述了3个级别的各浏览器协议支持情况

##  0/24

A B C 三类是我们目前使用的网段

概念	  特征	                          网络范围	      默认掩码
A类地址	第1个8位中的第1位始终为0	       0-127.x.x.x	   255.0.0.0/8
B类地址	第1个8位中的第1、2位始终为10	   128-191.x.x.x	 255.255.0.0/16
C类地址	第1个8位中的第1、2、3位始终为110	192-y.x.x.x	   255.255.255.0/24

192.168.2.0/24表示的IP范围
192.168.2.0换成32位二进制，四组，每组8位

/24 表示前24位不变，后8位由全0变化到全1的过程，也就是由“00000000”变化到“11111111”
又因为全0是子网网络地址，全1是子网广播地址，这两个地址是不分配给主机使用的。
所以有效的可分配的范围是前24位不变，后8位由“00000001”变化为“11111110”的范围
再转换回十进制就是192.168.2.1~192.168.2.254

1、如果是192的C段地址，那么，网络地址就是：192.168.1.0，地址掩码是：255.255.255.0。 
2、如果地址掩码是：255.255.0.0，那么网络地址就是：192.168.0.0。 
3、网络地址很大一部分是由地址掩码决定的。 


---
weight: 3300
title: "Network 网络"
subtitle: "Network 网络"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Network 网络"
# featuredImagePreview: "images/Kubernetes/k8s.jpg"
# featuredImage: "/images/Kubernetes/k8s.jpg"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Cloud-Native]
categories: [Cloud-Native] 

lightgallery: true

toc:
  auto: false
---

网络知识

<!--more-->

## SSL/TLS，HTTPS

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

## 网络NAT

从定义上讲，SNAT是原地址转换，DNAT是目标地址转换。区分这两个功能可以简单的由服务的发起者是谁来区分，内部地址要访问公网上的服务时，内部地址会主动发起连接，将内部地址转换成公有ip。网关这个地址转换称为SNAT. 当内部需要对外提供服务时，外部发起主动连接，路由器或着防火墙的网关接收到这个连接，然后把连接转换到内部，此过程是由带公有ip的网关代替内部服务来接收外部的连接，然后在内部做地址转换，此转换称为DNAT.主要用于内部服务对外发布

## 数字证书

公私钥成对，唯一
私钥签名，公钥验签

- 明文传输
- 对称加密，但是传输密钥是个问题(线下可以U盘拷贝，网络上只能明文传输密钥)
- 非对称，传输方公钥加密，接收方私钥解密(公钥是公开的，谁都可以伪装成传输方，带来了新问题，证明传输方)
- 传输方再创建一对公私钥，私钥签名，这个时候只有传输方的公钥才能验签。也就是传输方2，传输方3都不能伪造签名，接收方的公钥只能解开传输方的签名，不能解开其它的
- 私钥签名，公钥验签，数据正文采用散列算法得到摘要，摘要写到签名中。接收方验签，然后对正文得到摘要，验证是否和签名中的摘要一致，确保数据没被篡改



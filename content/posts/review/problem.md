---
weight: 1
title: "Java review Spring"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Java review Spring"
resources:
- name: "base-image"
  src: "base-image.jpg"

tags: [Java, Note]
categories: [Java]

lightgallery: true

toc:
  auto: false
---

review

<!-- more -->

## 一致性哈希

先了解一个概念，取模和取余，如果除数都是正整数，那么取余和取模都是一样的，求余数
当 x 和 y 的正负号一样的时候，两个函数结果是等同的；当 x 和 y 的符号不同时，rem函数结果的符号和 x 的一样，而 mod 和 y 一样

在分布式中，或者用分库分表来说明，user表存在在3台机器上，用`用户id的hashcode % 3`来决定该用户的数据存储在哪台机器上。当机器增加到4台的时候，此时的模3就不对了，造成数据混乱。而一致性哈希算法就是解决这个问题的，一致性哈希保证当分布式环境的节点增加的时候，原来请求分配到的节点还是原来的节点

```java
public static void main(String[] args) {
    // 有5个用户
    Integer a = 123456;
    Integer b = 123457;
    Integer c = 123458;
    Integer d = 123459;
    Integer e = 123460;

    // 平均分配到3台机器
    System.out.println(a.hashCode() % 3);
    System.out.println(b.hashCode() % 3);
    System.out.println(c.hashCode() % 3);
    System.out.println(d.hashCode() % 3);
    System.out.println(e.hashCode() % 3);
    System.out.println("--------------");
    // 平均分配到4台机器
    System.out.println(a.hashCode() % 4);
    System.out.println(b.hashCode() % 4);
    System.out.println(c.hashCode() % 4);
    System.out.println(d.hashCode() % 4);
    System.out.println(e.hashCode() % 4);

    //        结果
    //        0
    //        1
    //        2
    //        0
    //        1
    //                --------------
    //        0
    //        1
    //        2
    //        3
    //        0

    // 对于d,e来说，增加机器的时候就错乱了
}
```

## http无状态

这句话体现在每个请求都是独立的，第二次的请求和第一次不会有关联

## 对称加密和非对称加密

1、加密和解密过程不同

对称加密过程和解密过程使用的同一个密钥，加密过程相当于用原文+密钥可以传输出密文，同时解密过程用密文-密钥可以推导出原文。但非对称加密采用了两个密钥，一般使用公钥进行加密，使用私钥进行解密。

2、加密解密速度不同

对称加密解密的速度比较快，适合数据比较长时的使用。非对称加密和解密花费的时间长、速度相对较慢，只适合对少量数据的使用。

3、传输的安全性不同

对称加密的过程中无法确保密钥被安全传递，密文在传输过程中是可能被第三方截获的，如果密码本也被第三方截获，则传输的密码信息将被第三方破获，安全性相对较低。
非对称加密算法中私钥是基于不同的算法生成不同的随机数，私钥通过一定的加密算法推导出公钥，但私钥到公钥的推导过程是单向的，也就是说公钥无法反推导出私钥。所以安全性较高。

## 分布式事务

tcc lcn mq atomik seata

## 分布式session

Spring-boot-statrte-data-redis

## 静态化

velocity freemarker 模板引擎

## 基本服务容器配置

### nnginx

docker run -p 80:80 --name nginx \
-v /Users/liuzhi/mydata/nginx/html:/usr/share/nginx/html \
-v /Users/liuzhi/mydata/nginx/logs:/var/log/nginx  \
-d nginx:1.10

docker run -p 8091:80 --name nginx \
-v /Users/liuzhi/mydata/nginx/html:/usr/share/nginx/html \
-v /Users/liuzhi/mydata/nginx/logs:/var/log/nginx  \
-v /Users/liuzhi/mydata/nginx/conf:/etc/nginx \
-d nginx:1.10

### rabbitmq

docker run -d --name rabbitmq \
--publish 5671:5671 --publish 5672:5672 --publish 4369:4369 \
--publish 25672:25672 --publish 15671:15671 --publish 15672:15672 \
rabbitmq:3.7.15

### elasticsearch

docker run -p 9200:9200 -p 9300:9300 --name elasticsearch \
-e "discovery.type=single-node" \
-e "cluster.name=elasticsearch" \
-d elasticsearch:6.4.0

docker run -p 9201:9200 --name elasticsearch2 \
-d elasticsearch:5.2.2

docker run -p 9200:9200 -p 9300:9300 --name elasticsearch-5 \
-e "discovery.type=single-node" \
-e "cluster.name=elasticsearch" \
-d elasticsearch:5.2.2

### mongo

docker run -p 27017:27017 --name mongo-4-2-3 \
-v /Users/liuzhi/mydata/mongo-4-2-3/db:/data/db \
-d mongo:4.2.3

### Redis

docker run --name redis-5.7 --requirepass "$123qwe" -p 6377:6379 redis:5.0.7-buster

appendonly yes 开启持久化，数据存储在data目录，所以这个目录要映射出来，不然重启数据丢失，持久化就没有意义

docker run -p 6377:6379 --name redis-5.7 \
-v /Users/liuzhi/mydata/redis/data:/data \
-d redis:5.0.7-buster redis-server --appendonly yes --requirepass "qweEX123"

docker run --name redis -d redis:5.0.7-buster -p 6379:6379 --requirepass "qweEX123"

docker run --name redis-3.2 -p 6479:6379 -d redis:3.2 

### mysql 启动容器时区配置

docker run --name mysql2 -p 3506:3306 \ 
-e MYSQL_ROOT_PASSWORD=test123456 \ 
-e TZ=Asia/Shanghai -d mysql:5.7 \ 
--default-time_zone='+8:00'

docker run -p 5506:3306 --name mysql-5.7-docker \
-v /data/mysql-docker/log:/var/log/mysql \
-v /data/mysql-docker/data:/var/lib/mysql \
-v /data/mysql-docker/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=root123456  \
-d hub.eos.h3c.com/base/mysql:5.7.24 \
--default-time_zone='+8:00'

可以用TZ改时区，默认是UTC TZ=Asia/Shanghai 改成CST
记得一定加default-time_zone

### nacos

docker  run \
--name nacos2 -d \
-p 8848:8848 \
--privileged=true \
--restart=always \
-e JVM_XMS=256m \
-e JVM_XMX=256m \
-e MODE=standalone \
-e PREFER_HOST_MODE=hostname \
hub.eos.h3c.com/nacos/nacos-server:1.3.1

## 常用方法

`public boolean equalsIgnoreCase(String anotherString)` 字符串对象调用，和一个字符串比较，不区分大小写


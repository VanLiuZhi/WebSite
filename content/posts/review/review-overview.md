---
weight: 6600
title: "Java review 散问题"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Java review 散问题"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Java, Note]
categories: [Clear-Mind]

lightgallery: true

toc:
  auto: false
---

review

<!-- more -->

## 待处理

接口幂等性：
1. 全局唯一ID
2. 去重表
3. 状态机

创建多线程有几种方式？

UDP是基于报文发送的，从UDP的帧结构可以看出，在UDP首部采用了16bit来指示UDP数据报文的长度，因此在应用层能很好的将不同的数据报文区分开，从而避免粘包和拆包的问题。

redission实现的分布式锁，同学们可以看一下，解决了传统分布式锁续约的问题。

redis做分布式锁，如果是集群，主节点在同步到从节点前挂掉，如何保证锁的互斥性？这个面试的时候经常问，面试官想考察的点是redlock(红锁)

1.用zookeeper创建一个临时节点代表锁，比如在/exlusive_lock下创建临时顺序子节点/exlusive_lock/lock。
2.所有客户端争相创建此节点，但只有一个客户端创建成功。
3.创建成功代表获取锁成功，此客户端执行业务逻辑
4.未创建成功的客户端，监听/exlusive_lock变更
5.获取锁的客户端执行完成后，删除/exlusive_lock/lock，表示锁被释放
6.锁被释放后，其他监听/exlusive_lock变更的客户端得到通知，再次争相创建临时子节点/exlusive_lock/lock。此时相当于回到了第2步。

## 微服务

ServiceMesh(服务网格)

RMI（Remote Method Invocation）远程方法调用。能够让在客户端Java虚拟机上的对象像调用本地对象一样调用服务端java 虚拟机中的对象上的方法。使用代表：EJB

RPC（Remote Procedure Call Protocol）远程过程调用协议，通过网络从远程计算机上请求调用某种服务。它是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。 使用代表：Dubbo

CAP原则
一致性（Consistency）、可用性（Availability）、分区容错性（Partition tolerance）

BASE是Basically Available（基本可用）、Soft state（软状态）和Eventually consistent（最终一致性）三个短语的简写

ACID
原子性（atomicity)
一致性（consistency)
隔离性（isolation）
持久性（durability）


1. 传统集中式代理:基于反向代理的中心化架构
2. 客户端嵌入式代理:嵌入应用内部的去中心化架构
3. 主机独立进程代理 :基于独立代理进程的Service Mesh架构

Nginx7层，硬件4层

## 容器类

//一般讨论集合类无非就是。这里的两种数组类型更是如此
// 1底层数据结构
// 2增删改查方式
// 3初始容量，扩容方式，扩容时机。
// 4线程安全与否
// 5是否允许空，是否允许重复，是否有序 

普通的集合类，迭代的时候，其它线程修改了容器，会直接触发fail-fast
线程安全的集合类才能在迭代的时候修改容器


## 常用包

<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.7</version>
</dependency>
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>23.0</version>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.16.18</version>
    <optional>true</optional>
</dependency>
<!--A set of annotations used for code inspection support and code documentation.-->
<dependency>
    <groupId>org.jetbrains</groupId>
    <artifactId>annotations</artifactId>
    <version>18.0.0</version>
    <scope>compile</scope>
</dependency>
<!--security-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<!--配置文件转换支持，主要是旧体系框架向Spring boot转换-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>

# problem 遇到的问题大汇总

1. 泊松分布
泊松分布的参数λ是单位时间(或单位面积)内随机事件的平均发生次数。 泊松分布适合于描述单位时间内随机事件发生的次数。

2. concurrentHashMap锁分段的思想是怎么做的？

3. CAS算法是什么？
参考 https://zhuanlan.zhihu.com/p/93418208

3. 下面的main中，调用方法A创建了线程，然后执行方法B，谁先执行？

```java
public static void main(String[] args) throws Exception {
        instance = new Instance();
        Thread thread = new Thread(instance);
        thread.start(); // run 中执行方法A
        instance.BMethod();
    }
```
正常情况下，方法B先执行。主线程启动起来，因为创建线程等的资源消耗，所以主线程会先执行，然后才到子线程，也不排斥代码复杂的情况下，子线程准备就绪，先main线程执行

4. 操作系统中，CPU竞争有很多种策略。Unix系统使用的是时间片算法，而Windows则属于抢占式的

5. Thread.Sleep() Thread.Sleep(0)

sleep是当前线程告诉操作系统放弃cpu资源竞争多长时间，参数0的话，触发操作系统立刻重新进行一次CPU竞争，让其它线程有机会获取到CPU资源

6. 如何用jstack分析程序？
https://www.jianshu.com/p/dee9d02f11b3

7. 如何用Java Dump分析堆内存？

---
weight: 4400
title: "Java Redis"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Java Redis"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Java, Note]
categories: [Java-MicroServices]

lightgallery: true

toc:
  auto: false
---

## 概念：

Jedis：是Redis的Java实现客户端，提供了比较全面的Redis命令的支持，

Redisson：实现了分布式和可扩展的Java数据结构。

Lettuce：高级Redis客户端，用于线程安全同步，异步和响应使用，支持集群，Sentinel，管道和编码器。

## 优点：

Jedis：比较全面的提供了Redis的操作特性

Redisson：促使使用者对Redis的关注分离，提供很多分布式相关操作服务，例如，分布式锁，分布式集合，可通过Redis支持延迟队列

Lettuce：主要在一些分布式缓存框架上使用比较多

## 可伸缩：

Jedis：使用阻塞的I/O，且其方法调用都是同步的，程序流需要等到sockets处理完I/O才能执行，不支持异步。Jedis客户端实例不是线程安全的，所以需要通过连接池来使用Jedis。

Redisson：基于Netty框架的事件驱动的通信层，其方法调用是异步的。Redisson的API是线程安全的，所以可以操作单个Redisson连接来完成各种操作

Lettuce：基于Netty框架的事件驱动的通信层，其方法调用是异步的。Lettuce的API是线程安全的，所以可以操作单个Lettuce连接来完成各种操作

## redis的数据类型

总共5种

1. String

2. hash

hset key filed value 设置值
hget key filed 　获取值

相当于一个key对应一个map

3. list

链表(双向链表)，增删快，提供了操作某一段元素的API。适用于：最新消息排行等功能；消息队列

4. set

集合。哈希表实现，元素不重复，为集合提供了求交集、并集、差集等操作。适用于：共同好友；利用唯一性，统计访问网站的所有独立ip；> 好友推荐时，根据tag求交集，大于某个阈值就可以推荐

5. sorted set

有序集合。将Set中的元素增加一个权重参数score，元素按score有序排列。数据插入集合时，已经进行天然排序。适用于：排行榜；带权重的消息队列

其它类型:

bitmap：更细化的一种操作，以bit为单位
hyperloglog：基于概率的数据结构
Geo：地理位置信息储存起来，并对这些信息进行操作

## StringRedisTemplate与RedisTemplate的区别?

两者的关系是StringRedisTemplate继承RedisTemplate。

两者的数据是不共通的；也就是说StringRedisTemplate只能管理StringRedisTemplate里面的数据
RedisTemplate只能管RedisTemplate中的数据。

SDR默认采用的序列化策略有两种，一种是String的序列化策略，一种是JDK的序列化策略。
StringRedisTemplate默认采用的是String的序列化策略，保存的key和value都是采用此策略序列化保存的。
RedisTemplate默认采用的是JDK的序列化策略，保存的key和value都是采用此策略序列化保存的。

总结：
当你的redis数据库里面本来存的是字符串数据或者你要存取的数据就是字符串类型数据的时候，那么你就使用StringRedisTemplate即可，但是如果你的数据是复杂的对象类型，而取出的时候又不想做任何的数据转换，直接从Redis里面取出一个对象，那么使用RedisTemplate是更好选择

## 各个序列化类

当我们的数据存储到Redis的时候，我们的键（key）和值（value）都是通过Spring提供的Serializer序列化到数据库的。RedisTemplate默认使用的是JdkSerializationRedisSerializer，StringRedisTemplate默认使用的是StringRedisSerializer。

Spring Data JPA为我们提供了下面的Serializer：GenericToStringSerializer、Jackson2JsonRedisSerializer、JacksonJsonRedisSerializer、JdkSerializationRedisSerializer、OxmSerializer、StringRedisSerializer。

序列化方式对比：

- JdkSerializationRedisSerializer: 使用JDK提供的序列化功能。 优点是反序列化时不需要提供类型信息(class)，但缺点是需要实现Serializable接口，还有序列化后的结果非常庞大，是JSON格式的5倍左右，这样就会消耗redis服务器的大量内存。

- Jackson2JsonRedisSerializer： 使用Jackson库将对象序列化为JSON字符串。优点是速度快，序列化后的字符串短小精悍，不需要实现Serializable接口。但缺点也非常致命，那就是此类的构造函数中有一个类型参数，必须提供要序列化对象的类型信息(.class对象)。 通过查看源代码，发现其只在反序列化过程中用到了类型信息。 
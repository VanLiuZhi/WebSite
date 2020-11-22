---
weight: 9900
title: "Redis在web开发中的运用"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Redis在web开发中的运用"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Web, Note, DataBase, Python]
categories: [Middleware(中间件)]

lightgallery: true

toc:
  auto: false
---

REmote DIctionary Server(Redis) 是一个由Salvatore Sanfilippo写的key-value存储系统。
Redis是一个开源的使用ANSI C语言编写、遵守BSD协议、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。

<!-- more -->

## Database Number

Redis 使用 DB number 实现类似关系型数据库中 schema 的功能。不同 DB number 表示的数据库是隔离的，但是目前只能使用数字来表示一个数据库，Ubuntu 默认的配置文件配置了16个数据库，DB number 是从0开始的，并且默认连接0号数据库。

`redis-cli -n <dbnumber>` 连接指定数据库

## 在docker-compose中使用Redis

进入交互环境 `docker-compose exec redis redis-cli`
清除缓存 `docker-compose exec redis redis-cli flushall`

其它命令类似

## redis 和 Python

一般简单使用redis，只需要安装redis的Python包，如果是使用Redis作为数据库的话，可以使用 `walrus(海象)` 来处理，这样就可以设置模型，序列化返回数据，如果是一般的设置键值，查询的操作，使用原生的redis包即可，walrus是对redis包的扩展

## python 操作 redis

首先要创建连接`rdb = Database.from_url(REDIS_URL)`

使用rdb提供的方法去设置key_value数据即可，一个很重要的概念就是数据类型，不同的数据类型要使用对应的方法来操作，要使用什么数据类型取决于业务。

无论是什么类型的数据，它必须有唯一key，而value就是数据类型

1. string 最基本的数据类型，使用极其简单，调用set，get方法，也可以使用对对象进行序列化的方式来存储类似dict，list这样的数据结构
2. Hash 用来存储Python的dict类型数据
3. List 同样用来存储Python的list类型数据
4. set 存储集合
5. 有序集合 zset，这种类型的数据在存储的时候需要制定排序值

redis包的各个类型的数据结构都要使用对应的操作方法，根据我使用的经验来看，文档很少，官方文档直接指向Redis数据库的操作，要是看源码无法理解，可以直接看官方命令操作Redis数据库

## 缓存命中率

这是一个很重要的概念，如果你的系统不谈这个概念，那么说明你的访问量还是很小的。缓存服务作为Web架构的核心部分，充当着很重要的角色，那么什么是缓存命中率呢？这里就涉及到为什么会有没访问到缓存的情况，假如有一个接口A获取特定的数据，这个接口是被缓存的，无需通过数据库，但是你的数据总有更新的时候，如果需要更新数据了，就要重建缓存，从原始数据库取，就会出现没有访问到缓存的情况。

缓存粒度越大，缓存命中率就会随之降低。

## Redis查询当前库有多少个 key

info可以看到所有库的key数量

dbsize则是当前库key的数量
keys *这种数据量小还可以，大的时候可以直接搞死生产环境

dbsize和keys *统计的key数可能是不一样的，如果没记错的话，keys *统计的是当前db有效的key，而dbsize统计的是所有未被销毁的key（有效和未被销毁是不一样的，具体可以了解redis的过期策略）

redis-cli -a hccx\!0528  直接输入密码进入
redis-cli -a qweEX123

redis-cli -n 1   进去之后 auth  输入密码

## 集群

Redis Cluster是一组Redis实例，旨在通过对数据库进行分区来扩展数据库，从而使其更具弹性。
群集中的每个成员（无论是主副本还是辅助副本）都管理哈希槽的子集。如果主机无法访问，则其从机将升级为主机。在由三个主节点组成的最小Redis群集中，每个主节点都有一个从节点（以实现最小的故障转移），每个主节点都分配有一个介于0到16383之间的哈希槽范围。节点A包含从0到5000的哈希槽，节点B从5001到10000，节点C从10001到16383

## 集群CTL镜像

kubectl run -it ubuntu --image=ubuntu --restart=Never -n public-service bash
kubectl run -it ubuntu --image=hub.eos-ts.h3c.com/ubuntu --restart=Never -n redis bash
kubectl run -it ubuntu --image=hub.eos-ts.h3c.com/redis-trib:v1 --restart=Never -n redis bash

```
cat > /etc/apt/sources.list << EOF
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security multiverse
EOF
```

代理设置

export http_proxy="http://Lys3415:destinymy6EX@proxy02.h3c.com:8080"
export https_proxy="https://Lys3415:destinymy6EX@proxy02.h3c.com:8080"

apt-get update
apt-get install -y libncursesw5 libreadline6 libtinfo5 --allow-remove-essential
apt-get install -y libpython2.7-stdlib python2.7 python-pip redis-tools dnsutils
pip install --upgrade pip
pip install redis-trib==0.5.1

修改redis-trib源码

从pip上下载源码并解压，此时有 redis-trib-0.5.0 的两层目录，修改源码后，把两层 redis-trib-0.5.0 目录删了一层，使用 7-zip工具，添加压缩包选择tar压缩，再压缩一次选择gzip压缩。这样就得到修改源码后的安装包

源码地址：

https://github.com/projecteru/redis-trib.py

## k8s 部署redis

kubectl run -it ubuntu --image=hub.eos-ts.h3c.com/redis-trib:v1 --restart=Never -n public-service bash

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: redis-conf
data:
  redis.conf: |
    bind 0.0.0.0
    port 6379
    requirepass 123456
    pidfile .pid
    appendonly yes
    cluster-config-file nodes-6379.conf
    pidfile /data/middleware-data/redis/log/redis-6379.pid
    cluster-config-file /data/middleware-data/redis/conf/redis.conf
    dir /data/middleware-data/redis/data/
    logfile "/data/middleware-data/redis/log/redis-6379.log"
    cluster-node-timeout 5000
    protected-mode no
```

## 查看集群信息

redis-cli -c

auth 身份认证

CLUSTER NODES #列出节点信息
CLUSTER INFO  #集群状态

set key value
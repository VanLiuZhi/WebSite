---
weight: 1
title: "RocketMQ"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "RocketMQ"
resources:
- name: "base-image"
  src: "base-image.jpg"

tags: [Java, Note]
categories: [Java]

lightgallery: true

toc:
  auto: false
---

title: RocketMQ


<!-- more -->

## 安装

这个安装还是特别容易的，下载对应程序，配置参数，启动，和Tomcat差不多

遇到的问题:

- 网卡绑定错误

虚拟机测试很容易遇到这个问题，手动设置ip解决

RocketMQ中broker配置brokcerIP1和brokerIP2的作用
brokerIP1 当前broker监听的IP
brokerIP2 存在broker主从时，在broker主节点上配置了brokerIP2的话,broker从节点会连接主节点配置的brokerIP2来同步。

默认不配置brokerIP1和brokerIP2时，都会根据当前网卡选择一个IP使用，当你的机器有多块网卡时，很有可能会有问题

```s
brokerIP1=10.10.10.3
brokerIP2=10.10.10.3
```

- Java环境问题

在runserve.sh文件中，会去判断当前是否有Java环境，然后系统会去设置，由于默认的设置执行的不一定是你自己机器的环境变量，所以需要手动修改

- JVM内存

默认的内存配置是特别大的，测试可以改小一点

## 命令

1、rocketmq的启动
进入rocketMQ解压目录下的bin文件夹
启动namesrv服务：nohup sh bin/mqnamesrv &
日志目录：{rocketMQ解压目录}/logs/rocketmqlogs/namesrv.log

启动broker服务：nohup sh bin/mqbroker &
日志目录：{rocketMQ解压目录}/logs/rocketmqlogs/broker.log

以上的启动日志可以在启动目录下的nohub.out中看到

2、rocketmq服务关闭

关闭namesrv服务：sh bin/mqshutdown namesrv
关闭broker服务 ：sh bin/mqshutdown broker

## 配置参数

```s
# 主节点
brokerClusterName=rocketmq-cluster
brokerName=broker-a
brokerId=0
namesrvAddr=rocketmq1:9876
defaultTopicQueueNums=4
autoCreateTopicEnable=true
autoCreateSubscriptionGroup=true
listenPort=10911
deleteWhen=04
fileReservedTime=120
mapedFileSizeCommitLog=1073741824
mapedFileSizeConsumeQueue=300000
destroyMapedFileIntervalForcibly=120000
redeleteHangedFileInterval=120000
diskMaxUsedSpaceRatio=88
storePathRootDir=/app/svr/rocketmq/data/store
storePathCommitLog=/app/svr/rocketmq/data/store/commitlog
maxMessageSize=65536
flushCommitLogLeastPages=4
flushConsumeQueueLeastPages=2
flushCommitLogThoroughInterval=10000
flushConsumeQueueThoroughInterval=60000
checkTransactionMessageEnable=false
sendMessageThreadPoolNums=128
pullMessageThreadPoolNums=128
brokerRole=SYNC_MASTER
flushDiskType=ASYNC_FLUSH

# 从节点
brokerClusterName=rocketmq-cluster
brokerName=broker-a
brokerId=1 
namesrvAddr=rocketmq1:9876
defaultTopicQueueNums=4
autoCreateTopicEnable=true
autoCreateSubscriptionGroup=true
listenPort=10922
deleteWhen=04
fileReservedTime=120
mapedFileSizeCommitLog=1073741824
mapedFileSizeConsumeQueue=300000
destroyMapedFileIntervalForcibly=120000
redeleteHangedFileInterval=120000
diskMaxUsedSpaceRatio=88
storePathRootDir=/app/svr/rocketmq/data/store2
storePathCommitLog=/app/svr/rocketmq/data/store2/commitlog
maxMessageSize=65536
flushCommitLogLeastPages=4
flushConsumeQueueLeastPages=2
flushCommitLogThoroughInterval=10000
flushConsumeQueueThoroughInterval=60000
checkTransactionMessageEnable=false
sendMessageThreadPoolNums=128
pullMessageThreadPoolNums=128
brokerRole=SLAVE
flushDiskType=ASYNC_FLUSH


sudo mkdir /app/svr/rocketmq/data
sudo mkdir -p /app/svr/rocketmq/data/store2/commitlog
sudo mkdir /app/svr/rocketmq/data/store2/consumequeue
sudo mkdir /app/svr/rocketmq/data/store2/index

mkdir -p /app/svr/rocketmq/logs
cd /app/svr/rocketmq/conf
sed -i 's#${user.home}#/app/svr/rocketmq#g' *.xml

sh mqbroker -c /app/svr/rocketmq/conf/2m-2s-sync/broker-a.properties > /dev/null 2>&1 &
nohup sh mqbroker -c /app/svr/rocketmq/conf/2m-2s-sync/broker-a.properties &

sh mqadmin clusterlist -n 192.168.90.101:9876

brokerIP1=192.168.90.101
brokerIP2=192.168.90.101
```

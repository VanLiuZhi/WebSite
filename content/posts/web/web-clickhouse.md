---
weight: 5500
title: "clickhouse"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "clickhouse"
authorLink: "https://www.liuzhidream.com"
description: "clickhouse"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Python, Note]
categories: [Python]

lightgallery: true

toc:
  auto: false
---

## docker 部署

`/etc/clickhouse-server/` 这个是配置文件目录，先不映射这个目录，启动容器，把配置文件拷贝出

每一个实例只能存储一个分片或者分片的副本，构建3分片高可用集群需要6个实例，即3分片3副本。配置文件参考附件

```s
docker run -d \
--name clickhouse \
--ulimit nofile=262144:262144 \
--volume=/data/cs/clickhouse/:/var/lib/clickhouse  \
--volume=/data/cs/clickhouse-server/:/etc/clickhouse-server/ \
--add-host server01:10.64.200.71 \
--add-host server02:10.64.200.72 \
--add-host server03:10.64.200.73 \
--hostname $HOSTNAME \
--network host \
-p 9000:9000 \
-p 8123:8123 \
-p 9009:9009 \
hub.eos.h3c.com/yandex/clickhouse-server:21.1.6.13


docker run -d \
--name clickhouse2 \
--ulimit nofile=262144:262144 \
--volume=/data/cs1/clickhouse/:/var/lib/clickhouse  \
--volume=/data/cs1/clickhouse-server/:/etc/clickhouse-server/ \
--add-host server01:10.64.200.71 \
--add-host server02:10.64.200.72 \
--add-host server03:10.64.200.73 \
--hostname $HOSTNAME \
--network host \
-p 9001:9000 \
-p 8124:8123 \
-p 9010:9009 \
hub.eos.h3c.com/yandex/clickhouse-server:21.1.6.13
```

## 建表测试

```sql
create table if not exists eos_appstatistics.tb_record on cluster eos_ck_cluster
(
         id String,
         application_id String,
         modular_id String,
         function_id String,
         url String,
         user_id String
)
engine =MergeTree()
ORDER BY (record_time, record_time_hour, application_id, ip, user_id, cityHash64(id))
SAMPLE BY cityHash64(id)
SETTINGS index_granularity = 8192;


CREATE TABLE IF NOT EXISTS eos_appstatistics.tb_record_dist ON cluster eos_ck_cluster
AS eos_appstatistics.tb_record
ENGINE = Distributed(eos_ck_cluster,eos_appstatistics,tb_record,rand());
```

## zk 部署

```yaml
version: '3.1'
services:
  zoo1:
    image: zookeeper:3.5
    restart: always
    container_name: zoo1
    ports:
      - 2181:2181
    volumes:
      - /root/zookeeper/zoo1/data:/data
      - /root/zookeeper/zoo1/datalog:/datalog
    environment:
      ZOO_MY_ID: 1
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181
  zoo2:
    image: zookeeper:3.5
    restart: always
    container_name: zoo2
    ports:
      - 2182:2181
    volumes:
      - /root/zookeeper/zoo2/data:/data
      - /root/zookeeper/zoo2/datalog:/datalog
    environment:
      ZOO_MY_ID: 2
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181
  zoo3:
    image: zookeeper:3.5
    restart: always
    container_name: zoo3
    ports:
      - 2183:2181
    volumes:        
      - /root/zookeeper/zoo3/data:/dada        
      - /root/zookeeper/zoo3/datalog:/datalog  
    environment:
      ZOO_MY_ID: 3 
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181
```

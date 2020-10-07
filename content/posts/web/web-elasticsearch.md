---
weight: 9900
title: "Elasticsearch 全文搜索解引擎在 Web 开发中的运用"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Elasticsearch 全文搜索解引擎在 Web 开发中的运用"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Python, Note]
categories: [Middleware(中间件)]

lightgallery: true

toc:
  auto: false
---

搜索服务是Web开发中非常常见的服务，一般比较基本的会直接访问到数据库，但是如果要查询多个数据库才能得到搜索结果，这无疑加大了数据库的负担，
而且频繁的访问数据库容易成为应用的瓶颈。可以使用elasticsearch来提供搜索服务，降低数据库的压力。

<!-- more -->

## 概述

elasticsearch是基于Java的，所以在安装的时候可能需要Java环境，使用Docker来运行服务会比较简单。Elastic 本质上是一个分布式数据库，允许多台服务器协同工作，每台服务器可以运行多个 Elastic 实例。单个 Elastic 实例称为一个节点（node）。一组节点构成一个集群（cluster）。下面我们以单服务的形式在Docker中来使用它，为Web应用提供搜索服务支持。另外elasticsearch的操作其实是向服务发生请求。

## base

通俗的将Elastic就是一个数据库，它以索引Index为顶层单位，一个Index可以理解为一个数据库，其中里面的单条记录称为Document，同一个 Index 里面的 Document，不要求有相同的结构（scheme），但是最好保持相同，这样有利于提高搜索效率。

关于Tyoe：

1. Document 可以分组，比如weather这个 Index 里面，可以按城市分组（北京和上海），也可以按气候分组（晴天和雨天）。这种分组就叫做 Type，它是虚拟的逻辑分组，用来过滤 Document。

2. 不同的 Type 应该有相似的结构（schema），举例来说，id字段不能在这个组是字符串，在另一个组是数值。这是与关系型数据库的表的一个区别。性质完全不同的数据（比如products和logs）应该存成两个 Index，而不是一个 Index 里面的两个 Type（虽然可以做到）。

## 在Python中使用elasticsearch

首先，安装elasticsearch，这里我使用Docker来运行，默认端口为9200。然后安装elasticsearch-dsl，不要使用elasticsearch，因为elasticsearch是底层服务，我们需要使用像elasticsearch-dsl这样的高级封装库，它能更方便的编写和操作查询。

```python

from elasticsearch_dsl import Document, Integer, Text, Boolean, Q, Keyword, SF, Date
from elasticsearch_dsl.connections import connections

connections.create_connection(hosts=ES_HOSTS) # 连接服务

class Item(Document):
    id = Integer()
    title = Text()
    kind = Integer()
    content = Text()
    can_show = Boolean()
    created_at = Date()
    tags = Text(fields={'raw': Keyword()})

    class Index:
        name = 'test' # 指定Index的名称

    @classmethod
    def add(cls, item):
        obj = cls(**kwargs)
        obj.save()
        return obj
```

## bulk 进行批量操作

bulk相关的API可以在单个请求中一次执行多个操作(index,udpate,create,delete)，这比执行多次操作性能要好。相关API在使elasticsearch.helpers中，helpers是bulk的帮助程序，是对bulk的封装。

有三种方式bulk（），streaming_bulk（），parallel_bulk（）

```py
items = []
search = Items.search()
objects = ({
            '_op_type': 'update',
            '_id': f'{doc.id}_{doc.kind}',
            '_index': 'test',
            '_type': 'doc',
            '_source': doc.to_dict()
        } for doc in items)
client = connections.get_connection(search._using)
rs = list(parallel_bulk(client, objects,
                        chunk_size=500))
```

举例了parallel_bulk的用法，这里需要迭代才会执行，使用list

## 版本总结

es 7.0 对应  head 插件 5.x  这是我上次的坑 如果不用5.x 的head 会连不上es

es7可以不指定index来搜索了，默认index是_doc，而7一下还是要通过index来区别数据

es5可以指定多个type在一个index中，不建议这么做，es7不能再指定多个type

可以 用*模糊index查询 ，比如 `abc-*` * 可以是用户表示，这样就全文查询所有用户了

## restful 接口实例

就是用_search 然后带上请求体，请求体由es特定的语法组成
es7可以不指定index来搜索了，默认index是_doc，而7一下还是要通过index来区别数据

- 查询所有的index

curl -X GET 'http://125.124.23.121:9200/_cat/indices?v'
curl -X GET 'http://127.0.0.1:9200/_cat/indices?v'

- 查询所有的mapping，type

curl 'localhost:9200/_mapping?pretty=true'

- 查询所有记录

curl 'localhost:9200/traffic-zjyzsd/edrtraffic/AW5oGoTioA_5X_ZtZ_53?pretty=true'
curl 'localhost:9200/my_index/my_index2/_search?pretty=true'  /索引/type   不指定索引就是查询全部类型

1. 使用match匹配，accounts索引下，type是person， 字段 是 desc ，字段内容包含 “软件的”

$ curl 'localhost:9200/accounts/person/_search'  -d '
{
  "query" : { "match" : { "desc" : "软件" }}
}'

2. 默认是10条记录，from指定开始位置，默认是0，从头开始，以此做分页

$ curl 'localhost:9200/accounts/person/_search'  -d '
{
  "query" : { "match" : { "desc" : "管理" }},
  "from": 1,
  "size": 1
}'

3. Elastic 认为它们是or关系，就是满足一个就行

$ curl 'localhost:9200/accounts/person/_search'  -d '
{
  "query" : { "match" : { "desc" : "软件 系统" }}
}'

4. 如果要执行多个关键词的and搜索，必须使用布尔查询。

$ curl 'localhost:9200/accounts/person/_search'  -d '
{
  "query": {
    "bool": {
      "must": [
        { "match": { "desc": "软件" } },
        { "match": { "desc": "系统" } }
      ]
    }
  }
}'

5. 根据id查询记录

curl 'localhost:9200/accounts/person/1?pretty=true'

- 导入数据

curl -H 'Content-Type: application/x-ndjson'  -s -XPOST localhost:9200/_bulk --data-binary edr-test.json

- 多条件查询

curl 'localhost:9200/edr-*/edrdoc/_search?pretty=true'  -d '{"size":2, "query": {"bool": {"must": [{ "term": { "eventId" : "6002" } },{"term": { "kid":"KB3000483" } }]}}}'

- 单条件查询，match和term

curl 'localhost:9200/edr-*/edrdoc/_search?pretty=true' -d '{"query" : { "match" : { "kid" : "KB3000483" }}}'
curl 'localhost:9200/edr-*/edrdoc/_search?pretty=true' -d '{"query" : { "term" : { "eventId" : "6002" }}}'

- 时间范围查询

curl 'localhost:9200/edr-*/edrdoc/_search?pretty=true' -d '{"size":10, "query": {"range": { "uploadtime": { "gte":1578436338552, "lte": 1578436338559} } }}'

- 删除数据，条件删除和匹配删除

curl -X Delete 'localhost:9200/edr-usertest/edrdoc/AXEvV--mxoIOfpIMeV0J

curl -X POST 'localhost:9200/edr-usertest/edrdoc/_delete_by_query?pretty=true' -d '{"query": { "term" : { "id" : "9856519d-cd0b-486f-b8a8-17363159315f" }}}'

## 批量删除

直接在kibana 上执行 DELETE xx_sw* 索引可以用通配符，用来统一删除数据
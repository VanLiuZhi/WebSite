---
weight: 9900
title: "ES存储原理与架构"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "ES存储原理与架构"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Technology, Docker, Note]
categories: [Middleware(中间件)]

lightgallery: true

toc:
  auto: false
---

## 概述

通过了解存储原理才能更好的解决出现的问题

## 单索引最大文档数

2,147,483,519 (= Integer.MAX_VALUE - 128)

## 关于监控索引的处理

系统会创建一些监控数据 `.monitoring-es-7-*` 加上日期作为索引名称，如果用了kibana也会创建
默认设置存储7天，可以修改，有些时候当日的文件会达到好几个G，7天的话也是对存储空间的一种消耗

下文有比较详细的介绍 https://blog.csdn.net/qq_36317804/article/details/103288642

监控信息注意是: kibana各种请求日志，es各种集群状态等

调整存储时间

```json
PUT _cluster/settings
{"persistent": {"xpack.monitoring.history.duration":"1d"}}
```

设置需要采集的监控索引

```json
PUT _cluster/settings
{"persistent": {"xpack.monitoring.collection.indices": "*,-.*"}}
```

要注意普通版本无法调整这个时间间隔

## 分片/副本

分片设置就不能改动了(猜测和一致性算法有关)，副本可以动态修改

Elasticsearch 对数据的隔离和迁移是以分片为单位进行的，分片太大，会加大迁移成本

单个 Lucene 越大(也就是分片)，索引会越大，查询的操作成本自然要越高，IO 压力越大，自然会影响查询体验

1）分片数 = 索引大小/分片大小经验值 30GB
2）分片数建议和节点数一致

设计的时候1）、2）两者权衡考虑+rollover 动态更新索引结合

副本: 单机架构设置副本无效，副本要大于等于1
副本设置为 0，长久以后会出现——数据写入向指定机器倾斜的情况
副本用来应对不断攀升的吞吐量以及确保数据的安全性(请求落在副本上也能完成，提高了吞吐量。副本冗余了数据，节点宕机保证数据完整性)
当主分片的节点故障时，会把副本选为主分片，es不会把主分片和副本都落在一个节点上，这样就无法保证高可用

每个分片都是独立的Lucene索引，更多的分片意味着在单个较小的索引上进行操作(特别是数据索引操作)会比较高效

如果跨分片查询，还要组合数据，虽然用户不用关注实现，但是带来了一定的开销

### 规划

考虑到高可用和吞吐量，最好就是每个节点都存储了索引对应的分片，副本数也可以

举例：索引使用10个分片，那么10个分片散在节点上，最大节点数就是10。当然还要加上副本数，副本为2，那么就是10x2=20个副本，加上分片共计30
最大节点数是30个

当然这不是最佳规划，只是一种中间策略，权衡了资源和性能分配(其实资源分配已经很多了，10个分片用了30个节点)

## 堆内存的重要性

为了能够每个节点存储尽可能多的数据，重要的是尽可能多地管理堆内存使用量并减少其开销。 节点拥有的堆空间越多，它可以处理的数据和分片越多

每个分片都有数据需要保存在内存中并使用堆空间。这包括在分片级别保存信息的数据结构，也包括在段级别的数据结构，以便定义数据驻留在磁盘上的位置。 这些数据结构的大小不是固定的，并且将根据用例而有所不同

提示1：小分片会导致小分段(segment)，从而增加开销。目的是保持平均分片大小在几GB和几十GB之间。对于具有基于时间的数据的用例，通常看到大小在20GB和40GB之间的分片

提示2：由于每个分片的开销取决于分段数和大小，通过强制操作迫使较小的段合并成较大的段可以减少开销并提高查询性能。一旦没有更多的数据被写入索引，这应该是理想的。请注意，这是一个消耗资源的（昂贵的）操作，较为理想的处理时段应该在非高峰时段执行。

提示3：可以在集群节点上保存的分片数量与您可用的堆内存大小成正比，但这在Elasticsearch中没有的固定限制(7.X版本开始限制单节点能存在的分片数)。 一个很好的经验法则是：确保每个节点的分片数量保持在低于每1GB堆内存对应集群的分片在20-25之间。 因此，具有30GB堆内存的节点最多可以有600-750个分片，但是进一步低于此限制，您可以保持更好。 这通常会帮助群体保持处于健康状态

## 分片的大小如何影响性能？

在Elasticsearch中，每个查询在每个分片的单个线程中执行。然而，可以并行处理多个分片，并可以在相同分片上执行多个查询和聚合。

【小分片的利弊】这意味着，在不涉及高速缓存时，最小查询延迟将取决于数据、查询的类型、分片的大小。查询大量小分片将使得每个分片的处理速度更快，但是随着更多的任务需要按顺序排队和处理，它不一定要比查询较小数量的更大的分片更快。如果有多个并发查询，则有很多小碎片也会降低查询吞吐量。

提示：从查询性能角度确定最大分片大小的最佳方法是使用逼真的数据和查询进行基准测试（真实数据而非模拟数据）。 始终使用查询和索引负载进行基准测试，代表节点在生产中需要处理的内容，因为单个查询的优化可能会产生误导性的结果。

总结: 一定的QPS下，多分片能提高并发，但是QPS太高太低都不行，此时可能用小数量的大分片还更好，这个QPS是多少只能通过基准测试得到

## 集群状态

对于每个Elasticsearch索引，其映射和状态的信息都存储在集群状态。 这些集群状态信息保存在内存中以便快速访问。 因此，如果在集群中拥有大量索引，可能导致大的集群状态（特别是如果映射较大）。 所有更新集群状态操作为了在集群中保证一致性，需要通过单个线程完成，因此更新速度将变慢。

## 脑裂

分布式架构中最常见的问题，es的处理方式是配置最小数量的主节点设置

discovery.zen.minimum_master_nodes: (有master资格节点数/2) + 1

这个参数控制的是，选举主节点时需要看到最少多少个具有master资格的活节点，才能进行选举。官方的推荐值是(N/2)+1，其中N是具有master资格的节点的数量

要注意因为es选举算法的特殊性，是不需要奇数节点的，但是为防止脑裂还是要注意这个配置

## 别名

可以对索引取别名，好处在于，索引abc-123，索引qwe-123这样的多个索引，查询的时候可以索引模糊匹配(abc-*)，但是通常不能完全满足条件。这时候可以对索引创建别名，
这样多个索引都是一个别名，方便多索引查询

分片也可以用别名

## 规划经验

ES JVM heap 最大可以设置32G
30G heap 大概能处理的数据量 10 T

A. 用于构建业务搜索功能模块，且多是垂直领域的搜索。数据量级几千万到数十亿级别。一般2-4台机器的规模
B. 用于大规模数据的实时OLAP（联机处理分析），经典的如ELK Stack，数据规模可能达到千亿或更多。几十到上百节点的规模

## 索引及存储结构

1. 不变性

写到磁盘的倒排索引是不变的。索引的不变性也有缺点。如果你想让新修改过的文档可以被搜索到，你必须重新构建整个索引。这在一个index可以容纳的数据量和一个索引可以更新的频率上都是一个限制

2. segment

如何在不丢失不变形的好处下让倒序索引可以更改？
解决办法：使用不只一个的索引。新添额外的索引来反映新的更改来替代重写所有倒序索引的方案

一个segment是一个完整的倒序索引的子集，所以现在index在Lucene中的含义就是一个segments的集合，每个segment都包含一些提交点(commit point)
对文档的更新，新建，删除等都不会去操作原文档，会创建新的，这写操作在新的segment中被记录下来，然后提交新的segment

不过只用segment的，索引的速度会很慢，所以segment会先被写入内存。

使用内存的话就会存在数据丢失的问题，es是可靠的，它使用translog日志来恢复未被提交的segment数据

translog日志也可以用来提供实时的CRUD。当你试图通过文档ID来读取、更新、删除一个文档时，它会首先检查translog日志看看有没有最新的更新，然后再从响应的segment中获得文档。这意味着它每次都会对最新版本的文档做操作，并且是实时的

通过每隔一秒的自动刷新机制会创建一个新的segment，用不了多久就会有很多的segment。segment会消耗系统的文件句柄，内存，CPU时钟。最重要的是，每一次请求都会依次检查所有的segment。segment越多，检索就会越慢

ES通过在后台merge这些segment的方式解决这个问题。小的segment merge到大的，大的merge到更大的

3. 存储索引的流程

数据生成索引存入内存Buffer，同时存入TranSlog
内存中的数据每隔一秒以segment的形式写入系统文件缓存
每隔一段时间，文件缓存中的数据存入硬盘，同时清除对应的translog

索引过程：

1) 有一系列被索引文件

2) 被索引文件经过语法分析和语言处理形成一系列词(Term) 。

3) 经过索引创建形成词典和反向索引表。

4) 通过索引存储将索引写入硬盘。

搜索过程：

a) 用户输入查询语句。

b) 对查询语句经过语法分析和语言分析得到一系列词(Term) 。

c) 通过语法分析得到一个查询树。

d) 通过索引存储将索引读入到内存。

e) 利用查询树搜索索引，从而得到每个词(Term) 的文档链表，对文档链表进行交，差，并得到结果文档。

f) 将搜索到的结果文档对查询的相关性进行排序。

g) 返回查询结果给用户

## 索引模板（Index templates）

通过向es创建一个模板，模板有匹配索引的机制，以后创建索引的时候，被匹配上的索引将会使用模板中的配置，比如分片如何设置等

```yaml
PUT _template/feature_template

{
  "index_patterns": "feature*",
  "order": 0,
  "settings": {
    "number_of_replicas": 0,
    "refresh_interval": "30s"
  }
}
```

## 索引生命周期，索引策略

索引生命周期管理（Index Lifecycle Management ，简称ILM)

如果是电商项目，基本就是把商品数据写入，实现全文索引，一般只会新增，而且量不大
但是日志数据就很需要生命周期，比如当前时间是6.3，日志数据只存储2天的

6.3 创建的索引，需要提供查询和写入
6.2 昨天的数据，只需要提供查询
6.1 过期的数据，要被删除

ILM 一共将索引生命周期分为四个阶段(Phase)：

```yaml
Hot    阶段
Warm   阶段
Cold   阶段
Delete 阶段
```

1. Hot

类比为人类`婴儿到青年`的阶段，此阶段需要处理大量的读写操作，数据量会不断的增加，需要高配置，建议内存和磁盘占比为 1:32  比如64GB内存搭配2TB SSD

2. Warm 

类比为人类`青年到中年`的阶段，此阶段不再写入，只提供读取。中等配置即可

3. Cold 

类比为人类`中年到老年`的阶段，就好像退休了一样，只提供查询，并且被查到的概率很低了，低配置即可

4. Delete

Delete 阶段可类比为人类寿终正寝的阶段，索引被删除

### 生命周期流程

es不要去索引都要经历4个生命周期，所谓生命周期管理就是怕配置索引何时从一个周期进入到下一个周期，比如索引存在7天后，由Hot进入Warm

### 生命周期管理

ILM 将不同的生命周期管理策略称为 `Policy`，而所谓的 Policy 是由不同阶段(`Phase`)的不同动作(`Action`)组成的

Hot Phase
    Rollover 滚动索引操作，可用在索引大小或者文档数达到某设定值时，创建新的索引用于数据读写，从而控制单个索引的大小。这里要注意的一点是，如果启用了 Rollover，那么所有阶段的时间不再以索引创建时间为准，而是以该索引 Rollover 的时间为准

Warm Phase
    Allocate 设定副本数、修改分片分配规则(如将分片迁移到中等配置的节点上)等
    Read-Onlly 设定当前索引为只读状态
    Force Merge 合并 segment 操作
    Shrink 缩小 shard 分片数

Cold Phase
    Allocate 同上

Delete Phase
    Delete 删除

API操作示例

```yaml
PUT /_ilm/policy/test_ilm2
{
    "policy": {
        "phases": {
            "hot": {
                "actions": {
                    "rollover": {
                        "max_age": "30d",
                        "max_size": "50gb"
                    }
                }
            },
            "warm": {
                "min_age": "3d",
                "actions": {
                    "allocate": {
                        "require": {
                            "box_type": "warm"
                        },
                        "number_of_replicas": 0
                    },
                    "forcemerge": {
                        "max_num_segments": 1
                    },
                    "shrink": {
                        "number_of_shards": 1
                    }
                }
            },
            "cold": {
                "min_age": "7d",
                "actions": {
                    "allocate": {
                        "require": {
                            "box_type": "cold"
                        }
                    }
                }
            },
            "delete": {
                "min_age": "30d",
                "actions": {
                    "delete": {}
                }
            }
        }
    }
}
```

比较难理解的应该是rollover阶段的流程，重点理解RollOver

### RollOver

在日志处理中，如果我们只用每日一索引的策略，那么隐藏了下面的问题：

1. 为了达到较高的写入速度，活跃索引分片需要分布在尽可能多的节点上

即针对当天的索引，为了尽可能提高集群的写入能力，最好就是把这个索引拆成多个分片，让每个节点都存储一个分片，这样请求数据在路由的时候就能把数据都分散到各个节点去写，每个节点都发挥了自己的计算能力。如果是单分片，那么所有的数据压力都在这台上，并不能很好的利用集群的计算能力(如果你数据量都达不到30GB的经验分片阈值，那就不用分片了，也看具体情况)

2. 为了提高搜索速度和降低资源消耗，分片数量需要尽可能地少，但是也不能有过大的单个分片进而不便操作

3. 一天一个索引确实易于清理陈旧数据，但是一天到底需要多少个分片呢？

4. 每天的写入压力是一成不变的吗？还是一天分片过多，而下一天分片不够用呢？


### 滚动模式工作流程：

有一个用于写入的索引别名，其指向活跃索引
另外一个用于读取（搜索）的索引别名，指向不活跃索引
活跃索引具有和热节点数量一样多的分片，可以充分发挥昂贵硬件的索引写入能力
当活跃索引太满或者太老的时候，它就会滚动：新建一个索引并且索引别名自动从老索引切换到新索引
移动老索引到冷节点上并且缩小为一个分片，之后可以强制合并和压缩

### 生命周期使用总结:

复杂的是1，2，3阶段

其中1阶段要用，那么必须创建模板，创建策略，然后策略关联模板，一个策略可以有多个模板，这个时候优先级会发挥作用

1阶段的重点就是要设置别名

1. 索引必须要有一个数字后缀，不然生命周期会报错
2. 必须要创建一个别名，然后这个别名要设置能写数据
3. 模板在关联策略的时候，要指定RollOver发生的时候，使用的别名，所以这个别名要存在

生命周期1由于使用了别名，所以模板应该是一类索引数据，或者是针对单个索引来做的，别名也是针对索引来做的，不能N个日志用一个模板，用一个别名，es就是用别名来指向需要滚动的索引，一个别名指向多个乱套了

简单的ELK不太好用生命周期，或者说是需要找到适合的处理策略，因为可能一个日志就要用一个模板，还要建立对应的别名，所以需要程序来控制API

### kabina调试技巧:

模板和策略关联的时候，策略只作用于被模板匹配上的索引，所以改了策略和模板都是不生效的

可以去把索引删了，这样就又走了一遍流程，配置生效

生命周期还可以取消和索引关联

调试的最好的做法是修改策略，然后页面操作，重试生命周期，这个API也支持

## 索引的映射

映射就是对字段元信息描述

下面的命令获取索引的映射

GET eos-apollo-client-blue-2020.05.30/_mapping 

mappings" : {"properties": {}}  properties对象的每个成员就是一个字段

比如像下面这样

```json
"@version" : {
    "type" : "text",
    "fields" : {
        "keyword" : {
            "type" : "keyword",
            "ignore_above" : 256
        }
    }
},
```

可以再嵌套一个properties，beat就是字段的前缀，下面的hostname，ip最终的字段名字为 beat.hostname

```json
"beat" : {
    "properties" : {
        "hostname" : {
            "type" : "text",
            ......
        "ip" ......
```

## 索引策略生效时间

通过命令修改，默认是10分钟

```json
PUT _cluster/settings
{
  "transient": {
    "indices.lifecycle.poll_interval": "3s" 
  }
}
```

## 索引，模板，策略的关系

模板和策略都是可以独立创建的

模板可以关联策略

可以对某个索引指定策略

在使用的时候要注意，最好就是通过模板关联策略，这样符合条件的索引都使用模板，策略也会生效

没有通过模板关联，新索引就不能继承策略

## 时间配置参考

在配置很多参数的时候，需要指定对应的时间，比如年月，分钟，秒。请参考官方地址

https://www.elastic.co/guide/en/elasticsearch/reference/7.5/common-options.html#time-units

## 配置策略

调用下面的请求创建一个测试策略，kibana的参数时间选择太大，可以手动创建

PUT _ilm/policy/my_test_policy
{
  "policy" : {
      "phases" : {
        "hot" : {
          "actions" : {
            "rollover" : {
              "max_size" : "50gb",
              "max_age" : "3m"
            },
            "set_priority" : {
              "priority" : 100
            }
          }
        },
        "delete" : {
          "min_age" : "3m",
          "actions" : {
            "delete" : { }
          }
        }
      }
    }
}

## 刷新到文件系统的时间间隔

refresh_interval

这个参数说明白一点就是当我们新的数据被写入到es后，你不会立刻查询到数据，因为数据还没被索引，这个参数就控制了数据被写入后多久能被查询到

对于实时性不高的系统，不要刷新太快

## 预加载 fielddata

doc_values和fielddata就是用来给文档建立正排索引的。他俩一个很显著的区别是，前者的工作地盘主要在磁盘，而后者的工作地盘在内存

indices.fielddata.cache.size 节点用于 fielddata 的最大内存，如果 fielddata 达到该阈值，就会把旧数据交换出去。该参数可以设置百分比或者绝对值。默认设置是不限制
indices.breaker.fielddata.limit 默认值是JVM堆内存的60%,注意为了让设置正常生效，一定要确保 indices.breaker.fielddata.limit 的值大于 indices.fielddata.cache.size 的值。否则的话，fielddata 大小一到 limit 阈值就报错，就永远道不了 size 阈值，无法触发对旧数据的交换任务了


GET /_stats/fielddata?fields=* //各个分片、索引的fielddata在内存中的占用情况
GET /_nodes/stats/indices/fielddata?fields=* //每个node的fielddata在内存中的占用情况
GET /_nodes/stats/indices/fielddata?level=indices&fields=* //每个node中的每个索引的fielddata在内存中的占用情况

## index_buffer

存储新索引的数据，在内存中存储(默认是堆的10%)，填满后会写到磁盘的一个分段中。为了提高性能会先写到filesysten cache后，再刷到硬盘上

官方参考: https://www.elastic.co/guide/en/elasticsearch/reference/current/indexing-buffer.html

## refresh 和 flush

1. refresh
当我们向ES发送请求的时候，我们发现es貌似可以在我们发请求的同时进行搜索

而这个实时建索引并可以被搜索的过程实际上是一次es 索引提交（commit）的过程，如果这个提交的过程直接将数据写入磁盘（fsync）必然会影响性能，所以es中设计了一种机制，即：先将index-buffer中文档（document）解析完成的segment写到filesystem cache之中，这样避免了比较损耗性能io操作，又可以使document可以被搜索

以上从`index-buffer`中取数据到`filesystem cache`中的过程叫做`refresh`

2. flush
如果数据在filesystem cache之中是很有可能在意外的故障中丢失
这个时候就需要一种机制，可以将对es的操作记录下来，来确保当出现故障的时候，保留在filesystem的数据不会丢失，并在重启的时候可以从这个记录中将数据恢复过来elasticsearch提供了translog来记录这些操作

当向elasticsearch发送创建document索引请求的时候，document数据会先进入到index buffer之后，与此同时会将操作记录在translog之中，当发生refresh时（数据从index buffer中进入filesystem cache的过程）translog中的操作记录并不会被清除，而是当数据从filesystem cache中被写入磁盘之后才会将translog中清空

而从`filesystem cache`写入磁盘的过程就是`flush`

## 调优节点丢失问题

如果GC时间过长，swt导致该节点不工作，服务发现机制时间间隔太小，就会导致节点丢失，触发数据重平衡
可以调整服务发现相关配置，或者调整触发数据重平衡时间

discovery.zen.ping.timeout: 200s
discovery.zen.fd.ping_timeout: 200s
discovery.zen.fd.ping.interval: 30s
discovery.zen.fd.ping.retries: 6

## 断路器

indices.breaker.fielddata.limit
fielddata 断路器默认设置堆的 60% 作为 fielddata 大小的上限。

indices.breaker.request.limit
request 断路器估算需要完成其他请求部分的结构大小，例如创建一个聚合桶，默认限制是堆内存的 40%。

indices.breaker.total.limit
total 揉合 request 和 fielddata 断路器保证两者组合起来不会使用超过堆内存的 70%。

indices.fielddata.cache.size
缓存回收大小，无默认值， 有了这个设置，最久未使用（LRU）的 fielddata 会被回收为新数据腾出空间

前三项可以动态设置,最后一项要在配置文件中修改

```json
PUT /_cluster/settings
{
  "persistent": {
    "indices.breaker.fielddata.limit": "60%"
  }
} 


PUT /_cluster/settings
{
  "persistent": {
    "indices.breaker.request.limit": "40%"
  }
} 


PUT /_cluster/settings
{
  "persistent": {
    "indices.breaker.total.limit": "70%"
  }
} 
```

```
#------------有了这个设置，最久未使用（LRU）的 fielddata 会被回收为新数据腾出空间   
indices.fielddata.cache.size:  40%
```

建议

最好为断路器设置一个相对保守点的值。 记住 fielddata 需要与 request 断路器共享堆内存、索引缓冲内存和过滤器缓存。Lucene 的数据被用来构造索引，以及各种其他临时的数据结构。 正因如此，它默认值非常保守，只有 60% 。过于乐观的设置可能会引起潜在的堆栈溢出（OOM）异常，这会使整个节点宕掉。
另一方面，过度保守的值只会返回查询异常，应用程序可以对异常做相应处理。异常比服务器崩溃要好。

```
当前fieldData缓存区大小     <   indices.fielddata.cache.size 
当前fieldData缓存区大小   +  下一个查询加载进来的fieldData   <   indices.breaker.fielddata.limit 
indices.breaker.request.limit  +  indices.breaker.fielddata.limit  <  indices.breaker.total.limit
```

官方参考

https://www.elastic.co/guide/cn/elasticsearch/guide/current/_limiting_memory_usage.html
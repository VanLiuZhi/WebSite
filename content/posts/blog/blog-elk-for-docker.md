---
weight: 1000
title: "ELK实践与运用总结"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "ELK实践与运用总结"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Technology, Docker, Note]
categories: [Technology技术]

lightgallery: true

toc:
  auto: false
---

日志处理作为系统基础设施的一环，有着举足轻重的作用，一般常用ELK作为日志处理平台。ELK是ElasticSearch、Logstash、Kibana的简称，这三者是核心套件，但是随着功能的迭代和组件的更替，也出现了ELKF，其中的F指的是filebeat，也可以在ELK中加入Kafka组件，以满足复杂场景的需求，所以我们通常说的ELK泛指与之相关的各种组件

## 概述

为什么要做日志采集？

当我们的系统不是很复杂的时候，假设是单机架构的系统，以一个spring boot项目为例，我们可以通过logback做日志切割，把整个服务运行过程中的日志记录到文件中，方便后续排查问题。但是这样也有很多问题，

1. 比如日志在服务器上，而开发人员才需要看日志，一般都是运维做服务器管理的，所以有些小伙伴会开发日志下载功能，方便开发人员查看。不过日志都是实时产生的，通过下载文件查看日志的做法并不具备实时性

2. 当上到分布式系统的时候，一个业务的完成流程可能由多个服务参与，日志文件也是散落在各个服务器上的，需要把日志统一到一起才方便排查问题

ELK就提供了这样的解决方案，整体的思路是通过Logstash把日志采集后，统一发到ElasticSearch，ElasticSearch是一个高效的搜索引擎，然后我们通过官方提供的Kibana UI组件，去获取ElasticSearch中的数据

下面我们来搭建一套ELK环境，全部采用Docker部署，之后采集k8s容器的日志到ES中，通过logstash处理日志，在kibana页面上查看日志

## 资源规划

ES集群 > 3台
logstash 1台（可以和kibana共用一台）

由于通过Docker部署的，jvm参数会在容器启动的时候固定(虽然可以通过非常规手段来调整，但是一般不建议这么做)，所以我们要提前规划好资源

建议每日日志在1亿左右规模的系统，由于数据量不大，用3个节点即可，每台配置4C 8G，硬盘100G

有一个非常重要的配置就是es jvm最大堆不要超过内存的一半（官方建议即使物理机内存达到128GB，也不要设置为64GB，最大设置到32GB）

建议就是ES独立部署，该机器没有其它服务抢占资源。logstash也独立部署，如果服务器资源足够，可以和kibana部署到一起

提前安装好Docker服务

## ElasticSearch 部署

通过官方镜像来部署，ElasticSearch可以很容易扩容，通过集群的能力提高系统对数据的处理能力，我们部署3台节点

各个版本的镜像地址说明，参考 ELK官方 docker 镜像地址: https://www.docker.elastic.co/

### 服务器配置

docker pull elasticsearch:7.5.0 拉取镜像

```s
# 创建目录
mkdir -p /data/es/data
mkdir -p /data/es/config
# 给目录授权
chmod -R 777 /data/es
# 修改配置，防止启动容器时，报错m.max_map_count[65530] likely too low。
vim /etc/sysctl.conf
# 添加
vm.max_map_count=262144 
# 启用配置：
sysctl -p
```

## yaml配置文件和服务启动

每个节点都创建配置文件: cd /data/es/config  vim es.yml

对应每个节点修改 `节点名称` node.name，`网关地址` network.publish_host，其它配置各个节点相同，安全验证的配置一开始要先注释掉

执行启动命令，根据实际情况调整堆大小和文件映射

docker run -e ES_JAVA_OPTS="-Xms8g -Xmx8g" -d -p 9200:9200 -p 9300:9300 -v /data/es/config/es.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /data/es/data:/usr/share/elasticsearch/data --name es elasticsearch:7.5.0 --restart=always

集群正常启动后，可以查看统计信息:

`curl -XGET 'http://10.90.x.x:9200/_cluster/stats?pretty'`

```yaml
#集群名称
cluster.name: es-cluster
#节点名称
node.name: node-a
#是不是有资格竞选主节点
node.master: true
#是否存储数据
node.data: true
#最大集群节点数
node.max_local_storage_nodes: 10
#网关地址
network.host: 0.0.0.0
network.publish_host: 10.90.x.x
#端口
http.port: 9200
#内部节点之间沟通端口
transport.tcp.port: 9300
http.cors.enabled: true
http.cors.allow-origin: "*"
#es7.x 之后新增的配置，写入候选主节点的设备地址，在开启服务后可以被选为主节点
discovery.seed_hosts: ["10.90.x.x:9300","10.90.x.x:9300","10.90.x.x:9300"]
#es7.x 之后新增的配置，初始化一个新的集群时需要此配置来选举master，这样可以在集群启动后自己决定谁是master
cluster.initial_master_nodes: ["node-a"]
#数据存储路径
path.data: /usr/share/elasticsearch/data
#日志存储路径
path.logs: /usr/share/elasticsearch/data/logs

# 优化配置

# request cache, 默认无限制
indices.fielddata.cache.size: 30%
# 40%
# indices.breaker.fielddata.limit: 40%

# query cache，默认10%，限制为10% 属于查询缓存
# indices.queries.cache.size: 10%

# common space，默认70%
# indices.breaker.total.limit: 70%

# request agg data，默认60%
# indices.breaker.request.limit: 1%

# 写线程数设置，如果报错相关异常，可以调整这个值，默认注释
thread_pool.write.queue_size: 1000

# 启用安全验证，第一次安装的时候要注释掉
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.license.self_generated.type: basic
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: /usr/share/elasticsearch/data/elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: /usr/share/elasticsearch/data/elastic-certificates.p12

# 启用监控数据配置，这样在kibana中才能查看到集群监控数据，默认注释
xpack.monitoring.collection.enabled: true
## 监控数据存储时长，标准版配置这个参数没有效果
xpack.monitoring.history.duration: 1d
```

其它配置

```yaml
# 每个节点上允许存在的分片数量，假设现在需要在节点上存在3个分片，而设置2个就会报错
index.routing.allocation.total_shards_per_node: 2

# 分片和副本配置，下面是默认值（一般通过模板配置来做，比如匹配上的索引应用此模板的设置，可以在模板中设置索引的分片和副本数）
index.number_of_shards: 5
index.number_of_replicas: 1
```

## 配置安全认证

进入容器，执行命令，需要输入两次密码，这里两次都输入一样的

1. bin/elasticsearch-certutil ca

会让输入一个文件名 我们输入 elastic-stack-ca.p12
之后会让输入密码，输入完成执行完毕

2. bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12

拿刚才创建的文件执行命令

类似刚才上面的步骤，先输入密码验证(elastic-stack-ca.p12)， 输入一个文件名  elastic-certificates.p12
再输入一个密码，完成

此时得到两个文件

```
elastic-certificates.p12
elastic-stack-ca.p12
```

3. 再执行两个命令，需要输入2中输入的密码

bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password

密码会被保存到config目录下的elasticsearch.keystore文件中，这个文件如果被删除了，在执行上面的两个命令的时候会提示你创建

4. 把elastic-certificates.p12文件，elasticsearch.keystore拷贝出

elasticsearch.keystore文件复制到各个容器中，elastic-certificates.p12映射到主机目录(各个容器也要复制过去)，待会配置要用到

scp -r elastic-certificates.p12 root@10.90.x.x:/data/es/data

docker cp /data/es/data/elasticsearch.keystore es-cluster:/usr/share/elasticsearch/config/
docker cp /data/es/data/elasticsearch.keystore es:/usr/share/elasticsearch/config/

5. 修改配置文件，elastic-certificates.p12 赋权777

xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.license.self_generated.type: basic
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: /usr/share/elasticsearch/data/elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: /usr/share/elasticsearch/data/elastic-certificates.p12

此时，elastic-certificates.p12每个es都有了，并且赋权777，在配置文件中引用
elasticsearch.keystore也被拷贝到各个es，但是没有映射出来

下一步重启各个es

6. 初始化密码

es重启完成后，再次访问需要密码了

进入主节点的容器内，去初始化密码

bin/elasticsearch-setup-passwords interactive  需要初始化6个账户的密码，以后就用这些账户登录，当然为了方便你可以把密码设置成一样的

然后重启各个es(这里只需要去主节点初始化密码就行了)

7. kibana 配置密码

elasticsearch.username: "kibana"
elasticsearch.password: "之前为kibana帐户创建的密码"

修改配置文件，重启

这里要用 elastic 账户登录，用kibana账户会报403

8. 小结

证书可以复用的，这样就不必每次安装新的集群就去创建证书
总共是3个文件

```
elastic-certificates.p12
elastic-stack-ca.p12
elasticsearch.keystore
```

其中elastic-stack-ca.p12只在创建证书的容器中才存在，密码总共需要输入2次，原则上2次应该用2个密码，不过这个密码只在创建证书的时候需要，如果嫌麻烦就统一用一个也行

## ES 插件离线安装

插件离线安装，主要是要用文件资源路径，然后高版本不一定是用plugin命令，在bin命令下可以自己尝试命令

中文分词插件地址：`https://github.com/medcl/elasticsearch-analysis-ik/`

在容器中执行命令 `./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.5.0/elasticsearch-analysis-ik-7.5.0.zip`

但是可能需要离线安装，把文件cp到容器中，通过命令指定文件，注意路径`file:///`

`bin/plugin install file:///usr/local/elasticsearch/elasticsearch-head-master.zip`

安装成功后，重启容器

## 如何查看ES安装的插件

关于这点有疑问，可能高版本自带的插件就不展示出来了，只能展示自己安装的(猜测)，ES是有自带一部分插件的

`http://esip地址/_cat/plugins`

`http://xxx:9200/_cat/plugins`

三台都安装了中文分词器后，页面反馈如下

```s
node-1 analysis-ik 7.5.0
node-2 analysis-ik 7.5.0
node-3 analysis-ik 7.5.0
```

## kibana 部署

```s
#逐级创建目录
mkdir /data/es/kibana/config
mkdir /data/es/kibana/plugins
#给目录授权
cd /data/es
chmod 777 kibana
cd /data/es/kibana/config
#制作配置文件
vim kibana.yml 
```

```yaml
server.name: kibana
server.host: "0.0.0.0" 
elasticsearch.hosts: ["http://10.90.x.x:9200","http://10.90.x.x:9200","http://10.90.x.x:9200"]
xpack.monitoring.ui.container.elasticsearch.enabled: true
i18n.locale: "zh-CN"

# 启用安全认证后，需要添加如下配置
elasticsearch.username: "kibana"
elasticsearch.password: "之前为kibana帐户创建的密码"
```

docker pull kibana:7.5.0

启动服务：

docker run -d --name kibana  -p 5601:5601 -v /data/es/kibana/config/kibana.yml:/usr/share/kibana/config/kibana.yml  -v /data/es/kibana/plugins:/usr/share/kibana/plugins:rw  --name kibana kibana:7.5.0

### 命令行工具使用

```yaml
# 节点如果有未分片的，可以通过这个命令找到，一般是网络问题导致分片失败并一直无法恢复。关闭分片再开启可以解决问题
GET /_cluster/allocation/explain

# 解决索引某个shard无法恢复的问题
POST /_cluster/reroute?retry_failed=true
```

### 创建索引生命周期策略

直接通过页面填写表单即可，一般会创建删除策略，在模板配置的时候和策略关联上

### 索引模板使用

创建索引模板k8s-template，设置索引匹配模式

索引设置

```json
{
  "index": {
    "lifecycle": {
      "name": "k8s-logs-delete-policy" // 关联生命周期策略
    },
    "refresh_interval": "30s",
    "number_of_shards": "1",
    "number_of_replicas": "1"
  }
}
```

映射设置

```json
{
  "dynamic_templates": [
    {
      "message_field": {
        "path_match": "message",
        "mapping": {
          "norms": false,
          "type": "text"
        },
        "match_mapping_type": "string"
      }
    },
    {
      "string_fields": {
        "mapping": {
          "norms": false,
          "type": "text",
          "fields": {
            "keyword": {
              "ignore_above": 256,
              "type": "keyword"
            }
          }
        },
        "match_mapping_type": "string",
        "match": "*"
      }
    }
  ],
  "properties": {
    "@timestamp": {
      "type": "date"
    },
    "geoip": {
      "dynamic": true,
      "properties": {
        "ip": {
          "type": "ip"
        },
        "latitude": {
          "type": "half_float"
        },
        "location": {
          "type": "geo_point"
        },
        "longitude": {
          "type": "half_float"
        }
      }
    },
    "@version": {
      "type": "keyword"
    }
  }
}
```

如果需要使用生命周期，滚动策略要创建别名

## logstash 部署

docker pull logstash:7.5.0

启动命令：

docker run -d -p 5044:5044 -p 9600:9600 -it -v /data/logstash/config/:/usr/share/logstash/config/  --name logstash logstash:7.5.0

创建目录：mkdir -p /data/logstash/config

创建文件，共创建4个文件，分别是logstash.yml，log4j2.properties，pipelines.yml，es.conf，之后启动服务

logstash 最核心的能力还是对采集到的日志做过滤。大致是这样一个效果，比如一条日志经过filter，被格式化成json数据，通过grok过滤器处理数据，配置如下

es.conf

```conf
input {
   beats {
       port => 5044
        codec => plain{
           charset=>"UTF-8"
       }
  }
}
filter {
  grok {
     match => [
       "message", "\s{0,5}%{TIMESTAMP_ISO8601:logTime}.{0,10}\|.*\|%{LOGLEVEL:logLevel}\|\s{0,3}%{GREEDYDATA:logMessage}",
       "message", "\s{0,3}%{TIMESTAMP_ISO8601:logTime}.\s{0,10}%{LOGLEVEL:logLevel}\s{0,10}%{GREEDYDATA:logMessage}"
     ]
     #用上面提取的message覆盖原message字段
     #overwrite => ["message"]
 }
}
output {
   elasticsearch {
      action => "index"
      hosts  => ["10.90.x.x:9200","10.90.x.x:9200","10.90.x.x:9200"]
      index  => "k8s-%{index}-%{+YYYY.MM.dd}"   #使用log-pilot给的索引名+年月日
      codec => plain {
            format => "%{message}"
            charset => "UTF-8"
        }
      user => "elastic"
      password => "填入设置的密码"
    }
}
```

log4j2.properties

```properties
logger.elasticsearchoutput.name = logstash.outputs.elasticsearch
logger.elasticsearchoutput.level = debug
```

logstash.yml

```yaml
http.host: "0.0.0.0"
http.port: 9600
xpack.monitoring.elasticsearch.hosts: ["http://10.90.x.x:9200","http://10.90.x.x:9200","http://10.90.x.x:9200"]
xpack.monitoring.elasticsearch.username: "elastic"
xpack.monitoring.elasticsearch.password: "填入设置的密码"
```

pipelines.yml

```yaml
- pipeline.id: my-logstash
  path.config: "/usr/share/logstash/config/*.conf"
  pipeline.workers: 3
```

部署完成后，访问地址验证：

10.90.xx.xx:9600

### 监控配置

```yaml
#是否集群
xpack.monitoring.enabled: true
#初始用户，必须是这个
xpack.monitoring.elasticsearch.username: "logstash_system"
#密码，es中为logstash_system设置的密码
xpack.monitoring.elasticsearch.password: "填入设置的密码"
#es的集群x
xpack.monitoring.elasticsearch.hosts: ["http://10.90.x.x:9200","http://10.90.x.x:9200","http://10.90.x.x:9200"]
```

### 配置和概念

会自动生成一个字段@timestamp，默认该字段存储的是Logstash收到消息/事件(event)的时间，也就是说这个字段不是日志产生的真实时间

@timestamp 这个字段是logstash解析出来的，当成字段存储在es中，这个也可以自定义匹配规则

文件布局规划: https://segmentfault.com/a/1190000015242897 在容器映射的时候可能会用到

每个 logstash 过滤插件，都会有四个方法叫 add_tag, remove_tag, add_field 和 remove_field。它们在插件过滤匹配成功时生效

## 前端可视化工具cerebro

docker run -d -p 9000:9000 --name es-cerebro lmenezes/cerebro:latest
docker run -d -p 9000:9000 -v /data/cerebro/application.conf:/opt/cerebro/conf/application.conf --name es-cerebro lmenezes/cerebro:latest

## log-pilot 采集k8s日志

启动的容器服务，通过配置tag，会被log-pilot解析到，去获取容器的日志

### 特性

Log-Pilot具有自动发现机制: 容器配置好tag后就会被采集组件发现
CheckPoint句柄保持的机制: 对于日志文件的句柄做跟踪
自动日志数据打标: 通过对容器加tag，这个tag会被记录到日志中，取出日志的时候就可以通过这个tag来区分数据
有效应对动态配置: 容器扩缩容的时候能够自动处理
日志重复和丢失以及日志源标记等问题: 通过句柄保持实现

### lable

aliyun.logs.$name = $path
变量 name 是日志名称，只能包含 0~9、a~z、A~Z 和连字符（-）
变量 path 是要收集的日志路径，必须具体到文件，不能只写目录。文件名部分可以使用通配符，例如，/var/log/he.log 和 /var/log/*.log 都是正确的值，但 /var/log 不行，不能只写到目录。stdout 是一个特殊值，表示标准输出

aliyun.logs.$name.format：日志格式，目前支持以下格式
none：无格式纯文本
json：json 格式，每行一个完整的 json 字符串
csv：csv 格式

aliyun.logs.$name.tags：上报日志时，额外增加的字段，格式为 k1=v1,k2=v2，每个 key-value 之间使用逗号分隔，例如 aliyun.logs.access.tags="name=hello,stage=test"，上报到存储的日志里就会出现 name 字段和 stage 字段
如果使用 ElasticSearch 作为日志存储，target 这个 tag 具有特殊含义，表示 ElasticSearch 里对应的 index

### 使用

1. 配置一个log-pilot demonSet发布到k8s中，这样每个Node就有采集组件了
2. 为要采集的Docker容器添加lable，使用的关键就在于这个tag要怎么添加

PILOT_LOG_PREFIX: "aliyun,custom" 通过修改这个环境变量能改变lable的前缀，默认是aliyun(有些版本不适用了)

docker pull log-pilot:0.9.6-filebeat

部署yaml

```yaml
---
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: log-pilot
  namespace: kube-system
  labels:
    k8s-app: log-pilot
    kubernetes.io/cluster-service: "true"
spec:
  template:
    metadata:
      labels:
        k8s-app: log-es
        kubernetes.io/cluster-service: "true"
        version: v1.22
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      serviceAccountName: dashboard-admin
      containers:
      - name: log-pilot
        # 版本请参考https://github.com/AliyunContainerService/log-pilot/releases
        image: log-pilot:latest
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        env:
          - name: "NODE_NAME"
            valueFrom:
              fieldRef:
                fieldPath: spec.nodeName
          - name: "LOGGING_OUTPUT"
            value: "logstash"
          - name: "LOGSTASH_HOST"
            value: "10.90.x.x"
          - name: "LOGSTASH_PORT"
            value: "5044"
          - name: "LOGSTASH_LOADBALANCE"
            value: "true"
          #- name: "FILEBEAT_OUTPUT"
          #  value: "elasticsearch"
          #- name: "ELASTICSEARCH_HOST"
          #  value: "elasticsearch"
          #- name: "ELASTICSEARCH_PORT"
          #  value: "9200"
          #- name: "ELASTICSEARCH_USER"
          #  value: "elastic"
          #- name: "ELASTICSEARCH_PASSWORD"
          #  value: "changeme"
        volumeMounts:
        - name: sock
          mountPath: /var/run/docker.sock
        - name: root
          mountPath: /host
          readOnly: true
        - name: varlib
          mountPath: /var/lib/filebeat
        - name: varlog
          mountPath: /var/log/filebeat
        securityContext:
          capabilities:
            add:
            - SYS_ADMIN
      terminationGracePeriodSeconds: 30
      volumes:
      - name: sock
        hostPath:
          path: /var/run/docker.sock
      - name: root
        hostPath:
          path: /
      - name: varlib
        hostPath:
          path: /var/lib/filebeat
          type: DirectoryOrCreate
      - name: varlog
        hostPath:
          path: /var/log/filebeat
          type: DirectoryOrCreate
```

deployment 注入环境变量配置，假设应用名称叫做 monitor-center

```yaml
- name: aliyun_logs_monitor-center-stdout
  # 采集控制台
  value: "stdout"
- name: aliyun_logs_monitor-center-tomcat
  # 采集指定目录
  value: "/usr/local/tomcat/logs/*.log"
- name: aliyun_logs_monitor-center-netcore
  # 采集指定目录
  value: "/app/logs/*.log"
- name: aliyun_logs_monitor-center-java
  # 采集指定目录
  value: "/logs/*.log"
- name: aliyun_logs_monitor-center-stdout_tags
  # 为 aliyun_logs_monitor-center-stdout 采集 控制台 配置打上标签，下面类似
  value: "app=monitor-center,lang=all,sourceType=stdout"
- name: aliyun_logs_monitor-center-tomcat_tags
  value: "app=monitor-center,lang=java,sourceType=log"
- name: aliyun_logs_monitor-center-netcore_tags
  value: "app=monitor-center,lang=net,sourceType=log"
- name: aliyun_logs_monitor-center-java_tags
  value: "app=monitor-center,lang=java,sourceType=log"
```

### 排查filebeat配置是否正确

kubectl -n kube-system get pod | grep log-pilot

kubectl exec -it log-pilot-nspdv sh -n kube-system

cat /etc/filebeat/filebeat.yml

## 其它知识点

常见的日志采集组件

1. filebeat
2. Logstash
3. logpilot
4. fluentd

### ElasticSearch Curator

是一个python开发的工具，可以更方便的操作es，不用直接发HTTP请求了
不过这个工具在更高的版本中也已经过时了，es7.X 请直接学习生命周期的运用

### elastalert

es扩展插件。可以实现针对特定的匹配日志做处理，产生告警

### Elastic Beats

Beats就是轻量的意思，是获取数据的源头服务，这些可以是日志文件（Filebeat），网络数据（Packetbeat），服务器指标（Metricbeat）

所以社区出了 ELKB 的概念，而我们使用log-pilot来采集容器数据，其实已经是 ELKB 的思想了

https://www.cnblogs.com/sanduzxcvbnm/p/12076383.html

### 关于type

5.x 版本可以创建多个type
6.x 版本只能创建一个type
7.x 版本去掉了type

es是基于索引index的，不需要通过type(关系数据库中的表)来提高查询速度

### 关于节点 

主节点，数据节点，预处理节点(协调节点)。节点类型可以显示的指定，默认都充当了各种功能，主节点是选出来的
主节点要负责同步集群状态，协调节点负责转发请求，如果协调节点也参与数据处理，那么协调节点负载过高，无法转发请求，可能就会影响整体性能
在大规模节点下，比如10台以上，可以每个节点专一职责，对于预处理节点由于不需要存储数据，对CPU内存的要求也不会很高

最好使用nginx负载均衡，轮询节点处理请求，不要每次都把请求发到一个节点上

### 关于节点重平衡

突然的一台宕机没必要触发集群重平衡，可以关闭，或者设置触发延迟时间

### 关于自动创建索引

如果不允许自动创建索引，那么ELK中向ES推日志的时候，就没法在前端看到，需要手动创建索引。通过修改配置可以开关这个功能

### 默认堆

es 默认堆最大最小就是2GB，所以如果是为了测试不知道堆的时候，可能因为默认值太大导入docker启动失败

## Docker ES 修改

如果要调整ES jvm参数，步骤

1. 先关闭容器，rm容器
2. 修改配置文件，把安全相关的注释了，不然容器无法启动
3. 修改jvm再次启动容器
4. 复制keywords相关文件到容器
5. 修改配置文件取消注释
6. 重启容器

因为使用了安全校验，keywords文件在容器销毁后消失，注意插件也没了

然后存储数据是映射到本地的，不用理会，当集群失去节点后，会重新平衡索引到其它节点，当集群新增节点后，会再次平衡数据
存储到本地的数据在节点失效的时候就已经失效了，节点重新加入数据又被写入新的，总之在docker模式下，不需要太关注数据的文件

以上的做法其实是重新部署的，还有一种方式是非规操作，可以不用销毁容器，不过不太稳定，建议jvm一开始就规划好。如果非要修改，先采用非规操作，就是去修改docker container里面的文件，这个有点复杂。这样数据还是存在的，如果集群节点较多，有副本备份，那么可以直接销毁容器，这样数据也是不丢失的

## 问题汇总

记录常见问题处理方案

### search.max_buckets 过小异常

```s
trying to create too many buckets. must be less than or equal to: [100000] but was [100001]. this limit can be set by changing the [search.max_buckets] cluster level setting.
```

出现上述错误，可能是search.max_buckets设置问题，使用transient临时修改

```s
curl -H 'Content-Type: application/json' ip:port/_setting/cluster -d '{
"transient": {
    "search.max_buckets": 217483647
        }
}'
```

通过kibana，persistent是持久化配置

```s
PUT /_cluster/settings
{
  "persistent": {
    "search.max_buckets": 217483647
  }
}
```

或者直接改配置文件

官方参考 https://www.elastic.co/guide/en/elasticsearch/reference/master/search-aggregations-bucket.html

### 分片数过小

需要调整分片数大小，7.5默认分片数是2000，超过这个值集群无法创建分片，通过kibana，persistent是持久化配置，transient是暂时的

```s
PUT /_cluster/settings
{
  "persistent": {
    "cluster": {
      "max_shards_per_node":10000
    }
  }
}
```

### 写入线程过小

配置文件修改后重启，写入线程大小修改： thread_pool.write.queue_size: 1000

### 修改默认放回值条目

index.max_result_window: 1000000  默认只能返回10000，可能调用链太长需要修改这个配置(skywalking)

ELK 架构参考 https://www.one-tab.com/page/tto_XdDeQlS44BY-ziLvKg
ELK 搭建 https://www.one-tab.com/page/Fb3B3qd2Q9yR9W92dZ2pYQ


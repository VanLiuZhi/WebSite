---
weight: 3300
title: "Sw Deploy"
subtitle: "Sw Deploy"
date: 2020-09-24T16:27:55+08:00
lastmod: 2020-09-24T16:27:55+08:00
draft: true
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Sw Deploy"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Linux, Docker, Note, Cloud]
categories: [Cloud-Native]

lightgallery: true

toc:
  auto: false
---

Deploy

<!--more-->


## 架构图

![integration](http://skywalking.apache.org/assets/img/integration.2e80f9dd.jpg)

整个系统分为三部分：

- agent：采集tracing（调用链数据）和metric（指标）信息并上报
- OAP：收集tracing和metric信息通过analysis core模块将数据放入持久化容器中（ES，H2（内存数据库），mysql等等），并进行二次统计和监控告警
- webapp(UI)：前后端分离，前端负责呈现，并将查询请求封装为graphQL提交给后端，后端通过ribbon做负载均衡转发给OAP集群，再将查询结果渲染展示


## 镜像版本选择

本次提供了8.0.0和6.6.0的部署文件，主要是3个镜像文件。由于官方提供的部署方式是Helm的，所以我们自己去Docker Hub上找到官方镜像，编写yaml文件来部署

选择ui和oap相匹配的版本，注意oap有es版本，因为官方默认也是用es来做存储的，所以我们也采用es的版本

1. apache/skywalking-oap-server:6.6.0-es7 or apache/skywalking-oap-server:8.0.0-es7
2. apache/skywalking-ui:6.6.0 or apache/skywalking-ui:8.0.0

由于agent官方是没有提供镜像的，需要我们去官方发行版中找到agent的文件，自己制作镜像

下载地址 http://skywalking.apache.org/downloads/ 选择对应的版本下载

关于镜像需要了解的点就这些


## k8s 部署要点

通过deployment部署 OAP 和 UI服务，通过Side Car模式把agent挂载到微服务应用里面，对应用代码零侵入，而且不限制原来应用的语言。OAP需要提供的配置文件通过ConfigMap注入，包括

1. application.yml 应用配置
2. alarm-settings.yml 告警规则配置
3. log4j2.xml 日志格式配置，为了ELK采集

### OAP

`端口映射`，主要是grpc和rest

grpc是代理接入，数据推送和远程调用需要使用到的，比较重要

rest是使用官方GraphQL API的端口，详情查看 https://github.com/apache/skywalking-cli

```yaml
ports:
- containerPort: 11800
  name: grpc
  protocol: TCP
- containerPort: 12800
  name: rest
  protocol: TCP
```

`环境变量`

如果是部署6.X版本，需要加上这句话

```yaml
- name: SW_L0AD_CONFIG_FILE_FROM_VOLUME
  value: "true"
```

原因是在6.X的官方镜像中，启动脚本docker-entrypoint.sh中，有这么一段逻辑

```sh
if [[ -z "$SW_L0AD_CONFIG_FILE_FROM_VOLUME" ]] || [[ "$SW_L0AD_CONFIG_FILE_FROM_VOLUME" != "true" ]]; then
    generateApplicationYaml
    echo "Generated application.yml"
    echo "-------------------------"
    cat ${var_application_file}
    echo "-------------------------"
fi
```

没有环境变量的话，将使用系统生成的配置文件，我们注入的配置文件就失效了，会导致服务无法启动

### java agent

skywalking的数据采集是采用服务推送的模式，数据指标推送给OAP服务处理。有多种实现方式，基于代理的实现方式是对系统侵入比较小的，通过Side Car模式来实现代理也是微服务架构中常用的模式

制作镜像的时候，我们只需要把文件复制到镜像中就行了

```
FROM busybox:latest 

RUN mkdir -p /opt/skywalking/agent/

ADD agent/ /opt/skywalking/agent/

WORKDIR /
```

使用agent，通过initContainers把agent挂载到volume中

```yaml
initContainers:
- name: init-agent
  image: hub.eos-ts.h3c.com/skywalking-agent:latest
  command:
  - 'sh'
  - '-c'
  - 'set -ex;mkdir -p /skywalking/agent;cp -r /opt/skywalking/agent/* /skywalking/agent;'
  volumeMounts:
  - name: agent
    mountPath: /skywalking/agent
```

然后通过commod覆盖容器的启动命令

```yaml
command: ["/bin/bash", "-c", "java -javaagent:/opt/skywalking/agent/skywalking-agent.jar -Dskywalking.agent.service_name=eos-user -Dskywalking.collector.backend_service=$SKYWALKING_ADDR -jar /app.jar"]
```

这里涉及到Docker的`ENTRYPOINT`，`CMD` 和 k8s yaml 的 `command` 和 `agrs`的运用，

由于命令和参数分割比较难处理，推荐统一使用命令，因为服务也不会存在加参数的情况。



Docker的ENTRYPOINT就对应了yaml的command，通过commod来覆盖容器本身的启动命令，

所以需要注意的就是容器启动有没有什么特殊性，比如是通过脚本启动的，直接通过java -jar的方式还不行，根据实际情况做调整



参数含义：

1. javaagent Java本身的规范，指定代理路径
2. Dskywalking.agent.service_name 服务在skywalking中展示的名称
3. Dskywalking.collector.backend_service 连接 OAP 服务 grpc 的地址



agent本身的配置

主要是对agent如何采集数据的配置，这个配置在`agent/config/agent.config`下，所以在镜像构建的时候就要配置好，也可以通过环境变量注入

环境变量注入思路：agent.config可以读取环境变量的值，所以我们在dockerfile或者yaml中注入环境变量可以替换agent的参数

配置参考：https://github.com/VanLiuZhi/document-cn-translation-of-skywalking/blob/master/docs/zh/8.0.0/setup/service-agent/java-agent/README.md



## 配置文件

下面对配置文件做一个概述，我们可以参考原始镜像的目录来调整挂载路径，最好不要把整个目录都覆盖了，只挂载需要替换的配置文件是比较合理的做法。具体的配置文件可以从镜像内部获取，或者参考官方发行版的配置

### application.yml

`cluster 配置`

主要是配置OAP服务的部署模式，这里采用k8s来部署集群，通过修改副本数就可以实现服务高可用

```yaml
kubernetes:
  watchTimeoutSeconds: ${SW_CLUSTER_K8S_WATCH_TIMEOUT:60}
  namespace: ${SW_CLUSTER_K8S_NAMESPACE:skywalking-min}
  labelSelector: ${SW_CLUSTER_K8S_LABEL:app=oap}
  uidEnvName: ${SW_CLUSTER_K8S_UID:SKYWALKING_COLLECTOR_UID}
```

修改选择标签到对应的OAP deployment。另外注意使用k8s作为集群模式，需要提供k8s RBAC 访问权限，经过测试，在8.0.0版本下，没有访问权限的话无法使用k8s来部署集群

`storage 配置，数据存储位置`

注意是es的地址和账号密码，nameSpace的作用是在索引前面加前缀，方便区分集群中的其它索引。调整副本和分片数，指定数据存储时间recordDataTTL

```yaml
storage:
  elasticsearch7:
    nameSpace: "eos_sw"
    clusterNodes: ${SW_STORAGE_ES_CLUSTER_NODES:10.90.xx.xx:9200}
    protocol: ${SW_STORAGE_ES_HTTP_PROTOCOL:"http"}
    # trustStorePath: ${SW_SW_STORAGE_ES_SSL_JKS_PATH:"../es_keystore.jks"}
    # trustStorePass: ${SW_SW_STORAGE_ES_SSL_JKS_PASS:""}
    user: ${SW_ES_USER:"elastic"}
    password: ${SW_ES_PASSWORD:"123456"}
    indexShardsNumber: ${SW_STORAGE_ES_INDEX_SHARDS_NUMBER:1}
    indexReplicasNumber: ${SW_STORAGE_ES_INDEX_REPLICAS_NUMBER:1}
    # Those data TTL settings will override the same settings in core module.
    recordDataTTL: ${SW_STORAGE_ES_RECORD_DATA_TTL:3} # Unit is day
    otherMetricsDataTTL: ${SW_STORAGE_ES_OTHER_METRIC_DATA_TTL:45} # Unit is day
    monthMetricsDataTTL: ${SW_STORAGE_ES_MONTH_METRIC_DATA_TTL:18} # Unit is month
    # Batch process setting, refer to https://www.elastic.co/guide/en/elasticsearch/client/java-api/5.5/java-docs-bulk-processor.html
    bulkActions: ${SW_STORAGE_ES_BULK_ACTIONS:1000} # Execute the bulk every 1000 requests
    flushInterval: ${SW_STORAGE_ES_FLUSH_INTERVAL:10} # flush the bulk every 10 seconds whatever the number of requests
    concurrentRequests: ${SW_STORAGE_ES_CONCURRENT_REQUESTS:2} # the number of concurrent requests
    resultWindowMaxSize: ${SW_STORAGE_ES_QUERY_MAX_WINDOW_SIZE:10000}
    metadataQueryMaxSize: ${SW_STORAGE_ES_QUERY_MAX_SIZE:5000}
    segmentQueryMaxSize: ${SW_STORAGE_ES_QUERY_SEGMENT_SIZE:200}
```

关于application的配置需要注意的一点就是各个版本的配置是有一定出入的，比如8.X和6.X版本的es存储配置就不一样，所以建议从官方发行版下载文件，参考文件来修改

### alarm-settings.yml

告警配置

举例：监控服务响应时间

```yaml
# 规则名称
service_instance_resp_time_rule:
  # 指定采集度量，service_instance_resp_time 是通过OAL查询定义的一个度量
  metrics-name: service_instance_resp_time
  op: ">"
  # 阈值
  threshold: 10
  # 周期
  period: 10
  # 次数
  count: 2
  # 告警后静默时间
  silence-period: 5
  message: Response time of service instance {name} is more than 10ms in 2 minutes of last 10 minutes
```

`官方的描述`：阈值被设置为10毫秒，基本请求很容易到达阈值。周期是10分钟，次数是2次。最近2分钟内的平均响应时间超过10毫秒就告警，告警触发后，沉默5分钟后才会告警

官方的描述中，总觉得period周期这个概念没有体现，当然也可能是中文翻译的问题，上面的message部分就是官方原文描述，个人结合实际测试的理解：

period是周期，是评判的范围，单位是分钟，现在设置为10，那么count次数就是在10分钟内如果有2次指标超过阈值，就会触发告警，然后静默5分钟，
继续循环。10分钟的范围，假设前30秒就有2次达到阈值，那么就会触发告警，接着进入沉默期，然后继续判断。
如果10分钟的范围内只有1次，等待11分钟的时候又有一次，那么不告警。沉默期为0也不会马上发，最少间隔1分钟

实际测试的结果：

silence-period 的配置，这个配置决定了告警后静默时间，假如当前服务一直有问题，而且silence-period = 0 ，那么1分钟推送一次告警。如果silence-period = 1，那么就是2分钟推送一次告警(建立在当前服务一直有问题的前提下)

可以监控的指标：

1. 服务监控
2. 实例监控
3. 服务与服务之间监控
3. 实例与实例之间监控
4. 端点监控(就是接口路径)
5. 端点和端点之间监控
6. JVM和数据库访问监控

OAL示例：
```s
// 计算 Endpoint1 和 Endpoint2 的 p99 值
Endpoint_p99 = from(Endpoint.latency).filter(name in ("Endpoint1", "Endpoint2")).summary(0.99)

// 计算端点名以 `serv` 开头的端点的 p99 值
serv_Endpoint_p99 = from(Endpoint.latency).filter(name like ("serv%")).summary(0.99)

// 计算每个端点的响应平均时长
Endpoint_avg = from(Endpoint.latency).avg()

// 计算每个端点 p50, p75, p90, p95 and p99 的延迟柱状图, 每隔 50 毫秒一条柱
Endpoint_percentile = from(Endpoint.latency).percentile(10)

// 统计每个服务响应状态为 true 的百分比
Endpoint_success = from(Endpoint.*).filter(status = "true").percent()

// 统计每个服务响应码在 [200, 299] 之间的百分比
Endpoint_200 = from(Endpoint.*).filter(responseCode like "2%").percent()

// 统计每个服务响应码在 [500, 599] 之间的百分比
Endpoint_500 = from(Endpoint.*).filter(responseCode like "5%").percent()

// 统计每个服务的调用总量
EndpointCalls = from(Endpoint.*).sum()

disable(segment);
disable(endpoint_relation_server_side);
disable(top_n_database_statement);
```

在6.5.0之后的版本中，可以通过配置中心来动态修改告警规则配置

# skywalking

官方中文翻译：https://github.com/SkyAPM/document-cn-translation-of-skywalking

快速搭建：http://skywalking.apache.org/zh/blog/2020-04-19-skywalking-quick-start.html

-Dskywalking.agent.service_name=skywalking-test-local -Dskywalking.collector.backend_service=127.0.0.1:11800 -javaagent:D:\\JavaLearProject\\apache-skywalking-apm-bin-es7\\agent\\skywalking-agent.jar

## 概念

Backend的gRPC相关的API可访问0.0.0.0/11800，rest相关的API可访问0.0.0.0/12800

## 启动模式

启动模式
在不同的部署工具（如K8S）中，可能需要不同的启动模式。 我们还提供另外两种可选的启动模式。

默认模式
默认模式。如果需要，进行初始化工作，启动监听并提供服务。

运行 /bin/oapService.sh(.bat) 来启动这个模式。也可以在使用 startup.sh(.bat)来启动。

初始化模式
在此模式下，OAP服务器启动以执行初始化工作，然后退出。 您可以使用此模式初始化存储，例如ElasticSearch索引、MySQL和TIDB表，和init数据。

运行 /bin/oapServiceInit.sh(.bat) 来启动这个模式。

非初始化模式
在此模式下，OAP服务器不进行初始化。 但它等待存在弹性搜索索引、mysql和tidb表，开始倾听并提供服务。意味着此OAP服务希望别的OAP服务器进行初始化。

运行 /bin/oapServiceNoInit.sh(.bat) 来启动这个模式。

## 配置文件

application.yml 作为核心配置文件

Level 1, 模块名。模块定义项。
Level 2, 模块类型。 设置模块类型。
Level 3. 类型属性。

```yaml
storage:
  selector: mysql # the mysql storage will actually be activated, while the h2 storage takes no effect
  h2:
    driver: ${SW_STORAGE_H2_DRIVER:org.h2.jdbcx.JdbcDataSource}
    url: ${SW_STORAGE_H2_URL:jdbc:h2:mem:skywalking-oap-db}
    user: ${SW_STORAGE_H2_USER:sa}
    metadataQueryMaxSize: ${SW_STORAGE_H2_QUERY_MAX_SIZE:5000}
  mysql:
    properties:
      jdbcUrl: ${SW_JDBC_URL:"jdbc:mysql://localhost:3306/swtest"}
      dataSource.user: ${SW_DATA_SOURCE_USER:root}
      dataSource.password: ${SW_DATA_SOURCE_PASSWORD:root@1234}
      dataSource.cachePrepStmts: ${SW_DATA_SOURCE_CACHE_PREP_STMTS:true}
      dataSource.prepStmtCacheSize: ${SW_DATA_SOURCE_PREP_STMT_CACHE_SQL_SIZE:250}
      dataSource.prepStmtCacheSqlLimit: ${SW_DATA_SOURCE_PREP_STMT_CACHE_SQL_LIMIT:2048}
      dataSource.useServerPrepStmts: ${SW_DATA_SOURCE_USE_SERVER_PREP_STMTS:true}
    metadataQueryMaxSize: ${SW_STORAGE_MYSQL_QUERY_MAX_SIZE:5000}
  # other configurations
```

storage is the 模块名
selector 模块类型.
default 模块默认属性.
driver, url, ... metadataQueryMaxSize 类型属性.

同时，模块包括必修模块和可选模块，必修模块提供后端框架， 即使模块化支持可插拔，删除这些模块是没有意义的，对于可选模块，其中一些有 一个名为“none”的提供程序实现，这意味着它只提供一个没有实际逻辑的shell，典型的例子是telemetry。 将“-”设置为“selector”意味着在运行时将排除整个模块。 我们强烈建议您不要尝试更改这些模块的api，除非你非常了解SkyWalking项目及其代码。

必须的模块列表

Core。做所有数据分析和流调度的基本和主要框架。
Cluster。管理集群中的多个后端实例，这可以提供高吞吐量的处理能力。
Storage。持久化分析结果。
Query。提供查询接口给UI。
对于Cluster 和Storage 有多个实现者(提供者), 查看 Cluster management 和 Choose storage 的link list文档。

一些receiver 模块也提供了。 Receiver是一个模块，负责接受后端的传入数据请求。大多数（所有）通过一些rpc协议，如GRPC和HTTPrestful提供。 Receiver有许多不同的模块名，你可以阅读link list中的Set receivers文档。


## k8s 中java进程

java -Dapp.id=spring-demo -javaagent:/opt/skywalking/agent/skywalking-agent.jar -Dskywalking.agent.service_name=spring-demo -Dskywalking.collector.backend_service=oap:11800 -jar /app/app.jar

## 告警

实体名称
定义范围和实体名称之间的关系.

服务: 服务名称
实例: {服务名称} 的 {实例名称}
端点: {服务名称} 中的 {端点名称}
数据库: 数据库服务名
服务关系: {源服务名称} 到 {目标服务名称}
实例关系: {源服务名称} 的 {源实例名称} 到 {目标服务名称} 的 {目标实例名称}
端点关系: {源服务名称} 中的 {源端点名称} 到 {目标服务名称} 中的 {目标端点名称}


触发告警条件：由周期，次数，沉默期来共同决定

官方描述：

端点平均响应时间在最近 2 分钟内超过1秒

```yaml
service_instance_resp_time_rule:
    metrics-name: service_instance_resp_time
    op: ">"
    threshold: 1000
    period: 10
    count: 2
    silence-period: 5
    message: Response time of service instance {name} is more than 1000ms in 2 minutes of last 10 minutes
```

举例：监控服务响应时间
```yaml
metrics-name: service_instance_resp_time
op: ">"
threshold: 10
period: 10
count: 2
silence-period: 5
```

阈值被设置为10毫秒，基本请求很容易到达阈值。周期是10分钟，次数是2次。最近2分钟内的平均响应时间超过10毫秒就告警，告警触发后，沉默5分钟后才会告警，这是官方的描述

个人结合实际测试的理解：period是周期，是评判的范围，单位是分钟，现在设置为10，那么count次数就是在10分钟内如果有2次指标超过阈值，就会触发告警，然后静默5分钟。继续循环

10分钟的范围，假设前30秒就有2次达到阈值，那么就会触发告警，接着进入沉默期，然后继续判断
如果10分钟的范围内只有1次，等待11分钟的时候又有一次，那么不告警。沉默期为0也不会马上发，最少间隔1分钟

实际测试的结果：

silence-period 的配置，这个配置决定了告警后静默时间，假如当前服务一直有问题，而且silence-period = 0 ，那么1分钟推送一次告警。如果silence-period = 1，那么就是2分钟推送一次告警(建立在当前服务一直有问题的前提下)

## 监控页面UI指标概念

`CPM`: 每分钟请求调用的次数
`SLA`: 服务等级协议（简称：SLA，全称：service level agreement）

是在一定开销下为保障服务的性能和可用性。

网站服务可用性SLA，9越多代表全年服务可用时间越长服务更可靠，停机时间越短

1年 = 365天 = 8760小时

99.9 = 8760 * 0.1% = 8760 * 0.001 = 8.76小时

99.99 = 8760 * 0.0001 = 0.876小时 = 0.876 * 60 = 52.6分钟

99.999 = 8760 * 0.00001 = 0.0876小时 = 0.0876 * 60 = 5.26分钟

从以上看来，全年停机5.26分钟才能做到99.999%，即5个9


`百分位数`：skywalking中有P50，P90，P95这种统计口径，就是百分位数的概念。

例如，p99 520 表示过去 1% 请求的平均延迟为 0.52 秒，99%的请求低于 0.52；p95 300 表示 95%的请求响应时间低于 0.3 秒

`应用性能指数（APDEX）`通过计算分数来反映系统状况，计算规则由多个指标组成

## k8s部署

总共用到下面这几个文件，目前官方的部署是基于helm来做的，只能自己编写yaml文件了

使用官方最新版 8.0.0 版本镜像，8.0.0-es7(oap) 8.0.0(ui)

skywalking-min-oap-configmap.yaml
skywalking-min-oap-deployment.yaml
skywalking-min-oap-namespace.yaml
skywalking-min-oap-service.yaml
skywalking-min-oap-serviceaccount.yaml
skywalking-min-ui-deployment.yaml
skywalking-min-ui-service.yaml

具体分为两个模块，oap和ui

其中ui比较简单，连接oap:12800服务即可

oap涉及到两个端口，暴露这两个端口

1. grpc 11800  提供远程调用，代理接入
2. rest 12800  提供GraphQL API

然后挂载配置文件，从源码上拷贝配置文件，并通过configmap挂载，注意只挂载自己需要的，具体挂载文件内容和路径参考容器自身情况
一般只需要挂载配置文件和告警规则文件，如果需要定制日志，那么再挂载log配置文件

配置文件注意：

主要是storage存储配置，这里采用es，需要修改es配置

然后集群模式采用k8s的服务发现，为此需要k8s rbac服务，在yaml中有定义一个service-account，不然没有访问权限无法使用k8s做集群模式

## ES调整

具体参考es部分

1. 需要调整分片数大小，通过kibana

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

2. 调整max_buckets 过小异常

3. 配置文件修改后重启，写入线程大小修改： thread_pool.write.queue_size: 1000

4. 调用链优化：index.max_result_window: 1000000  默认只能返回10000，可能调用链太长需要修改这个配置

---
weight: 1
title: "Skywalking 告警配置与 Naco 集成实践"
subtitle: "Skywalking Alarm Config"
date: 2020-09-04T14:41:38+08:00
lastmod: 2020-09-05T14:41:38+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Skywalking Alarm Config"
# featuredImagePreview: "images/base-image.jpg"
# featuredImage: "/images/base-image.jpg"
resources:
- name: "featured-image"
  src: "images/base-image.jpg"

tags: [Note]
categories: [Cloud-Native]

lightgallery: true

toc:
  auto: false
---

Skywalking 链路追踪告警配置与Naco集成可用性实践ss 

<!--more-->

# 告警

告警由一组规则驱动，这些规则定义在 `config/alarm-settings.yml`，在k8s的部署中，我们把这个配置文件映射到外部，方便修改

可以监控的指标：

1. 服务监控
2. 实例监控
3. 服务与服务之间监控
3. 实例与实例之间监控
4. 端点监控(就是接口路径)
5. 端点和端点之间监控
6. JVM和数据库访问监控
7. 通过扩展插件监控其它指标

## 告警配置与OAL

以下是一个我们定义的告警规则

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
  # 告警内容
  message: Response time of service instance {name} is more than 10ms in 2 minutes of last 10 minutes
```

涉及到一个`OAL`的概念，OAL是官方的一种查询语句，是对Metrics数据的一种转换，比如通过自定义OAL表达式，计算服务平均响应时间

除了以上的配置，还可用配置`include-names`和`exclude-names`对监控实体做限制，排除不想纳入监控的

具体配置可用参考: https://github.com/VanLiuZhi/document-cn-translation-of-skywalking/blob/master/docs/zh/8.0.0/setup/backend/backend-alarm.md

## 告警推送

告警产生后，有两种方式接收到告警数据

`Webhook` 和 `gRPCHook`

### Webhook

SkyWalking 的告警 Webhook 要求对等方是一个 Web 容器。告警的消息会通过 HTTP 请求进行发送, 请求方法为 POST, Content-Type 为 application/json, JSON 格式基于 `List<org.apache.skywalking.oap.server.core.alarm.AlarmMessage>` 包含以下信息.

```toml
scopeId. 所有可用的 Scope 请查阅 org.apache.skywalking.oap.server.core.source.DefaultScopeDefine.
name. 目标 Scope 的实体名称.
id0. Scope 实体的 ID.
id1. 未使用.
ruleName. 您在 alarm-settings.yml 中配置的规则名.
alarmMessage. 报警消息内容.
startTime. 告警时间, 位于当前时间与 UTC 1970/1/1 之间
```

我们准备一个接口，接收这个数据即可，数据结构如下

```json
[{
	"scopeId": 1, 
        "name": "serviceA", 
	"id0": 12,  
	"id1": 0,  
    "ruleName": "service_resp_time_rule",
	"alarmMessage": "alarmMessage xxxx",
	"startTime": 1560524171000
}, {
	"scopeId": 1,
        "name": "serviceB",
	"id0": 23,
	"id1": 0,
    "ruleName": "service_resp_time_rule",
	"alarmMessage": "alarmMessage yyy",
	"startTime": 1560524171000
}]
```

### gRPCHook

告警消息将使用 Protobuf(Google Protocol Buffer，谷歌设计的一种高效的数据通信格式，类似Json) 类型通过gRPC远程方法发送. 消息格式，以下文件定义了关键信息 `oap-server/server-alarm-plugin/src/main/proto/alarm-hook.proto`

协议的部分内容如下:

```
message AlarmMessage {
    int64 scopeId = 1;
    string scope = 2;
    string name = 3;
    string id0 = 4;
    string id1 = 5;
    string ruleName = 6;
    string alarmMessage = 7;
    int64 startTime = 8;
}
```

## 配置实践

光看文档的描述和介绍，还是很难理解到SkyWalking告警推送的一个流程，通过实践我们举几个例子来帮助理解

### 服务响应时间告警

配置文件如下，使用的OAL是service_resp_time，这是系统提供的

```yaml
# 服务平均响应时间（高危）
service_resp_time_HIGH_RISK_rule:
  # 指标名称，要和oal中设置的对应。规则命名唯一，必须以 `_rule` 结尾
  metrics-name: service_resp_time
  # 比较符
  op: ">"
  # 阈值
  threshold: 1500
  # 多久检查一次当前的指标数据是否符合告警规则
  period: 10
  # 达到多少次告警后，发送告警消息
  count: 5
  # 在多久时间之内，忽略相同的告警消息
  silence-period: 20
  message: 服务 {name} 平均响应时间10分钟内超过 1500ms
```

该配置的具体告警流程如下:

- period是周期，是评判的范围，单位是分钟，现在设置为10，那么count次数就是在10分钟内如果有5次指标超过阈值1500，就会触发告警，然后静默20分钟，继续循环
- 静默期间，如果产生了告警，并且也达到count次数，也不会触发告警，这个告警会在静默期结束后再推送
- 10分钟的范围，假设前30秒就有5次达到阈值，那么就会触发告警，接着进入沉默期，然后继续判断
- 如果10分钟的范围内只有1次，等待11分钟的时候又有一次，那么不告警
- 沉默期为0也不会马上发，最少间隔1分钟

{{< admonition type=tip title="Tip" open=true >}}
要特别注意的就是第二条，假设20分钟的静默期内，满足阈值，满足次数，那么是要推送一条告警的，如果又有一次，或者3次满足了，那么不会推送，要等到20分钟到了，才会推送，而且只推送一次。也就是这期间重复的告警要到静默期结束后才推送，并且不重复推送
{{< /admonition >}}

### 端点错误告警(自定义OAL)

默认官方提供了很多OAL，但是也要需要实现一些特殊业务的场景，比如我们要监控端点是否有报错，这个时候就可用自定义一个OAL表达式

找到`config`目录下的`official_analysis.oal`文件，`8.0.0`版本指标被分类为多个`.oal`文件

增加一条配置

```
endpoint_status = from(Endpoint.*).filter(status == false).cpm();
```

{{< admonition info 含义 >}}
该表达式的含义是: 
获取端点的Metrics，过滤出状态为false，为false的就是有报错的，然后计算该端点的cpm，也就是每分钟请求次数
{{< /admonition >}}

OAL更多概念参考文档: https://github.com/VanLiuZhi/document-cn-translation-of-skywalking/blob/master/docs/zh/8.0.0/concepts-and-designs/oal.md

然后我们定义一个告警规则

```yaml
endpoint_failed_LOW_RISK_rule:
  metrics-name: endpoint_status
  threshold: 0
  op: ">"
  period: 10
  count: 1
  silence-period: 3
  message: 端点 {name} 1分钟内的请求次数，错误数>=1 (触发条件10分钟内有过1次)
```

由于OAL是一个泛用的表达式，它针对全部的端点，所以我们只能去计算端点错误的cpm，只要端点有错误，那么cpm就会有值

我们定义threshold为0，因为假设只有一次报错，那么cpm就是1，而官方提供的op操作没有>=这种，所以阈值设置为0

场景举例：

1. 端点A此时报错，由于我们threshold设置0，count 1，满足要求，产生告警。之后如果还发生告警，只要在3分钟内，都不推送，3分钟到会推送一次

{{< mermaid >}}
sequenceDiagram
    participant 20：01
    participant 20：04
    20：01->>20：04: 第一次满足条件产生告警
    Note right of 20：06: 告警时序 <br/>Loping...
    20：01-->>20：04: 第二次不再告警
    20：04->>20：06: 静默期过，推送第二条告警
    20：04-->20：06: 产生第三条告警，但是前面已经推送了，需等待静默期结束
{{< /mermaid >}}

{{< admonition danger 特别注意 >}}
threshold如果设置为1，那么必须产生2条错误才能推送，这样1条错误的数据就无法告警了
{{< /admonition >}}

以上是自定义OAL监控端点错误的使用实践，官方默认的配置中是没有提供这个指标的，算是一种曲线救国的策略

**有些时候我们会觉得SkyWalking本身不够灵活，个人感觉这是作者对于软件本身运用场景的一种考量，SkyWalking是针对微服务大规模场景下的链路追踪分析工具，不会把关注点落在某一个端点的错误上，你会发现默认的指标中都是针对服务取一个百分比的，官方也提供了自己扩展的能力，不过从使用场景来看，你不应该这么做，应该从宏观的角度来分析问题，在大规模的集群中，1，2次的端点错误是很正常的**

{{< admonition warning >}}
在实践中发现，服务连接到MQ之后，本次消息处理失败，抛异常的时候，UI界面可用记录到本次错误，但是我们配置的端点监控无法告警，猜测也是因为MQ的特殊性，并不在端点中，MQ本身也没有纳入Agent
{{< /admonition >}}

## 告警动态配置(与Nacos集成)

由于默认的配置是写死在文件中的，无法动态修改，所以我们可用采用Nacos来加载配置文件，当修改配置文件后，会同步覆盖之前的配置
在6.5.0之后的版本中，可以通过配置中心来动态修改告警规则配置（可用使用多种类型的配置中心，这里我们采用nacos）

找到配置文件，我们加上nacos的地址和端口

```yaml
configuration:
  nacos:
    # Nacos Server Host
    serverAddr: 10.90.xx.xx
    # userName: nacos
    # password: nacos
    # Nacos Server Port
    port: 8848
    # Nacos Configuration Group
    group: 'skywalking'
    # Nacos Configuration namespace
    namespace: ''
    # Unit seconds, sync period. Default fetch every 60 seconds.
    period : 10
    # the name of current cluster, set the name if you want to upstream system known.
    clusterName: "default"
```

去nacos中，增加这几个Key，就可以覆盖配置文件中的配置

| Config Key | Value 描述 | Value 格式示例 |
|:----|:----|:----|
|receiver-trace.default.slowDBAccessThreshold| 数据库慢语句阀值, 覆盖 `applciation.yml` 中的 `receiver-trace/default/slowDBAccessThreshold` | default:200,mongodb:50|
|receiver-trace.default.uninstrumentedGateways| 需要网关，则重写 gateways.yml 文件. | 与 gateways.yml 文件一致|
|alarm.default.alarm-settings| 告警设置, 需要重写 alarm-settings.yml 文件. | 与 alarm-settings.yml 文件一致|
|core.default.apdexThreshold| apdex 阈值设置, 需要重写 service-apdex-threshold.yml 文件.|与 service-apdex-threshold.yml 文件一致|
|core.default.endpoint-name-grouping| 端点名称分组设置, 需要重写 endpoint_name_grouping.yml 文件.| 与 endpoint_name_grouping.yml 文件一致|

以上是可用动态修改的配置，比较常用的就是告警配置

{{< admonition warning >}}
特别注意，官方只提供了nacos的IP和Port配置，nacos是一个快速发展的配置中心组件，如果你的配置中心使用了鉴权体系，会导致nacos无法连接
{{< /admonition >}}

### 实践总结

`period : 60` 通过这个参数决定nacos的配置生效的时间。配置60的时候，当我们修改nacos的告警配置，那么1分钟后才会同步配置

### 关于服务健壮性

我们的SkyWalking是采用K8s部署的集群模式，只要k8s服务正常，SkyWalking就是高可用的，但是我们接入了nacos，如果nacos挂了会怎样呢？

- 修改配置，默认连接nacos，然后重启服务，此时nacos配置生效
- 直接停止nacos服务，发现SkyWalking并不会把告警配置恢复成自身配置文件设置的值，说明SkyWalking缓存了nacos配置

由于没有去研究源码，从日志分析的结果来看，SkyWalking同步配置的机制是要收到nacos的配置更新请求，直接挂掉nacos，并不影响SkyWalking

当我们在nacos中直接把配置文件写空，这个更改就会推送给SkyWalking导致告警配置错误

{{< admonition >}}
当然并不是说这就是万无一失的策略，如果k8s重启了SkyWalking，那么就要去nacos拉取配置，此时nacos挂了，就只能采用默认配置。
所以nacos的服务高可用还是有必要的。由于告警配置不会频繁修改，也可以把告警配置写一份在配置文件中，保证当nacos挂了之后，能使用配置文件中的配置保证告警策略能正常工作
{{< /admonition >}}

---
weight: 1
title: "kube-prometheus 监控 kubernets 总结"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "kube-prometheus 监控 kubernets 总结"
resources:
- name: "base-image"
  src: "base-image.jpg"

tags: [Linux, Docker, Note, Cloud]
categories: [Cloud-Native]

lightgallery: true

toc:
  auto: false
---

## 概述

云原生是近年讨论热度非常高的话题，kubernetes作为容器编排的事实标准被广泛运用，对于k8s的监控也是企业云服务建设的重要一环
本文将总结 k8s 监控组件kube-prometheus的使用和注意细节，算是对踩坑的一次总结。如果想了解各个组件的具体作用和用法建议去搜索一下相关资料

## 流程总结

总体概况一下要做的事情

1. 替换镜像(如果需要)

2. 调整yaml文件

3. 部署 promethues，grafana，alertManager，node-exporter(批量部署即可)

4. 数据持久化，配置调整

## 关于kube-prometheus

prometheus是非常流行的监控服务，但是对于k8s的监控需要做很多接口，这导致了prometheus监控K8s比较复杂，官方就出了`kube-prometheus`这么一个项目，是`Prometheus Operator`的定制版，如果我们想获取完整的k8s服务监控指标，推荐采用kube-prometheus的方式

这里需要我们先引申2个概念Jsonnet和CRD，通过官方文档的介绍了解到，kube-prometheus可以通过Jsonnet去修改定制的服务，不过一般也不需要这么做，这是给想去改源码的同学准备的
而CRD是对kuberneters资源的一种扩展(个人觉得CRD虽然在云原生提及的很多，但是这有一定的学习成本，目前感觉不是很好用)

## 关于版本

我们通过github搜索 prometheus，发现有好多的版本

相信大家比较熟知的还是Prometheus仓库下的版本，当时去网上一搜，找到的也是用这个仓库的来部署的，但是后来发现，这个部署的有问题，虽然说监控k8s，但是你会发现很多指标都没有，基本就是一些cpu和内存的指标，比如你想知道容器状态等指标都是没有的

一开始我以为是版本太低，后来我升级的版本后，发现指标是多了，但是依然没有关键的数据

直到后来我发现，官方还有一个prometheus-operator的仓库，这名字一看，明显是为了监控kubernetes

仓库分为两个版本 `Prometheus Operator`和`kube-prometheus`

这是原文比较

```
Prometheus Operator vs. kube-prometheus vs. community helm chart
The Prometheus Operator uses Kubernetes custom resources to simplifiy the deployment and configuration of Prometheus, Alertmanager, and related monitoring components.

kube-prometheus provides example configurations for a complete cluster monitoring stack based on Prometheus and the Prometheus Operator. This includes deployment of multiple Prometheus and Alertmanager instances, metrics exporters such as the node_exporter for gathering node metrics, scrape target configuration linking Prometheus to various metrics endpoints, and example alerting rules for notification of potential issues in the cluster.

The stable/prometheus-operator helm chart provides a similar feature set to kube-prometheus. This chart is maintained by the Helm community. For more information, please see the chart's readme
```

那么我们应该采用kube-prometheus来部署，kube-prometheus是对Prometheus Operator的一种定制，就是提供了很多默认的配置了，直接就可以使用

### Jsonnet

Jsonnet 是一个帮助你定义JSON的数据的特殊配置语言。Jsonnet 可以对JSON结构里面的元素进行运算，就像模版引擎给纯文本带来好处一样

```Jsonnet
// Jsonnet Example
{
    person1: {
        name: "Alice",
        welcome: "Hello " + self.name + "!",
    },
    person2: self.person1 { name: "Bob" },
}
```

转换编译成JSON是如下：

```json
{
   "person1": {
      "name": "Alice",
      "welcome": "Hello Alice!"
   },
   "person2": {
      "name": "Bob",
      "welcome": "Hello Bob!"
   }
}
```

官方的说法是: 建议使用Jsonnet来增强JSON。Jsonnet是完全向后兼容JSON的，引入了很多新特效，譬如注释、引用、算术运算、条件操作符，数组和对象内含，引入，函数，局部变量，继承等

个人觉得这个东西搞的太复杂了

### Kubernetes Custom Resources

自定义资源，在kubernetes中，我们用资源来概括部署在集群中各种东西，比如deployment就是一种资源

在声明deployment这种资源的时候，有一个kind的配置，表明了资源类型

```yaml
apiVersion: apps/v1
kind: Deployment
```

Kubernetes Custom Resources 就是自定义资源用的，可以扩展k8s，不过这属于比较高级的话题，在Prometheus-operator中有用到

```yaml
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
```

它会定义一些CustomResourceDefinition之类的yaml文件，所以当我们看到kind不是我们熟悉的类型的时候，很可能就是做了扩展

由于平时也不用，不做展开了，官方参考

https://kubernetes.io/zh/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definition-versioning/

## 镜像调整

由于服务器环境需要替换镜像，这里列出了需要调整的镜像和文件位置，方便参考

quay.io/prometheus/alertmanager
quay.io/prometheus/node-exporter:v0.18.1
quay.io/prometheus/prometheus
grafana/grafana:6.4.3
quay.io/coreos/kube-rbac-proxy:v0.4.1
quay.io/coreos/kube-state-metrics:v1.8.0
quay.io/coreos/k8s-prometheus-adapter-amd64:v0.5.0
quay.io/coreos/prometheus-operator:v0.34.0
quay.io/coreos/prometheus-config-reloader:v0.34.0
quay.io/coreos/configmap-reload:v0.0.1

my-register/quay.io/prometheus/alertmanager  `alertmanager-alertmanager.yaml`

my-register/grafana/grafana:6.4.3  `grafana-deployment.yaml`

my-register/quay.io/coreos/kube-rbac-proxy:v0.4.1  `kube-state-metrics-deployment.yaml`
my-register/quay.io/coreos/kube-state-metrics:v1.8.0

my-register/quay.io/prometheus/node-exporter:v0.18.1  `node-exporter-daemonset.yaml`
my-register/quay.io/coreos/kube-rbac-proxy:v0.4.1

my-register/quay.io/coreos/k8s-prometheus-adapter-amd64:v0.5.0  `prometheus-adapter-deployment.yaml`

my-register/quay.io/prometheus/prometheus  `prometheus-prometheus.yaml`
 
my-register/quay.io/coreos/prometheus-operator:v0.34.0  `prometheus-operator-deployment.yaml`
my-register/quay.io/coreos/prometheus-config-reloader:v0.34.0
my-register/quay.io/coreos/configmap-reload:v0.0.1

## k8s 与 版本对照

| kube-prometheus stack | Kubernetes 1.14 | Kubernetes 1.15 | Kubernetes 1.16 | Kubernetes 1.17 | Kubernetes 1.18 | Kubernetes 1.19 |
|-----------------------|-----------------|-----------------|-----------------|-----------------|-----------------|-----------------|
| `release-0.3`         | ✔               | ✔               | ✔               | ✔               | ✗               | ✗               |
| `release-0.4`         | ✗               | ✗               | ✔ (v1.16.5+)    | ✔               | ✗               | ✗               |
| `release-0.5`         | ✗               | ✗               | ✗               | ✗               | ✔               | ✗               |
| `release-0.6`         | ✗               | ✗               | ✗               | ✗               | ✔               | ✗               |
| `HEAD`                | ✗               | ✗               | ✗               | ✗               | ✔               | ✗               |

根据k8s版本对照，采用release-0.3版本

## 部署和卸载

参考官方的文档来部署即可

```sh
# Create the namespace and CRDs, and then wait for them to be availble before creating the remaining resources
kubectl create -f manifests/setup
until kubectl get servicemonitors --all-namespaces ; do date; sleep 1; echo ""; done
kubectl create -f manifests/
```

卸载

```sh
kubectl delete --ignore-not-found=true -f manifests/ -f manifests/setup
```

## yaml调整

由于官方的文件是基础版本的，还需要我们调整和修改

### 增加采集monitor

普罗米修斯前端页面Targets会发现这2个任务没有采集到

`monitoring/kube-controller-manager`
`monitoring/kube-scheduler`

关于监控是由ServiceMonitor类型的资源对象来实现的，找到和这两个相关的yaml

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  labels:
    k8s-app: kube-controller-manager
  name: kube-controller-manager
  namespace: monitoring
spec:
  endpoints:
  - interval: 30s
    metricRelabelings:
    - action: drop
      regex: etcd_(debugging|disk|request|server).*
      sourceLabels:
      - __name__
    port: http-metrics
  jobLabel: k8s-app
  namespaceSelector:
    matchNames:
    - kube-system
  selector:
    matchLabels:
      k8s-app: kube-controller-manager
```

比如kube-controller-manager，发现是去找对应的service，而kube-system下面是没有`k8s-app: kube-controller-manager`这个标签对应的资源，所以我们要自己创建它

`prometheus-KubeControllerManagerService.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: kube-system
  name: kube-controller-manager
  labels:
    k8s-app: kube-controller-manager
spec:
  # 使用了selector不需要指定 endpoints
  selector:
    component: kube-controller-manager
  ports:
  - name: https-metrics
    port: 10252
    targetPort: 10252
    protocol: TCP
---
# apiVersion: v1
# kind: Endpoints
# metadata:
#   labels:
#     k8s-app: kube-controller-manager
#   name: kube-controller-manager
#   namespace: kube-system
# subsets:
#   - addresses:
#       - ip: 10.90.xx.170
#       - ip: 10.90.xx.171
#       - ip: 10.90.xx.172
#     ports:
#     - name: https-metrics
#       port: 10252
#       protocol: TCP
```

`prometheus-kubeSchedulerService.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  namespace: kube-system
  name: kube-scheduler
  labels:
    k8s-app: kube-scheduler
spec:
  # 使用了selector不需要指定 endpoints
  selector:
    component: kube-scheduler
  ports:
  - name: http-metrics
    port: 10251
    targetPort: 10251
    protocol: TCP
---
# apiVersion: v1
# kind: Endpoints
# metadata:
#   labels:
#     k8s-app: kube-scheduler
#   name: kube-scheduler
#   namespace: kube-system
# subsets:
#   - addresses:
#       - ip: 10.90.xx.170
#       - ip: 10.90.xx.171
#       - ip: 10.90.xx.172
#     ports:
#     - name: http-metrics
#       port: 10251
#       protocol: TCP
```

增加这两个service后，prometheus采集任务就全部正常了

### 服务端口调整

默认是通过内部端口访问的，官方通过代理来演示，生产部署的话一般就通过NodePort或者Ingress

采用NodePort的方式，调整下面这几个文件

`alertmanager-service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    alertmanager: main
  name: alertmanager-main
  namespace: monitoring
spec:
  type: NodePort  # 类型改为 NodePort
  ports:
  - name: web
    port: 9093
    targetPort: web
    nodePort: 30300   # 增加 nodPort 端口
  selector:
    alertmanager: main
    app: alertmanager
  sessionAffinity: ClientIP
```

```sh
vim grafana-service.yaml      # 修改同上
vim prometheus-service.yaml   # 修改同上
```

## 数据持久化

官方默认的部署方式不是数据持久化的，如果有对应的需求，需要我们自己来修改

### prometheus

官方采用StorageClass来进行持久化的

这里可能就要展开 `pv` `pvc` `storageClass` 的区别和概念了。参考这篇博文 https://www.cnblogs.com/rexcheny/p/10925464.html

总的来说要使用StorageClass并不容易，它需要nfs服务，nfs插件服务(提供kubernetes去动态创建pvc,pc的能力)

首先我们要准备nfs服务，然后参考官方的方式，部署插件服务 https://github.com/kubernetes-retired/external-storage/tree/master/nfs-client 

定义StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: prometheus-data-db
provisioner: 填写插件服务名称
```

这样我们在 prometheus 自定义资源对象中，去指定存储就行了

```yaml
storage:
  volumeClaimTemplate:
    spec:
      storageClassName: prometheus-data-db
      resources:
        requests:
          storage: 100Gi
```

修改完成后，更新prometheus-prometheus.yaml，pv和pvc就会自动创建了，这就是StorageClass的作用


Kube-Prometheus数据持久时间，通过`retention`参数来实现

其它配置参考：https://github.com/coreos/prometheus-operator/blob/0e6ed120261f101e6f0dc9581de025f136508ada/Documentation/prometheus.md

### grafana 

grafana就比较简单了，定义pv和pvc

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: grafana
spec:
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    server: 10.90.xx.xx
    path: /home/k8s/grafana
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: grafana
  namespace: monitoring
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

把deployment的挂载处修改一下

```yaml
volumes:
- name: grafana-storage
  persistentVolumeClaim:
    claimName: grafana
# - emptyDir: {}
#   name: grafana-storage
```

## 告警配置

由于使用了CRD，prometheus-kube 的配置和传统的yaml文件不同

prometheus-operator和直接部署prometheus区别是operator把prometheus, alertmanager server 的配置, 还有scape config, record/alert
rule 包装成了k8s中的CRD，我们可以通过命令去查看自定义资源对象

```s
[root@k8s-master1 ~]# kubectl get crd | grep monitoring
alertmanagers.monitoring.coreos.com           2020-08-25T03:16:26Z
podmonitors.monitoring.coreos.com             2020-08-25T03:16:26Z
prometheuses.monitoring.coreos.com            2020-08-25T03:16:26Z
prometheusrules.monitoring.coreos.com         2020-08-25T03:16:26Z
servicemonitors.monitoring.coreos.com         2020-08-25T03:16:26Z
```

prometheus和alertmanager的pod中都有监控configmap/secrets的container，如果挂载的配置文件发生了修改(k8s同步configmap需要1分钟左右)，会让server进程reload配置文件

而告警配置文件就在 alertmanager-secret.yaml 中，通过base64加密了

所以要自定义告警配置，修改这个文件就可以了

可以明文发布，也可以加密成base64。这样配置的流程就和常规的yaml文件接轨了

如果要新增告警规则的话，去修改`./prometheus-rules.yaml`这个文件即可

```yaml
apiVersion: v1
data: {}
kind: Secret
metadata:
  name: alertmanager-main
  namespace: monitoring
stringData:
  alertmanager.yaml: |-
    global:
      resolve_timeout: 1m # 处理超时时间，超过这个时间未处理，会标记为resolved（已解决）
 
    receivers:
    - name: 'web.hook'
      webhook_configs:
      - url: 'http://xxx/xx/sw/prometheus'
    
    route:
      group_by:             # 分组标签从自定义标签中选择，也可以从原始数据的标签中选择
      - job
      group_interval: 10s
      group_wait: 30s
      receiver: 'web.hook'
      repeat_interval: 50s
      # routes:
      # - match:
      #     alertname: Watchdog
      #   receiver: null  

    # route:
    #   group_interval: 1m  # 在发送新警报前的等待时间(针对同一个分组的等待间隔)
    #   group_wait: 10s     # 最初即第一次等待多久时间发送一组警报的通知(这可以把多条数据合并到一个分组里面，具体合并多少由这个时间间隔决定)
    #   receiver: Default   # 接收对象
    #   repeat_interval: 1m # 发送重复警报的周期
type: Opaque
```

## alertmanager 配置

Alertmanager的配置主要包含两个部分：路由(route)以及接收器(receivers)。
有的告警信息都会从配置中的顶级路由(route)进入路由树，根据路由规则将告警信息发送给相应的接收器。
子路由: 所有的告警信息从顶级路由开始，根据标签匹配规则进入到不同的子路由，并且根据子路由设置的接收器发送告警
在Alertmanager中可以定义一组接收器，比如可以按照角色(比如系统运维，数据库管理员)来划分多个接收器。接收器可以关联邮件，Slack以及其它方式接收告警信息。

alertmanager 配置

```
全局配置（global）：用于定义一些全局的公共参数，如全局的SMTP配置，Slack配置等内容，有几个默认参数，如果不覆盖会用默认的；
模板（templates）：用于定义告警通知时的模板，如HTML模板，邮件模板等；
告警路由（route）：根据标签匹配，确定当前告警应该如何处理；
接收人（receivers）：接收人是一个抽象的概念，它可以是一个邮箱也可以是微信，Slack或者Webhook等，接收人一般配合告警路由使用；
抑制规则（inhibit_rules）：合理设置抑制规则可以减少垃圾告警的产生
```

全局配置中需要注意的是resolve_timeout，该参数定义了当Alertmanager持续多长时间未接收到告警后标记告警状态为resolved（已解决）。该参数的定义可能会影响到告警恢复通知的接收时间，读者可根据自己的实际场景进行定义，其默认值为5分钟

路由配置，一般是复杂一点是要用子路由，简单来说就是可以对不同的告警分发到不同的接收者
如果警报已经成功发送通知, 如果想设置发送告警通知之前要等待时间，则可以通过repeat_interval参数进行设置。

prometheus 中关于告警的配置

```
alert：告警规则的名称。
expr：基于PromQL表达式告警触发条件，用于计算是否有时间序列满足该条件。
for：评估等待时间，可选参数。用于表示只有当触发条件持续一段时间后才发送告警。在等待期间新产生告警的状态为pending。
labels：自定义标签，允许用户指定要附加到告警上的一组附加标签，在接收数据的时候，可以把原生告警中的标签数据也取出来
annotations：用于指定一组附加信息，比如用于描述告警详细信息的文字等，annotations的内容在告警产生时会一同作为参数发送到Alertmanager（在利用Java DTO接收数据的时候，annotations这个对象的成员变量就必须定义好，否则无法解析数据，可以通过内置的占位符替换变量，可以获取到原始数据的标签内容）
```

告警的计算周期默认是1分钟

告警状态：

当某个规则产生告警的时候，假如有2个符合条件，那么页面active会记录为2，此时这些active的告警会经历下面两种状态

pending: 告警已经产生，进入这种状态，但是还不发送告警 

firing: 告警有等待时间，假设是1分钟，说明1分钟内告警仍然持续，就会转为firing状态，发送告警

### 内置告警规则

对几个常用的告警规则做说明

Watchdog 看门狗服务，该告警一直处于报警状态
KubeContainerWaiting 获取处于等待状态的容器
KubeDeploymentGenerationMismatch 部署失败，但尚未成功回滚
KubeDeploymentReplicasMismatch 副本数不匹配
KubePodCrashLooping 容器处于重启循环中
KubePodNotReady pod未进入准备状态
KubeClientErrors k8s API server client 发生错误
CPUThrottlingHigh 
KubeControllerManagerDown 控制器服务宕机
KubeNodeNotReady 节点未进入就绪状态
KubeNodeUnreachable 节点不可达
KubeletDown Kubelet 服务宕机
KubeSchedulerDown 调度服务宕机


## 总结

1. 对于k8s的容器监控，通过搜索资料就会找到用 prometheuses ，但是基本都是用的官方版本，只能监控部分数据，完整的监控能力要使用 prometheuses-kube 这个项目

2. prometheuses-kube GitHub 有详细的使用说明，直接clone源码就能部署了，不过差了一部分东西，需要自己手动调整

3. prometheuses-kube源码是采用Jsonnet来编写的，这个就理解成json的加强可编程语言，我们不需要关注它

4. prometheuses-kube 由于采用了CRD的方式，修改配置不再是传统的yaml文件对象，按照官方的说法去修改即可自定义告警等配置

5. 如果需要数据持久化还需要准备文件系统等存储服务，prometheuses 采用的是storageclass，一般大家都是用nfs，需要部署插件服务才能使用。grafana由于是传统的deployment部署的，直接挂载pvc即可(如果是使用阿里云这些云服务器，一般都会提供文件存储服务，不过需要做监控的一般是自建云服务企业)

## 参考

https://github.com/prometheus-operator/kube-prometheus/tree/release-0.3

https://blog.z0ukun.com/?p=2605

https://blog.csdn.net/footrip/article/details/104572761/

https://finolo.gy/2019/12/Kubernetes%E7%9B%91%E6%8E%A7%E6%96%B9%E6%A1%88kube-prometheus-prometheus-node-exporter-grafana/

https://www.freesion.com/article/3311495457/

https://www.yuque.com/idevops/mrpe7c/pha0gg

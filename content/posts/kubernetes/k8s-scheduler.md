---
weight: 1103
title: "K8s Scheduler 调度相关总结"
subtitle: "K8s Scheduler"
date: 2020-09-07T16:44:29+08:00
lastmod: 2020-09-07T16:44:29+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "K8s Scheduler"
featuredImagePreview: "/images/Kubernetes/k8s-scheduler.jpg"
featuredImage: "/images/Kubernetes/k8s-scheduler.jpg"
# resources:
# - name: "featured-image"
#   src: "images/base-image.jpg"

tags: [cloud-native, k8s]
categories: [Kubernetes] 

lightgallery: true

toc:
  auto: false
---

kubernetes Pod 调度策略，包括亲和性，污点容忍，优先级等运用

<!--more-->

## kube-scheduler 

k8s通过 kube-scheduler 来实现pod调度，主要任务是把pod分配到集群的节点上，是默认的调度器

主要的工作流程：调度器通过 kubernetes 的 watch 机制来发现集群中新创建且尚未被调度到 Node 上的 Pod。调度器会将发现的每一个未调度的 Pod 调度到一个合适的 Node 上来运行

虽然看似简单，但是一个合理的调度器应该要解决一下问题

公平：如何保证每个节点都能被分配到资源
资源高效利用：集群所有资源最大化被使用
效率：调度性能要好，能够尽快的对大批量的Pod完成调度工作
灵活：允许用户根据自己的需求控制调度的流程

kubernetes在这一块是很灵活的，通过一定的调度流程保证资源高效使用和节点公平性，并且用户可用自定义调度器替换默认的调度策略

## 调度流程

调度流程主要由两个部分组成

1. 过滤 predicate
2. 打分 priority

过滤就是先排除掉不满足要求的节点，然后对节点的优先级排序，优先级最高的被调度到

### 过滤策略

- `PodFitsHostPorts`：如果 Pod 中定义了 hostPort 属性，那么需要先检查这个指定端口是否 已经被 Node 上其他服务占用了。

- `PodFitsHost`：若 pod 对象拥有 hostname 属性，则检查 Node 名称字符串与此属性是否匹配。

- `PodFitsResources`：检查 Node 上是否有足够的资源（如，cpu 和内存）来满足 pod 的资源请求。

- `PodMatchNodeSelector`：检查 Node 的 标签 是否能匹配 Pod 属性上 Node 的 标签 值。

- `NoVolumeZoneConflict`：检测 pod 请求的 Volumes 在 Node 上是否可用，因为某些存储卷存在区域调度约束。

- `NoDiskConflict`：检查 Pod 对象请求的存储卷在 Node 上是否可用，若不存在冲突则通过检查。

- `MaxCSIVolumeCount`：检查 Node 上已经挂载的 CSI 存储卷数量是否超过了指定的最大值。

- `CheckNodeMemoryPressure`：如果 Node 上报了内存资源压力过大，而且没有配置异常，那么 Pod 将不会被调度到这个 Node 上。

- `CheckNodePIDPressure`：如果 Node 上报了 PID 资源压力过大，而且没有配置异常，那么 Pod 将不会被调度到这个 Node 上。

- `CheckNodeDiskPressure`：如果 Node 上报了磁盘资源压力过大（文件系统满了或者将近满了）， 而且配置异常，那么 Pod 将不会被调度到这个 Node 上。

- `CheckNodeCondition`：Node 可以上报其自身的状态，如磁盘、网络不可用，表明 kubelet 未准备好运行 pod。 如果 Node 被设置成这种状态，那么 pod 将不会被调度到这个 Node 上。

- `PodToleratesNodeTaints`：检查 pod 属性上的 tolerations 能否容忍 Node 的 taints。

- `CheckVolumeBinding`：检查 Node 上已经绑定的和未绑定的 PVCs 能否满足 Pod 对象的存储卷需求

### 打分策略

- `SelectorSpreadPriority`：尽量将归属于同一个 Service、StatefulSet 或 ReplicaSet 的 Pod 资源分散到不同的 Node 上。

- `InterPodAffinityPriority`：遍历 Pod 对象的亲和性条目，并将那些能够匹配到给定 Node 的条目的权重相加，结果值越大的 Node 得分越高。

- `LeastRequestedPriority`：空闲资源比例越高的 Node 得分越高。换句话说，Node 上的 Pod 越多，并且资源被占用的越多，那么这个 Node 的得分就会越少。

- `MostRequestedPriority`：空闲资源比例越低的 Node 得分越高。这个调度策略将会把你所有的工作负载（Pod）调度到尽量少的 Node 上。

- `RequestedToCapacityRatioPriority`：为 Node 上每个资源占用比例设定得分值，给资源打分函数在打分时使用。

- `BalancedResourceAllocation`：优选那些使得资源利用率更为均衡的节点。

- `NodePreferAvoidPodsPriority`：这个策略将根据 Node 的注解信息中是否含有 scheduler.alpha.kubernetes.io/preferAvoidPods 来 计算其优先级。使用这个策略可以将两个不同 Pod 运行在不同的 Node 上。

- `NodeAffinityPriority`：基于 Pod 属性中 PreferredDuringSchedulingIgnoredDuringExecution 来进行 Node 亲和性调度。你可以通过这篇文章 Pods 到 Nodes 的分派 来了解到更详细的内容。

- `TaintTolerationPriority`：基于 Pod 中对每个 Node 上污点容忍程度进行优先级评估，这个策略能够调整待选 Node 的排名。

- `ImageLocalityPriority`：Node 上已经拥有 Pod 需要的 容器镜像 的 Node 会有较高的优先级。

- `ServiceSpreadingPriority`：这个调度策略的主要目的是确保将归属于同一个 Service 的 Pod 调度到不同的 Node 上。如果 Node 上 没有归属于同一个 Service 的 Pod，这个策略更倾向于将 Pod 调度到这类 Node 上。最终的目的`：即使在一个 Node 宕机之后 Service 也具有很强容灾能力。

- `CalculateAntiAffinityPriorityMap`：这个策略主要是用来实现pod反亲和。

- `EqualPriorityMap`：将所有的 Node 设置成相同的权重为 1。

更多策略参考官方文档: 

https://v1-16.docs.kubernetes.io/zh/docs/concepts/scheduling/

## 节点亲和性

通过该属性配置节点亲和性 `pod.spec.affinity.nodeAffinity`，主要有以下两种策略

1. `preferredDuringSchedulingIgnoredDuringExecution`：软策略

软策略是偏向于，更想(不)落在某个节点上，但如果实在没有，落在其他节点也可以

2. `requiredDuringSchedulingIgnoredDuringExecution`：硬策略

硬策略是必须(不)落在指定的节点上，如果不符合条件，则一直处于Pending状

### 硬策略

yaml文件示例

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: affinity-required
  labels:
      app: node-affinity-pod
spec:
  containers:
  - name: with-node-required
    image: nginx:1.2.1
    imagePullPolicy: IfNotPresent
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution: # 硬策略
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/hostname    #节点名称
            operator: NotIn        #不是
            values:
            - testcentos7          #node节点
```

该节点亲和性策略为不能调度到label为 kubernetes.io/hostname=testcentos7 的节点上，由于是硬策略，如果没满足条件，pod会一直处于Pending状态

可用修改 `operator` 为 `In` ，这样表示调度到label为 kubernetes.io/hostname=testcentos7 的节点上

### 软策略

再来看看软策略

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: affinity-preferred
  labels:
    app: node-affinity-pod
spec:
  containers:
  - name: with-node-preferred
    image: nginx:1.2.1
    imagePullPolicy: IfNotPresent
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1    #权重为1,软策略中权重越高匹配到的机会越大
        preference:  #更偏向于
          matchExpressions:
          - key: kubernetes.io/hostname #node名称
            operator: In  #等于，为
            values:
            - testcentos7   #node真实名称
```

该节点亲和性策略为，调度到label 为 kubernetes.io/hostname=testcentos7 的节点上，由于是软策略，如果没有满足条件的，也可用调度到其它节点

把 value 改为 node2，那么策略应该是调度到label 为 kubernetes.io/hostname=node2 的节点上，如果集群没有这个节点，那么就调度到其它节点。可用设置多个权重，选取更适合的节点

### 两种策略结合

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: NotIn
          values:
          - k8s-node2
    preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 1 
      preference:
      matchExpressions:
      - key: source
        operator: In
        values:
          - test
```

以上节点亲和性表示为: 不能调度到 label 为 kubernetes.io/hostname=k8s-node2，其它节点都可以，但最好是 source=test 的

### 键值运算关系

- In：label 的值在某个列表里
- NotIn：label 的值不在某个列表中
- Gt：label 的值大于某个值
- Lt：label 的值小于某个值
- Exists：某个 label 存在
- DoesNotExist：某个 label 不存在

如果nodeSelectorTerms下面有多个选项，满足任何一个条件就可以了；如果matchExpressions有多个选项，则必须满足这些条件才能正常调度

## Pod亲和性

节点亲和性描述的是 pod 和 节点的关系，而一个节点上会有多个pod，pod和pod之间也可以设置亲和性，比如某些pod属于同一个业务范围的，它们都在一个节点上的话，网络通信会比较好，这个时候就要用到`Pod亲和性`

通过该属性配置pod亲和性 `pod.spec.affinity.podAffinity/podAntiAffinity`，同样pod亲和性策略也和节点亲和性一样，有软策略和硬策略

- `preferedDuringSchedulingIgnoredDuringExecution`：软策略

软策略是偏向于，更想(不)落在某个节点上，但如果实在没有，落在其他节点也可以

- `requiredDuringSchedulingIgnoredDuringExecution`：硬策略

硬策略是必须(不)落在指定的节点上，如果不符合条件，则一直处于Pending状态

`podAffinity` 和 `podAntiAffinity` 也是和节点亲和性不一样的地方，podAffinity的策略是节点会调度在同一组Node上，podAntiAffinity是节点不会调度在同一组Node上，如果是软策略那么没有满足条件的就接受，硬策略就pod处于pending

### 硬策略同域

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: affinity-required
  labels:
    app: pod-test
spec:
  containers:
  - name: nginx-app
    image: nginx:1.2.1
    imagePullPolicy: IfNotPresent
  affinity:
    podAffinity:   # 在同一域下(同一组Node下)
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app    # 标签key
            operator: In
            values:
            - nginx     # 标签value
        topologyKey: kubernetes.io/hostname  # 域的标准为node节点的名称
```

该策略表示: 此必须要和 label 为 `app：nginx` 的pod在同一node下，如果不存在这样的pod，那么该pod不会被调度，处于pending状态

### 软策略非同域

```yaml
affinity:
  podAntiAffinity:    # 不在同一个域下
    preferedDuringSchedulingIgnoredDuringExecution:
    - weight: 1
      podAffinityTerm:
        labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - nginx-2
        topologyKey: kubernetes.io/hostname
```

该策略表示: 不能和label 为 `app：nginx` 的pod在同一node下，由于是软策略，如果没有那就调度到合适的节点

## 亲和性和非亲和性总结

|调度策略|	匹配标签|	操作符|	拓扑域支持|	调度目标|
|:----:|:----:|:----:|:----:|:----:|
|nodeAffinity| 主机|	In,NotIn,Exists,DoesNotExists,Gt,Lt|	否|	指定主机|
|podAffinity|	Pod|	In,NotIn,Exists,DoesNotExists,Gt,Lt|	是|	pod与指定pod在一拓扑域|
|podAnitAffinity|	Pod|	In,NotIn,Exists,DoesNotExists,Gt,Lt|	是|	pod与指定pod不在一拓扑域|

## 污点(Taint)和容忍(Toleration)

节点亲和性，是`Pod的一种属性`（偏好或硬性要求），它使Pod被吸引到一类特定的节点，Taint则相反，`它使节点能够排斥一类特定的Pod`

Taint与Toleration相互配合，可以用来避免Pod被分配到不合适的节点上，每个节点上都可以应用一个或两个taint，这表示对那些不能容忍这些taint和pod，是不会被该节点接受的，如果将toleration应用于pod上，则表示这些pod可以（但不要求）被调度到具有匹配taint的节点上

- 污点(Taint): 节点配置
- 容忍(Toleration): pod配置

## Taint

taint 是针对节点的配置，节点配置污点后，就可以拒绝pod的调度，甚至驱逐已存在的pod

`kubectl taint` 命令设置污点，参考为 `key=value:effect`

每个污点有一个 key 和 value 作为污点标签，其中 value 可以为空，effect描述污点的作用，当前 taint effect 支持如下三个选项：

- NoSchedule：表示 k8s 不会将Pod调度到具有该污点的Node上
- PreferNoSchedule：表示 k8s 将尽量避免将Pod调度到具有该污点的Node上
- NoExecute：表示 k8s 将不会将Pod调度到具有该污点的Node上，同时会将Node上已有的Pod驱逐出去


{{< admonition >}}
如果只关注污点，key，value是没有太多作用的，是Toleration会用到，用于匹配和筛选节点
{{< /admonition >}}

### 污点的设置、查看和去除

```toml
[root@Centos8 scheduler]# kubectl describe node centos8
Taints:             node-role.kubernetes.io/master:NoSchedule

## 设置污点
kubectl taint node [node name] key1=value:NoSchedule

## 节点说明中，查看Taint字段
kubectl describe node [node name]

## 去除污点，可以不用指定key
kubectl taint node [node name] key1:NoSchedule-
```

设置：`kubectl taint node k8s-worker05 node-role.kubernetes.io/master=:NoSchedule`

删除: `kubectl taint node k8s-worker03 node-role.kubernetes.io/master:NoSchedule-`

当我们把某个节点设置NoExecute，上面的常规pod会被驱逐掉(生命周期会经历一个删除的过程，最后pod删除，不是Evicted状态)，在其它节点被重新调度

## Toleration

设置了污点的Node将根据 taint 的 effect:NoSchedule、PreferNoSchedule、NoExecute和Pod之间产生互斥的关系，Pod将在一定程度上不会被调度到 Node 上

但我们可以在 Pod 上设置容忍(Toleration)，意思是设置了容忍的 Pod 将可以容忍污点的存在，可以被调度到存在污点的Node上

通过 `spec.tolerations` 设置容忍

```yaml
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoSchedule"
  tolerationSeconds: 3600
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoExecute"
- key: "key2"
  operator: "Exists"
  effect: "NoSchedule"
```

- 其中 key、value、effect 要与Node中的 taint 保持一致
- operator 的值为 Exists 将会忽略 value 的值
- tolerationSeconds 用于描述当Pod需要驱逐时可以在Node上继续保留运行的时间

当不指定key时，表示容忍所有污点的key：

```yaml
tolerations:
- operator: "Exists"
  effect: "NoSchedule"
```

上述yaml表示为: 容忍所有污点为NoSchedule的节点，不管key和value

当不指定 offect 值时，表示容忍所有的污点类型

```yaml
tolerations:
- key: "key1"
  value: "value1"
  operator: "Exists"
```

## 指定调度节点

`spec.nodeName` 值为node名称，pod将只会调度到对应节点上

`spec.nodeSelector` 通过标签来匹配

```yaml
spec:
  nodeSelector:
    app: web
```

## 调度策略PriorityClass

参考：https://www.ibm.com/support/knowledgecenter/zh/bluemix_stage/containers/cs_pod_priority.html

除了常用的nodeSelector，k8s还提供了优先级调度，也就是抢占式调度，优先级高的pod会先调度，在资源不足的情况下会牺牲掉低优先级的pod先把高优先级的pod先调度

具体做法就是在资源定义的时候关联一个PriorityClass对象，这个对象会设置优先级

```yaml
apiVersion: scheduling.k8s.io/v1alpha1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: "This priority class should be used for XYZ service pods only."
```

```
globalDefault

	可选：将此字段设置为 true 可将此优先级类设置为全局缺省值，此缺省值将应用于安排的未指定 priorityClassName 值的每个 pod。集群中只能有 1 个优先级类可设置为全局缺省值。如果没有全局缺省值，那么没有指定 priorityClassName 的 pod 的优先级为零 (0)。

缺省优先级类不会设置 globalDefault。如果在集群中创建了其他优先级类，那么可以通过运行 kubectl describe priorityclass <name>.
```

在containers配置中加上 `priorityClassName: high-priority`

系统默认的优先级类

```
名称	设置方	优先级值	用途
system-node-critical	Kubernetes	2000001000	选择在创建集群时部署到 kube-system 名称空间中的 pod 会使用此优先级类来保护工作程序节点的关键功能，例如联网、存储器、日志记录、监视和度量值 pod。
system-cluster-critical	Kubernetes	2000000000	选择在创建集群时部署到 kube-system 名称空间中的 pod 会使用此优先级类来保护集群的关键功能，例如联网、存储器、日志记录、监视和度量值 pod。
ibm-app-cluster-critical	IBM	900000000	选择在创建集群时部署到 ibm-system 名称空间中的 pod 会使用此优先级类来保护应用程序的关键功能，例如负载均衡器 pod
```

`kubectl get priorityclasses`
`kubectl get priorityclass <priority_class> -o yaml > Downloads/priorityclass.yaml`

## 资源配置与预留

https://www.jianshu.com/p/5dcf04e8d56b

https://www.cnblogs.com/dudu/p/12587234.html

## 参考

https://blog.csdn.net/xopqaaa/article/details/103200845



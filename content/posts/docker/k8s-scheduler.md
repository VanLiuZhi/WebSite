---
weight: 1
title: "K8s Scheduler"
subtitle: "K8s Scheduler"
date: 2020-09-07T16:44:29+08:00
lastmod: 2020-09-07T16:44:29+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "K8s Scheduler"
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

打分策略

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

In：label 的值在某个列表里
NotIn：label 的值不在某个列表中
Gt：label 的值大于某个值
Lt：label 的值小于某个值
Exists：某个 label 存在
DoesNotExist：某个 label 不存在

如果nodeSelectorTerms下面有多个选项，满足任何一个条件就可以了；如果matchExpressions有多个选项，则必须满足这些条件才能正常调度

## Pod亲和性

节点亲和性描述的是 pod 和 节点的关系，而一个节点上会有多个pod，pod和pod之间也可以设置亲和性，比如某些pod属于同一个业务范围的，它们都在一个节点上的话，网络通信会比较好
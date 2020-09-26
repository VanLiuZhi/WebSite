---
weight: 1101
title: "Kubernetes 笔记"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Kubernetes 笔记"
featuredImagePreview: "/images/Kubernetes/k8s-base.png"
featuredImage: "/images/Kubernetes/k8s-base.png"
# resources:
# - name: "featured-image"
#   src: "images/base-image.jpg"

tags: [cloud-native, k8s]
categories: [Kubernetes] 

lightgallery: true

toc:
  auto: false
---

Kubernetes 笔记

<!-- more -->

## 基础

各个服务架构一句话总结: Kubernetes是由容器和进程互相配合实现的

这句话的意思是，部分操作，比如apiserver，scheduler是由容器来完成的，而像Kubelet是一个进程。Kubelet 实现了集群中最重要的关于 Node 和 Pod 的控制功能

(kubelet等组件之所以不采用容器部署，是应为要操作宿主机的原因，比如分配资源，网络划分，隔着容器不好做这些事情)

协议规范:

CNI( Container Network Interface) 
CSI（Container Storage Interface） 
CRI（Container Runtime Interface）

etcd: 是一个高可用的分布式键值(key-value)数据库。etcd内部采用raft协议作为一致性算法，有点像zk，k8s集群使用etcd作为它的数据后端，etcd是一种无状态的分布式数据存储集群。数据以key-value的形式存储在其中

swap: 在k8s中最好禁用swap，以免发生集群无法调度问题

flannel: 用来通信的网络插件。要符和CNI规范


## k8s 用的的镜像示例

master节点

| Command                                                         | Description 
| --------------------------------------------------------------- | :---------------------------------: 
| REPOSITORY                                                      | TAGIMAGEIDCREATEDSIZE 
| registry.aliyuncs.com/google_containers/kube-proxy              | v1.14.120a2d703516510monthsago82.1MB 
| registry.aliyuncs.com/google_containers/kube-apiserver          | v1.14.1cfaa4ad74c3710monthsago210MB 
| registry.aliyuncs.com/google_containers/kube-controller-manager | v1.14.1efb3887b411d10monthsago158MB 
| registry.aliyuncs.com/google_containers/kube-scheduler          | v1.14.18931473d5bdb10monthsago81.6MB 
| quay-mirror.qiniu.com/coreos/flannel                            | v0.11.0-amd64ff281650a72112monthsago52.6MB 
| quay.io/coreos/flannel                                          | v0.11.0-amd64ff281650a72112monthsago52.6MB 
| registry.aliyuncs.com/google_containers/coredns                 | 1.3.1eb516548c18012monthsago40.3MB 
| registry.aliyuncs.com/google_containers/etcd                    | 3.3.102c4adeb21b4f14monthsago258MB 
| registry.aliyuncs.com/google_containers/pause                   | 3.1da86e6ba6ca12yearsago742kB 


node节点

| Command                                            | Description 
| -------------------------------------------------- | :------------------------------------------------: 
| REPOSITORY                                         | TAGIMAGEIDCREATEDSIZE 
| registry.aliyuncs.com/google_containers/kube-proxy | v1.14.120a2d703516510monthsago82.1MB 
| quay-mirror.qiniu.com/coreos/flannel               | v0.11.0-amd64ff281650a72112monthsago52.6MB 
| quay.io/coreos/flannel                             | v0.11.0-amd64ff281650a72112monthsago52.6MB 
| registry.aliyuncs.com/google_containers/pause      | 3.1da86e6ba6ca12yearsago742kB 

除了容器外，还需要kubelet才是完整的


## 常用命令

kubectl explain node

kubectl get pods -n kube-system -o wide
kubectl get nodes -o wide  

`kubectl get node` 可以使用 -o 配合 wide json yaml

配合jq做内容提取

kubectl get nodes -o json | jq ".items[] | {name: .metadata.name} + .status.nodeInfo"

## 与k8s交互

在 K8S 中进行部署或者说与 K8S 交互的方式主要有三种：

命令式
命令式对象配置
声明式对象配置

## 镜像拉取策略

默认值是IfNotPresent(在initContainers中，发现不配置并不会去使用本地的，所以最好显示的配置一下)

Always
总是拉取：
首先获取仓库镜像信息，如果仓库中的镜像与本地不同，那么仓库中的镜像会被拉取并覆盖本地。
如果仓库中的镜像与本地一致，那么不会拉取镜像。
如果仓库不可用，那么pod运行失败。

IfNotPresent
优先使用本地：
如果本地存在镜像，则使用本地的，
不管仓库是否可用。
不管仓库中镜像与本地是否一致。

Never
只使用本地镜像，如果本地不存在，则pod运行失败

## 固定node发布

deploy上用选择器，就可以固定在一个node上

## 资源删除

一般是 delete -f .yaml 删除对应的资源，然后apply再应用，资源生效

删除pod: 先删除pod，然后再删除deployment。只删除pod，去查看的时候pod还是存在的，再去删除dep，查看pod消失

## service 和 deployment

deployment 负责创建 pod，只有de存在，pod就会维持住，pod被删除了也会被dev拉起。这样pod的服务是不确定的，它的ip一直在变化

所以需要service

Service 是 Kubernetes 中的一种服务发现机制：

Pod 有自己的 IP 地址
Service 被赋予一个唯一的 dns name
Service 通过 label selector 选定一组 Pod
Service 实现负载均衡，可将请求均衡分发到选定这一组 Pod 中

## 标签操作

kubectl get pods --show-labels　　#查看pod所有标签信息
kubectl get pods -l app　　#过滤包含app的标签
kubectl get pods -l app    #过滤包含app的标签及显示值
kubectl label pods pod-demo release=canary　　#给pod-demo增加标签
kubectl label pods pod-demo release=stable --overwrite　　#修改标签

通过标签来过滤资源

kubectl get pods -l run=myapp
kubectl get pods -l run=myapp --show-labels
kubectl get pods -l run!=client --show-labels

## 通过标签部署到指定节点

nodeSelector 写上标签，调度的时候就只会调度到这几个有标签的节点(该特性在官方准备在新版本中去除了)

nodeName: 指定主机名称，比如 k8s-09

`kubernetes.io/hostname=k8s-09` 一般节点都会打上这个标签，描述了主机名称信息

```yaml
spec:
  # priorityClassName: system-cluster-critical
  nodeSelector:
    kubernetes.io/hostname: k8s-worker06
```

## 命名空间

资源创建的时候，可以指定命名空间，命名空间内各资源可以互相访问
如果pod和service在不同的命名空间，是不能直接访问的，这时候要跨命名空间访问

使用 服务加命名空间的方式  比如服务 是 oap，那么web-b命名空间想访问web-f命名空间的 oap service，使用`oap.web-f`

## 查看pod日志

使用logs命令，加上pod实例，如果pod里面有多个容器，还要加上容器名称，举例如下：

kubectl -n kube-system logs alertmanager-54f5b4447b-2jvzz prometheus-alertmanager

## 引用pod的信息

```yaml
env:
    - name: MY_NODE_NAME
      valueFrom:
        fieldRef:
          fieldPath: spec.nodeName
    - name: MY_POD_NAME
      valueFrom:
        fieldRef:
          fieldPath: metadata.name
    - name: MY_POD_NAMESPACE
      valueFrom:
        fieldRef:
          fieldPath: metadata.namespace
    - name: MY_POD_IP
      valueFrom:
        fieldRef:
          fieldPath: status.podIP
    - name: MY_POD_SERVICE_ACCOUNT
      valueFrom:
        fieldRef:
          fieldPath: spec.serviceAccountName
```

通过环境变量，引用当前pod的container的信息注入到外部

spec.nodeName ： pod所在节点的IP、宿主主机IP

status.podIP ：pod IP

metadata.namespace : pod 所在的namespace

更多参数：
https://kubernetes.io/docs/tasks/inject-data-application/environment-variable-expose-pod-information/

https://github.com/kubernetes/kubernetes/issues/24657


## 单个文件的方式挂载 configMap 

主要是用了subPath这个配置，另外记得volumes挂载不要用驼峰命名，可以用横杆加小写

挂载多个文件

```yaml
volumeMounts:
- mountPath: /skywalking/config/application.yml
    name: config
    subPath: application.yml
- mountPath: /skywalking/config/alarm-settings.yml
    name: alarm-config
    subPath: alarm-settings.yml

volumes:
- name: application-config
  configMap:
    defaultMode: 420
    name: oap-config
    items:
      - key: application.yml
        path: application.yml
- name: alarm-config
  configMap:
    defaultMode: 420
    name: oap-config
    items:
      - key: alarm-settings.yml
        path: alarm-settings.yml
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

## Taints与Tolerations 污点和容忍

```
kubectl taint node [node] key=value[effect]   
     其中[effect] 可取值: [ NoSchedule | PreferNoSchedule | NoExecute ]
      NoSchedule: 一定不能被调度
      PreferNoSchedule: 尽量不要调度
      NoExecute: 不仅不会调度, 还会驱逐Node上已有的Pod

kubectl taint node node1 key1=value1:NoSchedule
kubectl taint node node1 key1=value1:NoExecute
kubectl taint node node1 key2=value2:NoSchedule
```

kubectl describe node node1

删除

kubectl taint node node1 key1:NoSchedule-  # 这里的key可以不用指定value
kubectl taint node node1 key1:NoExecute-
# kubectl taint node node1 key1-  删除指定key所有的effect
kubectl taint node node1 key2:NoSchedule-

master节点设置

kubectl taint nodes master1 node-role.kubernetes.io/master=:NoSchedule


容忍tolerations主节点的taints

以上面为 master1 设置的 taints 为例, 你需要为你的 yaml 文件中添加如下配置, 才能容忍 master 节点的污点

在 pod 的 spec 中设置 tolerations 字段

```
tolerations:
- key: "node-role.kubernetes.io/master"
  operator: "Equal"
  value: ""
  effect: "NoSchedule"
```


## 亲和性和非亲和性

## Jsonnet

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

谷歌：建议使用Jsonnet来增强JSON。Jsonnet是完全向后兼容JSON的，引入了很多新特效，譬如注释、引用、算术运算、条件操作符，数组和对象内含，引入，函数，局部变量，继承等

## Kubernetes Custom Resources

自定义资源，在kubernetes中，我们用资源来概括部署在集群中各种东西，比如deployment就是一种资源

在生命deployment这种资源的时候，有一个kind的配置，表明了资源类型

```yaml
apiVersion: apps/v1
kind: Deployment
```

Kubernetes Custom Resources 就是自定义资源用的，可以扩展k8s，不过这属于比较高级的话题，目前在Prometheus-operator中有见到过

```yaml
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
```

它会定义一些CustomResourceDefinition之类的yaml文件，所以当我们看到kind不是我们熟悉的类型的时候，很可能就是做了扩展

由于平时也不用，不做展开了，官方参考

https://kubernetes.io/zh/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definition-versioning/

## Kubernetes中的开放接口CRI、CNI、CSI

参考 https://zhuanlan.zhihu.com/p/33390023

CRI（Container Runtime Interface）：容器运行时接口，提供计算资源
CNI（Container Network Interface）：容器网络接口，提供网络资源
CSI（Container Storage Interface）：容器存储接口，提供存储资源

## 容器重启策略

Pod 的 spec 中包含一个 restartPolicy 字段，其可能取值包括 Always、OnFailure 和 Never。默认值是 Always

restartPolicy 适用于 Pod 中的所有容器
restartPolicy 仅针对同一节点上 kubelet 的容器重启动作
当 Pod 中的容器退出时，kubelet 会按指数回退 方式计算重启的延迟（10s、20s、40s、...），其最长延迟为 5 分钟。 一旦某容器执行了 10 分钟并且没有出现问题，kubelet 对该容器的重启回退计时器执行 重置操作

参考 https://kubernetes.io/zh/docs/concepts/workloads/pods/pod-lifecycle/

## pod 调度不均衡

https://www.cnblogs.com/rancherlabs/p/12228019.html

https://cloud.tencent.com/developer/article/1671811

## 静态Pod

静态 Pod 在指定的节点上由 kubelet 守护进程直接管理，不需要 API 服务器 监管。 与由控制面管理的 Pod（例如，Deployment） 不同；kubelet 监视每个静态 Pod（在它崩溃之后重新启动）。

静态 Pod 永远都会绑定到一个指定节点上的 Kubelet。

kubelet 会尝试通过 Kubernetes API 服务器为每个静态 Pod 自动创建一个 镜像 Pod。 这意味着节点上运行的静态 Pod 对 API 服务来说是不可见的，但是不能通过 API 服务器来控制

## 删除被驱逐节点

kubectl -n eos-bpm get pods | grep Evicted |awk '{print$1}'|xargs kubectl -n eos-bpm delete pods

## Kubernetes Resource QoS Classes

QoS 是容器资源配置的一个分类，驱逐策略将根据不同的分类来决定先驱逐谁

Guaranteed：每个容器都必须设置CPU和内存的限制和请求（最大和最小）
Burstable：在不满足Guaranteed的情况下，至少设置一个CPU或者内存的请求
BestEffort：什么都不设置，可用使用到系统的最大资源

最小被驱逐的就是BestEffort类型

## 关于 request 和 limit

必须要满足这个条件

0 <= request <= limit

limit 0 就是资源没有限制，理解request是容器启动的最小资源，为0就是没有资源也可用启动，当然没有资源是起不来的
request只是保证被调度的时候，工作节点需要有这么多的资源。而limit会限制能使用的最大资源

## 查看所有pod

kubectl get pods --all-namespaces

## ResourceQuota(配额)

kubectl get resourcequota -n namespace
kubectl describe resourcequota -n namespace

kubectl get ResourceQuota -A

kubectl get resourcequota namespace --namespace=namespace --output=yaml

当一个集群有分配ResourceQuota和对应的Namespace时，部署的pod需要声明request和limit，否正pod启动失败
开启了resource quota时，用户创建pod，必须指定cpu、内存的 requests or limits ，否则创建失败。resourceQuota搭配 limitRanges口感更佳：limitRange可以配置创建Pod的默认limit/reques

在一个多用户、多团队的k8s集群上，通常会遇到一个问题，如何在不同团队之间取得资源的公平，即，不会因为某个流氓团队占据了所有资源，从而导致其他团队无法使用k8s。
k8s的解决方法是，通过RBAC将不同团队（or 项目）限制在不同的namespace下，通过resourceQuota来限制该namespace能够使用的资源

配置参考，通过命名空间去绑定，如果要制空配额设置，可用删除资源对象，或者把spec部分注释掉

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: quota-test
  namespace: test
spec:
  hard:
    requests.cpu: "2"
    requests.memory: 2Gi
    limits.cpu: "4"
    limits.memory: 4Gi
    requests.nvidia.com/gpu: 4
    pods: "3"
    services: "6"
```

参考

https://blog.csdn.net/sinron_wu/article/details/106518824

## 设置namespace上下文

kubectl config set-context $(kubectl config current-context) --namespace=sy

## k8s中的各种ip

1. NODE IP

也称为INTERNAL-IP，通过 `kubectl get node -o wide` 查看到的就是INTERNAL-IP

2. POD IP

pod网络的IP地址，是每个POD分配的虚拟IP，可以使用 `kubectl get pod -o wide` 来查看

3. CLUSTER-IP

它是Service的地址,是一个虚拟地址（无法ping），是使用kubectl create时，--port 所指定的端口绑定的IP,各Service中的pod都可以使用CLUSTER-IP:port的方式相互访问（当然更应该使用ServiceName:port的方式）可以使用`kubectl get svc -o wide`进行查看

## 操作工作负载的方式

rolling-update 版本操作

patch方式，patch不会去重建Pod，Pod的IP可以保持。kubectl patch -f node.json -p '{"spec":{"unschedulable":true}}'

create 会从完整的yaml文件操作，先删除资源对象再次创建

apply 只会更新yaml文件中声明的部分，所以apply操作的yaml文件可用不完整，但是create要求完整

## kubectl proxy 让外部网络访问K8S

基本使用
kubectl proxy --port=8009 

开放地址
kubectl proxy --address=0.0.0.0  --port=8009

授权其它主机(相当于其它服务都能随意服务，可用限制accept的范围)
kubectl proxy --address='0.0.0.0'  --accept-hosts='^*$' --port=8009
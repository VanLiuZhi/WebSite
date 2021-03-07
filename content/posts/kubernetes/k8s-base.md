---
weight: 1101
title: "Kubernetes 笔记"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Kubernetes 笔记"
# featuredImagePreview: "images/Kubernetes/k8s.jpg"
# featuredImage: "/images/Kubernetes/k8s.jpg"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Cloud-Native, K8s]
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

## -w 参数

可以通过加 -w 开启资源监控，比如 `kubectl get pod -w -oyaml` 当pod被修改后，可以观察到yaml文件的变化

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
kubectl taint node node1 key1-  删除指定key所有的effect
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

资源配额的支持在很多 Kubernetes 版本中是默认开启的。当 API 服务器的 --enable-admission-plugins= 参数中包含 ResourceQuota 时，资源配额会被启用

kubectl get resourcequota -n namespace
kubectl describe resourcequota -n namespace

kubectl get ResourceQuota -A

kubectl get resourcequota namespace --namespace=namespace --output=yaml

当一个集群有分配ResourceQuota和对应的Namespace时，部署的pod需要声明request和limit，否则pod启动失败
开启了resource quota时，用户创建pod，必须指定cpu、内存的 requests or limits ，否则创建失败。resourceQuota搭配 limitRanges使用：limitRange可以配置创建Pod的默认limit/reques

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

可以通过describe查看used，也就是当前配额的占用(只有pod声明了resources，并且不是无限制的，才会被配额控制和统计到，也就是pod resources 不要设置0，这种做法不合规范)


场景举例

1. 查看ResourceQuota的describe，used字段只会统计配置了资源限制的pod，也就是pod不声明resources，或者设置无限制这里used不会统计到
一般会认为used会把当前命名空间的资源占用统计到，但是从设计的角度来说，直接拉取pod的resources是最容易的，具体使用到多少资源属于监控的范畴，减少了API的压力

2. 已配置ResourceQuota，创建新的pod资源占用超过配额。此时pod无法创建，replicaset状态不满足。删除配额声明或修改上限，等会pod会继续创建

3. pod已存在，没有配额声明，但是资源声明2Gi，此时创建配额声明限制是1Gi，配额可以创建，已存在的pod也不会受到影响。配额的used字段显示是2Gi

4. 配额限制声明0，pod resources 声明是0，可以创建pod，假如pod声明是1Gi，不能创建(这种情况是比较坑的，最好避免)

5. 配额的声明只管pod设置了resources，如果pod的resources是无限制，也就是都是0，那么不受配额的影响(也是要避免出现这种情况)

6. 总之在使用了配额后，pod就必须声明资源限制，然后不要设置为0，这样才能更好的发挥ResourceQuota的作用，可以配合limitRange来对pod添加默认的resources

参考

https://blog.csdn.net/sinron_wu/article/details/106518824

## CPU和内存单位

在资源配置的时候用到

CPU:  2,  200m
memory:  100Mi  2Gi

cpu 1000m  =  1 不带单位

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

create 会从完整的yaml文件操作，先删除资源对象再次创建（测试并不会删除，如果对一个已存在的资源对象操作，会报错该资源已存在）

apply 只会更新yaml文件中声明的部分，所以apply操作的yaml文件可用不完整，但是create要求完整

## kubectl proxy 让外部网络访问K8S

基本使用
kubectl proxy --port=8009 

开放地址
kubectl proxy --address=0.0.0.0  --port=8009

授权其它主机(相当于其它服务都能随意服务，可以限制accept的范围)
kubectl proxy --address='0.0.0.0'  --accept-hosts='^*$' --port=8009

## kubelet 配置

https://v1-18.docs.kubernetes.io/zh/docs/tasks/administer-cluster/kubelet-config-file/

## Headless Service

通过service访问pod

1. 以Service的VIP(Virtual IP,即:虚拟IP)方式.比如:当我访问192.168.0.1这个Service的IP地址时,它就是一个VIP.在实际中,它会把请求转发到Service代理的具体Pod上.

2. 以Service的DNS方式.在这里又分为两种处理方法:

第一种是Normal Service.这种情况下,当访问DNS记录时,解析到的是Service的VIP.

第二种是Headless Service.这种情况下,访问DNS记录时,解析到的就是某一个Pod的IP地址.

可以看到,Headless Service不需要分配一个VIP,而是可以直接以DNS记录的方式解析出被代理Pod的IP地址.这样设计有什么好处呢?
这样设计可以使Kubernetes项目为Pod分配唯一"可解析身份".而有了这个身份之后,只要知道了一个Pod的名字以及它对应的Service的名字,就可以非常确定地通过这条DNS记录访问到Pod的IP地址.

## PVC访问模式，状态，回收策略

PVC是有命名空间的，而PV是集群级别的资源对象

```
访问模式包括：
 　▷ ReadWriteOnce —— 该volume只能被单个节点以读写的方式映射
 　▷ ReadOnlyMany —— 该volume可以被多个节点以只读方式映射
   ▷ ReadWriteMany —— 该volume只能被多个节点以读写的方式映射    
 状态
 　▷ Available：空闲的资源，未绑定给PVC 
 　▷ Bound：绑定给了某个PVC 
 　▷ Released：PVC已经删除了，但是PV还没有被集群回收 
 　▷ Failed：PV在自动回收中失败了 
 当前的回收策略有：
 　▷ Retain：手动回收
 　▷ Recycle：需要擦出后才能再使用
 　▷ Delete：相关联的存储资产，如AWS EBS，GCE PD，Azure Disk，or OpenStack Cinder卷都会被删除
 　　当前，只有NFS和HostPath支持回收利用，AWS EBS，GCE PD，Azure Disk，or OpenStack Cinder卷支持删除操作。
```

## PV与PVC绑定

用户创建包含容量、访问模式等信息的PVC，向系统请求存储资源。系统查找已存在PV或者监控新创建PV，如果与PVC匹配则将两者绑定。如果PVC创建动态PV，则系统将一直将两者绑定
PV与PVC的绑定是`一一对应关系`，不能重复绑定与被绑定。如果系统一直没有为PVC找到匹配PV，则PVC无限期维持在"unbound"状态，直到系统找到匹配PV。实际绑定的PV容量可能大于PVC中申请的容量

`PV 的生命周期独立于 Pod`
`Pod 消耗的是节点资源，PVC 消耗的是 PV 资源`

## 存储对象使用中保护

如果启用了存储对象使用中保护特性，则不允许将正在被pod使用的活动PVC或者绑定到PVC的PV从系统中移除，防止数据丢失。活动PVC指使用PVC的pod处于pending状态并且已经被指派节点或者处于running状态。

如果已经启用存储对象使用中保护特性，且用户删除正在被pod使用的活动PVC，不会立即删除PVC，而是延后到其状态变成非活动。同样如果用户删除已经绑定的PV，则删除动作延后到PV解除绑定后。

当PVC处于保护中时，其状态为"Terminating"并且其"Finalizers"包含"kubernetes.io/pvc-protection"

## PVC回收策略

pvc不会和pod联动删除，也就是说pvc要手动删除。pv才是集群的资源，通过策略决定当pvc删除的时候，pv做何处理
Retain的话，则pv存在，可以下次再由新的pvc绑定上去。Delete就是删除pvc的时候会把pv也删除了，这样数据就不存在了

如果是动态创建的pv，比如使用storageClass，pv的回收策略由storageClass决定

## 基于环境变量的服务发现

通常，创建service后，可以通过dns做服务发现。还可以基于环境变量的服务发现

```s
REDIS_MASTER_SERVICE_HOST=10.0.0.11
REDIS_MASTER_SERVICE_PORT=6379
REDIS_MASTER_PORT=tcp://10.0.0.11:6379
REDIS_MASTER_PORT_6379_TCP=tcp://10.0.0.11:6379
REDIS_MASTER_PORT_6379_TCP_PROTO=tcp
REDIS_MASTER_PORT_6379_TCP_PORT=6379
REDIS_MASTER_PORT_6379_TCP_ADDR=10.0.0.11
```

环境变量的服务发现有个前提，必须先创建service后创建pod，才能在pod中获取到注入的环境变量
假如顺序反了，但是之后pod被删除了，此时会创建新的pod，然后这个时候service已经存在了，就可以注入环境变量

## 扩缩容scale

deployment用rc statefulSet 用sts

kubectl -n monitoring  scale rc prometheus-k8s --replicas=1
kubectl -n monitoring  scale sts prometheus-k8s --replicas=1

## StorageClass

StorageClass 是集群的概念，并且创建后不能随意修改一些配置，比如provisioner

## terminationGracePeriodSeconds

K8S 会给老POD发送SIGTERM信号，并且等待 terminationGracePeriodSeconds 这么长的时间(默认为30秒)，然后终止pod
terminationGracePeriodSeconds是留给程序的缓冲时间

## 启动命令和参数

command: ["/bin/sh", "-c", "my.py xxx"]

建议使用这种方式，前0，1个元素固定不变，把执行命令和参数写到第3个元素位置

复杂一点的情况，可以使用args，命令的全部都算成一个元素，注意 `-` 只用了一个，里面的内容都会作为一个脚本的内容被执行

```yaml
command: ["/bin/sh"]
args:
  - "-c"
  - redis-trib.py replicate
    --password qwe123456
    --master-addr `dig +short redis-cluster5-redis-0.redis-cluster5-headless-service.redis.svc.cluster.local`:6379
    --slave-addr `dig +short redis-cluster5-redis-3.redis-cluster5-headless-service.redis.svc.cluster.local`:6379
```

```yaml
command: ['sh']
args:
- "-c"
- |
  set -ex
  cp /configmap/* /etc/rabbitmq
  rm -f /var/lib/rabbitmq/.erlang.cookie
  {{- if .Values.forceBoot }}
  if [ -d "${RABBITMQ_MNESIA_DIR}" ]; then
    touch "${RABBITMQ_MNESIA_DIR}/force_load"
  fi
  {{- end }}
```

如果要执行多组命令，用`;`分割

```yaml
command: ["/bin/sh"]
args:
  - "-c"
  - redis-trib.py replicate
    --password qwe123456
    --master-addr `dig +short redis-cluster5-redis-1.redis-cluster5-headless-service.redis.svc.cluster.local`:6379
    --slave-addr `dig +short redis-cluster5-redis-4.redis-cluster5-headless-service.redis.svc.cluster.local`:6379;
    redis-trib.py replicate
    --password qwe123456
    --master-addr `dig +short redis-cluster5-redis-2.redis-cluster5-headless-service.redis.svc.cluster.local`:6379
    --slave-addr `dig +short redis-cluster5-redis-5.redis-cluster5-headless-service.redis.svc.cluster.local`:6379
```

参考：

https://kubernetes.io/zh/docs/tasks/inject-data-application/define-command-argument-container/

## 删除K8s Namespace时卡在Terminating状态

`kubectl get namespace redis -o json > tmp.json`
`kubectl get namespace test-rabbitmq -o json > tmp.json`

修改tmp.json删除其中的spec字段，当前目录执行命令

`curl -H "Authorization: Bearer tokenXXX" -k  -H "Content-Type: application/json" -X PUT --data-binary @tmp.json https://xxx:6443/api/v1/namespaces/redis/finalize`

本次演示的是删除redis的命名空间

## 在deployment中使用pvc，pv通过storageClass创建

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-pvc
  annotations:
    volume.beta.kubernetes.io/storage-class: "course-nfs-storage" # storageClass名称
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Mi
```

```yaml
volumes:
  - name: nfs-pvc
    persistentVolumeClaim:
      claimName: test-pvc # pvc名称
```


## securityContext

我们可用通过securityContext获得较高的权限，执行一些命令

比如在容器启动前，修改somaxconn

```yaml
initContainers:
   - name: sysctl
      image: alpine:3.10
      securityContext:
          privileged: true
       command: ['sh', '-c', "echo 511 > /proc/sys/net/core/somaxconn"]
```

## Configmap 动态更新的实现

Configmap或Secret使用有两种方式，一种是env系统变量赋值，一种是volume挂载赋值，env写入系统的configmap是不会热更新的，而volume写入的方式支持热更新！

对于env环境的，必须要滚动更新pod才能生效，也就是删除老的pod，重新使用镜像拉起新pod加载环境变量才能生效。
对于volume的方式，虽然内容变了，但是需要我们的应用直接监控configmap的变动，或者一直去更新环境变量才能在这种情况下达到热更新的目的。

应用不支持热更新，可以在业务容器中启动一个sidercar容器，监控configmap的变动，更新配置文件，或者也滚动更新pod达到更新配置的效果。

可用使用Reloader来实现pod的滚动更新，生效configMap 参考: https://juejin.cn/post/6897882769624727559

## 获取API版本

kubectl api-versions

## K8S Pod 保护之 PodDisruptionBudget

这是k8s保证服务SLA的一种策略，当自愿中断（或者叫主动逃离，即显示的命令操作等，比如删除控制器，删除Pod。与之相对的非自愿中断就是比如OOM一类的）发生的过程中，PDB会保证Pod至少有一定的数量可用

比如10个Pod，PDB是2，那么因为滚动升级，k8s会保证至少有2个pod能够使用

```
1、minAvailable设置成了数值5：应用POD集群中最少要有5个健康可用的POD，那么就可以进行操作。

2、minAvailable设置成了百分数30%：应用POD集群中最少要有30%的健康可用POD，那么就可以进行操作。

3、maxUnavailable设置成了数值5：应用POD集群中最多只能有5个不可用POD，才能进行操作。

4、maxUnavailable设置成了百分数30%：应用POD集群中最多只能有30%个不可用POD，才能进行操作。
```

参考： https://www.jianshu.com/p/7da065228a04

## /etc/resolv.conf

/etc/resolv.conf它是DNS客户机配置文件，用于设置DNS服务器的IP地址及DNS域名，还包含了主机的域名搜索顺序。
该文件是由域名解析 器（resolver，一个根据主机名解析IP地址的库）使用的配置文件。
它的格式很简单，每行以一个关键字开头，后接一个或多个由空格隔开的参数。

resolv.conf  的关键字主要有四个，分别是：

```s
nameserver    //定义DNS服务器的IP地址
domain        //定义本地域名
search        //定义域名的搜索列表
sortlist      //对返回的域名进行排序
```

一般用的最多的是nameserver

1. nameserver　
表明DNS服务器的IP地址。可以有很多行的nameserver，每一个带一个IP地址。在查询时就按nameserver在本文件中的顺序进行，且只有当第一个nameserver没有反应时才查询下面的nameserver。

2. domain　　　
声明主机的域名。很多程序用到它，如邮件系统；当为没有域名的主机进行DNS查询时，也要用到。如果没有域名，主机名将被使用，删除所有在第一个点( .)前面的内容。

3. search　　　
它的多个参数指明域名查询顺序。当要查询没有域名的主机，主机将在由search声明的域中分别查找。domain和search不能共存；如果同时存在，后面出现的将会被使用。

4. sortlist　　
允许将得到域名结果进行特定的排序。它的参数为网络/掩码对，允许任意的排列顺序。

`k8s的运用`：k8s默认会给 /etc/resolv.conf 文件新增 4 个搜索域，查看pod的配置，可以找到以下内容

```
nameserver 10.96.0.10
search eos-mid-public.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```

Node节点警告事件，如果宿主机配置了搜索域，会收到以下警告事件。宿主机的搜索域被注到容器中了。解决方法是删除这个search配置

```
Warning  CheckLimitsForResolvConf  6m20s (x70 over 20h)  kubelet, k8s-worker07  Resolv.conf file '/etc/resolv.conf' contains search line consisting of more than 3 domains!
```

## java k8s API Unknown apiVersionKind

在本地开发是正常的，打成jar包后就会出现这个问题，主要是类没被装载到。ClassPath.getTopLevelClasses() returns empty list

参考issues: https://github.com/kubernetes-client/java/issues/365

解决方法

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <!-- Enable io.kubernetes:client-java to find its model classes. -->
        <requiresUnpack>
            <dependency>
                <groupId>io.kubernetes</groupId>
                <artifactId>client-java-api</artifactId>
            </dependency>
        </requiresUnpack>
    </configuration>
</plugin>
```

## hostNetwork

主机网络模式，可以让pod把端口直接绑定到宿主机

通常配合 `dnsPolicy: ClusterFirstWithHostNet` 一起使用，不然dns解析可能会有问题，比如服务发现

```yaml
hostNetwork: true
dnsPolicy: ClusterFirstWithHostNet
```

## kubeadm 重新生成token

1. 先获取token

- 列出token，可能会有找不到的情况
kubeadm token list | awk -F" " '{print $1}' |tail -n 1

- 如果过期可先执行此命令
kubeadm token create #重新生成token

2. 获取CA公钥的哈希值

openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^ .* //'

3. 从节点加入集群

kubeadm join 192.168.40.xx:6443 --token token填这里 --discovery-token-ca-cert-hash sha256:哈希值填这里

4. 节点加入集群的其它方式

kubeadm token create --print-join-command  直接得到join命令的全部内容，适用于node节点加入

kubeadm join 10.90.16.xx:6443 --token #token --discovery-token-ca-cert-hash #sha256

集群加入新的master，需要先获取证书，执行下面命令

旧版本： kubeadm init phase upload-certs --experimental-upload-certs
新版本： kubeadm init phase upload-certs --upload-certs

得到一个certificate-key，执行下面的命令

旧版本： kubeadm join 10.90.16.xx:6443 --token #token --discovery-token-ca-cert-hash #sha256 --experimental-control-plane --certificate-key #certificate-key
新版本： kubeadm join 10.90.16.xx:6443 --token #token --discovery-token-ca-cert-hash #sha256 --control-plane --certificate-key #certificate-key

unknown flag: --experimental-upload-certs，新版本不再支持此参数了，变更为：--upload-certs

## Pod状态

Pending         API Server已经创建该Pod，但在Pod内还有一个或多个容器的镜像没有创建，包括正在下载镜像的过程
Runnung         Pod内所有容器均已创建，且至少有一个容器处于运行状态、正在启动状态或正在重启状态
Succeeded       Pod内所有容器均成功执行后退出，且不会再重启
Failed          Pod内所有容器均已退出，但至少有一个容器退出为失败状态
Unknown         由于某种原因无法获取该Pod的状态，可能由于网络通信不畅导致

## Pod重启策略

Pod的重启策略（RestartPolicy）应用与Pod内所有容器，并且仅在Pod所处的Node上由kubelet进行判断和重启操作
当某个容器异常退出或者健康检查失败时，kubelet将根据RestartPolicy的设置来进行相应的操作

Pod的重启策略包括：Always、OnFailure和Never，默认值为Always

Always：当容器失效时，由kubelet自动重启该容器
OnFailure：当容器终止运行且退出码不为0时，由kubelet自动重启该容器
Never：不论容器运行状态如何，kubelet都不会重启该容器

## readiness和liveness

1. readiness和liveness的核心区别

实际上readiness 和liveness 就如同字面意思
readiness 就是意思是否可以访问，liveness就是是否存活
如果一个readiness 为fail 的后果是把这个pod 的所有service 的endpoint里面的改pod ip 删掉，意思就这个pod对应的所有service都不会把请求转到这pod来了
但是如果liveness 检查结果是fail就会直接kill container，当然如果你的restart policy 是always 会重启pod

2. 什么样才叫readiness／liveness检测失败呢

实际上k8s提供了3中检测手段

- http get 返回200-400算成功，别的算失败
- tcp socket 你指定的tcp端口打开，比如能telnet 上
- cmd exec 在容器中执行一个命令 推出返回0 算成功

每中方式都可以定义在readiness 或者liveness 中
比如定义readiness 中http get 就是意思说如果我定义的这个path的http get 请求返回200-400以外的http code 就把我从所有有我的服务里面删了吧，如果定义在liveness里面就是把我kill 了

3. 什么时候用readiness 什么时候用readiness

比如如果一个http 服务你想一旦它访问有问题我就想重启容器。那你就定义个liveness 检测手段是http get。反之如果有问题我不想让它重启，只是想把它除名不要让请求到它这里来。就配置readiness

`注意，liveness不会重启pod，pod是否会重启由你的restart policy 控制`

## exec 多容器 情况

使用 -c 指定 container

`kubectl -n kong2 exec -it ingress-kong2-55bb4845b4-zzq46 -c proxy env | grep HOSTNAME`

## 内存

kubernetes上报的
container_memory_working_set_bytes

container_memory_working_set_bytes是取自cgroup memory.usage_in_bytes 与memory.stat total_inactive_file(静态不活跃的内存)

memory.usage_in_bytes的统计数据是包含了所有的file cache的
total_active_file和total_inactive_file都属于file cache的一部分，并且这两个数据并不是业务真正占用的内存，只是系统为了提高业务的访问IO的效率，将读写过的文件缓存在内存中，file cache并不会随着进程退出而释放，只会当容器销毁或者系统内存不足时才会由系统自动回收

## patch 更新资源对象

path是个很有用的操作，有些修改apply并不会重启容器，这导致修改失效，通常这种情况是我们操作不符合规范，应该使用path去更新资源对象

`kubectl patch service istio-ingressgateway -n istio-system -p '{"spec":{"type":"NodePort"}}'`
`kubectl -n istio-system patch deployment prometheus -p '{"spec":{"template":{"spec":{"nodeSelector":{"nodeType": "mid"}}}}}`

## 部署ELB

私有云一般很少用到ELB，但是我们也可以去为集群做一个ELB，硬件负载均衡或者软件负载均衡都可以

用nginx来实现，比如在部署istio的时候，LoadBalancer 处于pending状态的。`15021:31929/TCP,80:30186/TCP,443:32085/TCP,31400:32543/TCP,15443:30676/TCP` 需要暴露这几个端口

找一台服务器部署nginx

```json
stream {
    server {
        listen 80;
        proxy_pass 192.168.120.110:30186;
    }
    server {
        listen 443;
        proxy_pass 192.168.120.110:32085;
    }
    server {
        listen 31400;
        proxy_pass 192.168.120.110:32543;
    }
    server {
        listen 15021;
        proxy_pass 192.168.120.110:31929;
    }
    server {
        listen 15443;
        proxy_pass 192.168.120.110:30676;
    }
}
```

比如15021:31929/TCP，在nginx中，监听当前的15021，并将请求发到192.168.120.110:31929，这个是k8s的nodeport地址

## 域名解析验证

1. 启动一个busybox容器

kubectl run -i --tty --image busybox dns-test --restart=Never --rm /bin/sh

2. 执行nslookup命令

nslookup redis-app-1.redis-service.default.svc.cluster.local

可能会返回这个信息，也是代表正常的 Can't find redis-app-1.redis-service.default.svc.cluster.local: No answer

## 删除节点

删除这个node节点

kubectl delete nodes node06

然后在node06这个节点上执行如下命令：

kubeadm reset 重置配置，根据输出提示，删除一些必要的文件即可



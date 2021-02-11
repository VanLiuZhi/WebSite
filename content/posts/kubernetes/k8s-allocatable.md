---
weight: 1
title: "K8s Allocatable"
subtitle: "K8s Allocatable"
date: 2021-01-04T13:56:16+08:00
lastmod: 2021-01-04T13:56:16+08:00
draft: true
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "K8s Allocatable"
# featuredImagePreview: "images/base-image.jpg"
# featuredImage: "/images/base-image.jpg"
# resources:
# - name: "featured-image"
#   src: "images/base-image.jpg"

tags: 
categories: 

lightgallery: true

toc:
  # 自动展开目录
  auto: false
---

k8s节点资源预留

<!--more-->

## 基础概念

k8s节点上的资源，是可以全部分配给Pod使用的，如果我们不做限制，假设某一个业务Pod进程CPU占用过高，会影响系统服务和进程的正常使用
所以应该给系统进程预留一部分资源保证基础服务的正常

## 节点资源类型

Node Capacity：Node的所有硬件资源
kube-reserved：给kube组件预留的资源：kubelet,kube-proxy以及docker等
system-reserved：给system进程预留的资源
eviction-threshold：kubelet eviction的阈值设定(驱逐相关)
Allocatable：真正scheduler调度Pod时的参考值（保证Node上所有Pods的request resource不超过Allocatable）

计算公式：节点上可配置值 = 总量 - 预留值 - 驱逐阈值
`Allocatable = Capacity - Reserved(kube+system) - Eviction Threshold`

## 节点预留配置

可以通过修改启动参数，或者修改配置文件的方式来实现对资源预留的配置

特别注意，资源隔离是通过Linux内核的Cgroup来实现的，需要在对应目录下创建文件，一般这个目录我们是没有的，也就是需要我们手动创建(下文会讲解如何创建这个文件)

### 修改配置文件

`/var/lib/kubelet/config.yaml`

打开配置文件，增加kube-reserved的配置

```yaml
apiVersion: kubelet.config.k8s.io/v1beta1
......
enforceNodeAllocatable:
- pods
- kube-reserved  # 开启 kube 资源预留
kubeReserved:
  cpu: 500m
  memory: 2Gi
  ephemeral-storage: 1Gi
kubeReservedCgroup: /kubelet.slice  # 指定 kube 资源预留的 cgroup
```

保存修改后，`systemctl restart kubelet` 重启服务，但是服务肯定是启动失败的
通过 `journalctl -u kubelet -f` 查看日志

```s
Failed to start ContainerManager Failed to enforce Kube Reserved Cgroup Limits on "/kubelet.slice": ["kubelet"] cgroup does not exist
```

这个问题就是我们上面提到的，Cgroup通过文件记录资源隔离，所以我们需要手动创建对应文件

修改一下启动参数，提高一下日志级别，这样就能输出缺失的目录

```s
vim /var/lib/kubelet/kubeadm-flags.env
KUBELET_KUBEADM_ARGS="--v=4 --cgroup-driver=systemd --network-plugin=cni"
```

```s
$ systemctl daemon-reload
$ systemctl restart kubelet
```

通过 `journalctl -u kubelet -f` 查看日志，可以找到缺失的文件目录了，创建这些目录，再次重启服务，发现还需要hugetlb，也创建它

全部需要创建的文件如下

```s
mkdir -p /sys/fs/cgroup/cpu,cpuacct/kubelet.slice.slice
mkdir -p /sys/fs/cgroup/systemd/kubelet.slice.slice
mkdir -p /sys/fs/cgroup/cpu,cpuacct/kubelet.slice.slice
mkdir -p /sys/fs/cgroup/cpuset/kubelet.slice.slice
mkdir -p /sys/fs/cgroup/pids/kubelet.slice.slice
mkdir -p /sys/fs/cgroup/memory/kubelet.slice.slice

mkdir -p /sys/fs/cgroup/hugetlb/kubelet.slice.slice
```

systemd 的 cgroup 驱动对应的 cgroup 名称是以 .slice 结尾的
在配置文件中，`kubeReservedCgroup: /kubelet.slice` 可以指定kube 资源预留的 cgroup，那么对应的创建的 cgroup 名称应该为 kubelet.slice.slice

再次重启服务，发现服务正常运行了，查看对应文件，已经写入值了

```s
[root@k8s-worker01 kubelet.slice.slice]# cat /sys/fs/cgroup/memory/kubelet.slice.slice/memory.limit_in_bytes
2147483648
[root@k8s-worker01 kubelet.slice.slice]#
```

kubectl describe node，可以看到对应的Allocatable已经改变

```yaml
Capacity:
 cpu:                4
 ephemeral-storage:  51175Mi
 hugepages-2Mi:      0
 memory:             16242632Ki
 pods:               110
Allocatable:
 cpu:                3500m
 ephemeral-storage:  46147305393
 hugepages-2Mi:      0
 memory:             14043080Ki
 pods:               110
```

### 修改启动参数

`/var/lib/kubelet/kubeadm-flags.env` 启动参数可以在这里配置，然后通过命令重启服务即可

```s
systemctl daemon-reload
systemctl restart kubelet
```

通过systemctl status kubelet -l，可以查看状态的完整信息，`--v=4 --cgroup-driver=systemd --network-plugin=cni --pod-infra-container-image=hub.eos.h3c.com/kubernetes/pause:3.1` 就是我们通过kubeadm-flags.env注入的启动参数

```s
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: active (running) since Mon 2021-01-04 13:51:49 CST; 1h 19min ago
     Docs: https://kubernetes.io/docs/
 Main PID: 149574 (kubelet)
    Tasks: 20
   Memory: 67.7M
   CGroup: /system.slice/kubelet.service
           └─149574 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --v=4 --cgroup-driver=systemd --network-plugin=cni --pod-infra-container-image=hub.eos.h3c.com/kubernetes/pause:3.1
```

配置示例：

```s
--enforce-node-allocatable=pods,kube-reserved,system-reserved
--kube-reserved-cgroup=/system.slice/kubelet.service
--system-reserved-cgroup=/system.slice
--kube-reserved=cpu=200m,memory=250Mi
--system-reserved=cpu=200m,memory=250Mi
--eviction-hard=memory.available<5%,nodefs.available<10%,imagefs.available<10%
--eviction-soft=memory.available<10%,nodefs.available<15%,imagefs.available<15%
--eviction-soft-grace-period=memory.available=2m,nodefs.available=2m,imagefs.available=2m
--eviction-max-pod-grace-period=30
--eviction-minimum-reclaim=memory.available=0Mi,nodefs.available=500Mi,imagefs.available=500Mi
```

各个参数含义

1. --enforce-node-allocatable

```s
含义：指定kubelet为哪些进程做硬限制，可选的值有：

* pods
* kube-reserved
* system-reserve

这个参数开启并指定pods后kubelet会为所有pod的总cgroup做资源限制(通过cgroup中的kubepods.limit_in_bytes)，限制为公式计算出的allocatable的大小

假如想为系统进程和k8s进程也做cgroup级别的硬限制，还可以在限制列表中再加system-reserved和kube-reserved，同时还要分别加上--kube-reserved-cgroup和--system-reserved-cgroup以指定分别限制在哪个cgroup里(也就是cgroup文件的写入路径，我们要手动创建)
```

2. 设置cgroup的路径

--kube-reserved-cgroup=/system.slice/kubelet.service
--system-reserved-cgroup=/system.slice

```s
含义：这个参数用来指定系统守护进程所使用的cgroup

注意，这里指定的cgroup及其子系统需要预先创建好，kubelet并不会为你自动创建好
```

3. 配置 k8s组件预留资源的大小，CPU、Mem

```s
指定为k8s系统组件（kubelet、kube-proxy、dockerd等）预留的资源量，

如：--kube-reserved=cpu=1,memory=2Gi,ephemeral-storage=1Gi。

这里的kube-reserved只为非pod形式启动的kube组件预留资源，假如组件要是以static pod（kubeadm）形式启动的，那并不在这个kube-reserved管理并限制的cgroup中，而是在kubepod这个cgroup中。

（ephemeral storage需要kubelet开启feature-gates，预留的是临时存储空间（log，EmptyDir），生产环境建议先不使用）

ephemeral-storage是kubernetes1.8开始引入的一个资源限制的对象，kubernetes 1.10版本中kubelet默认已经打开的了,到目前1.11还是beta阶段，主要是用于对本地临时存储使用空间大小的限制，如对pod的empty dir、/var/lib/kubelet、日志、容器可读写层的使用大小的限制
```

4. 配置 系统守护进程预留资源的大小

```s
含义：为系统守护进程(sshd, udev等)预留的资源量，

如：--system-reserved=cpu=500m,memory=1Gi,ephemeral-storage=1Gi。

注意，除了考虑为系统进程预留的量之外，还应该为kernel和用户登录会话预留一些内存
```

5. 配置 驱逐pod的硬阈值

```s
含义：设置进行pod驱逐的阈值，这个参数只支持内存和磁盘。

通过--eviction-hard标志预留一些内存后，当节点上的可用内存降至保留值以下时，

kubelet 将会对pod进行驱逐。
```

6. 配置 驱逐pod的软阈值

```s
--eviction-soft=memory.available<10%,nodefs.available<15%,imagefs.available<15%
```

7. 定义达到软阈值之后，持续时间超过多久才进行驱逐

```s
--eviction-soft-grace-period=memory.available=2m,nodefs.available=2m,imagefs.available=2m
```

8. 驱逐pod前最大等待时间=min(pod.Spec.TerminationGracePeriodSeconds, eviction-max-pod-grace-period)，单位为秒

```s
--eviction-max-pod-grace-period=30
```

9. 至少回收的资源量

```s
--eviction-minimum-reclaim=memory.available=0Mi,nodefs.available=500Mi,imagefs.available=500Mi
```

## 使用举例

```s
--enforce-node-allocatable=pods,kube-reserved,system-reserved
--kube-reserved-cgroup=/kubelet.service
--system-reserved-cgroup=/system.slice
--kube-reserved=cpu=500m,memory=1Gi,ephemeral-storage=1Gi
--system-reserved=cpu=500m,memory=1Gi,ephemeral-storage=1Gi
--eviction-soft=memory.available<10%,nodefs.available<10%,imagefs.available<10%
--eviction-soft-grace-period=memory.available=2m,nodefs.available=2m,imagefs.available=2m
--eviction-max-pod-grace-period=30
```

`/var/lib/kubelet/kubeadm-flags.env`

KUBELET_KUBEADM_ARGS=--cgroup-driver=systemd --network-plugin=cni --pod-infra-container-image=hub.eos.h3c.com/kubernetes/pause:3.1

`KUBELET_KUBEADM_ARGS="--v=4 --cgroup-driver=systemd --network-plugin=cni --pod-infra-container-image=hub.eos.h3c.com/kubernetes/pause:3.1 --enforce-node-allocatable=pods,kube-reserved,system-reserved --kube-reserved-cgroup=/kubelet.service --system-reserved-cgroup=/system.slice --kube-reserved=cpu=500m,memory=500Mi,ephemeral-storage=1Gi --system-reserved=cpu=500m,memory=500Mi,ephemeral-storage=1Gi --eviction-soft=memory.available<10%,nodefs.available<10%,imagefs.available<10% --eviction-soft-grace-period=memory.available=2m,nodefs.available=2m,imagefs.available=2m --eviction-max-pod-grace-period=30"`


```s
systemctl daemon-reload
systemctl restart kubelet
```

mkdir -p /sys/fs/cgroup/cpu,cpuacct/kubelet.service.slice
mkdir -p /sys/fs/cgroup/systemd/kubelet.service.slice
mkdir -p /sys/fs/cgroup/cpu,cpuacct/kubelet.service.slice
mkdir -p /sys/fs/cgroup/cpuset/kubelet.service.slice
mkdir -p /sys/fs/cgroup/pids/kubelet.service.slice
mkdir -p /sys/fs/cgroup/memory/kubelet.service.slice
mkdir -p /sys/fs/cgroup/hugetlb/kubelet.service.slice

mkdir -p /sys/fs/cgroup/cpu,cpuacct/system.slice.slice
mkdir -p /sys/fs/cgroup/systemd/system.slice.slice
mkdir -p /sys/fs/cgroup/cpu,cpuacct/system.slice.slice
mkdir -p /sys/fs/cgroup/cpuset/system.slice.slice
mkdir -p /sys/fs/cgroup/pids/system.slice.slice
mkdir -p /sys/fs/cgroup/memory/system.slice.slice
mkdir -p /sys/fs/cgroup/hugetlb/system.slice.slice

对应目录创建，重启kubelet

Scheduler会确保Node上所有的Pod Resource Request不超过NodeAllocatable。Pods所使用的memory和storage之和超过NodeAllocatable后就会触发kubelet Evict Pods

## 细节部分

1. 注意，因为kube-reserved设置的cpu其实最终是写到kube-reserved-cgroup下面的cpu shares。cpu shares，只有当集群的cpu跑满需要抢占时才会起作用，因此你会看到Node的cpu usage还是有可能跑到100%的，但是不要紧，kubelet等组件并没有受到影响，如果kubelet此时需要更多的cpu，那么它就能抢到更多的时间片，最多可以抢到kube-reserved设置的cpu nums

2. 资源预留管理是需要计算Pod Resource Request。也就是Pod的资源限制必须要设置，而且不能设置为0

3. 驱逐策略是针对使用值的，所以关注点不是Request。驱逐策略是通过系统监控来做评判的，可能存在误差

4. 软驱逐，硬驱逐的区别：软驱逐给了系统一个宽限期，不会像硬驱逐一样马上触发，由于通过系统监控指标来判断是否驱逐，以及考虑服务临时负载过高情况，软驱逐使用更加灵活(对于CPU这种可压缩资源)

5. 通过启动参数配置的值会覆盖配置文件的设置

6. 重启服务器后，/sys/fs 的配置重置了

## 参考

https://my.oschina.net/jxcdwangtao/blog/1629059

https://cloud.tencent.com/developer/article/1697043

https://www.jianshu.com/p/5dcf04e8d56b

http://docs.kubernetes.org.cn/723.html
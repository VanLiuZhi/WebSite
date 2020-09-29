---
weight: 1
title: "K8s Kubadm"
subtitle: "K8s Kubadm"
date: 2020-09-21T11:13:46+08:00
lastmod: 2020-09-21T11:13:46+08:00
draft: true
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "K8s Kubadm"
# featuredImagePreview: "images/base-image.jpg"
# featuredImage: "/images/base-image.jpg"
# resources:
# - name: "featured-image"
#   src: "images/base-image.jpg"

tags: 
categories: 

lightgallery: true

toc:
  auto: false
---



<!--more-->

## kubeadm init 的工作流程

1. Prefligth Checks 检查

这一步会检测系统是否满足安装要求，比如内核版本，docker是否可用等

2. 生成证书

`/etc/kubernetes/pki` 证书默认在这个目录生成，可用配置`--cert-dir`来修改目录

主要的证书文件是 `ca.crt` 和对应的私钥 `ca.key`

kubernetes 对外提供的服务都需要HTTPS协议，使用 kubectl 命令，需要通过 kube-apiserver 向 kubelet 发起请求，这个连接也必须是安全的
kubeadm 为 这一步生成的是 `apiserver-kubelet-client.out` 文件，对应的私钥是 `apiserver-kubelet-client.key`

还有其它很多证书，自行查看目录

3. 生成配置文件

`/etc/kubernetes/**.conf` 会在这个目录下生成配置文件

记录节点的服务器地址、监听端口。证书目录等信息，这样scheduler等组件直接读取配置和kubelet建立安全连接

4. 为 Master 组件生成 Pod 配置文件

在 Kubernetes 中，有一种特殊的容器启动方法叫做“Static Pod”。它允许你把要部署的 Pod 的 YAML 文件放在一个指定的目录里。这样，当这台机器上的 kubelet 启动时，它会自动检查这个目录，加载所有的 Pod YAML 文件，然后在这台机器上启动它们。

从这一点也可以看出，kubelet 在 Kubernetes 项目中的地位非常高，在设计上它就是一个完全独立的组件，而其他 Master 组件，则更像是辅助性的系统容器。

cd /etc/kubernetes/manifests
├── etcd.yaml
├── kube-apiserver.yaml
├── kube-controller-manager.yaml
└── kube-scheduler.yaml
然后 kubeadm 会调用 kubelet 运行这个 yml 文件，等到 Master 容器启动后，kubeadm 会通过检查localhost:6443/healthz 这个 Master 组件的健康检查 url，等待 Master 组件完全运行起来。

然后，kubeadm 就会为集群生成一个 bootstrap token。在后面，只要持有这个 token，任何一个安装了 kubelet 和 kubadm 的节点，都可以通过 kubeadm join 加入到这个集群当中。

这个 token 的值和使用方法，会在 kubeadm init 结束后被打印出来。

在 token 生成之后，kubeadm 会将 ca.crt 等 Master 节点的重要信息，通过 ConfigMap 的方式保存在 Etcd 当中，供后续部署 Node 节点使用。这个 ConfigMap 的名字是 cluster-info。

kubeadm init 的最后一步，就是安装默认插件。Kubernetes 默认 kube-proxy 和 DNS 这两个插件是必须安装的。它们分别用来提供整个集群的服务发现和 DNS 功能。其实，这两个插件也只是两个容器镜像而已，所以 kubeadm 只要用 Kubernetes 客户端创建两个 Pod 就可以了。

## 参考

https://www.jianshu.com/p/1e65610dd223

https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/high-availability/

https://zhangguanzhang.github.io/2019/11/24/kubeadm-base-use/#%E4%BA%8B%E5%89%8D%E5%87%86%E5%A4%87-%E6%AF%8F%E5%8F%B0%E6%9C%BA%E5%99%A8
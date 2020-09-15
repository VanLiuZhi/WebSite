---
weight: 1
title: "K8s Upgrade Version"
subtitle: "K8s Upgrade Version"
date: 2020-09-11T09:49:00+08:00
lastmod: 2020-09-11T09:49:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "K8s Upgrade Version"
# featuredImagePreview: "images/base-image.jpg"
# featuredImage: "/images/base-image.jpg"
# resources:
# - name: "featured-image"
#   src: "images/base-image.jpg"

tags: [cloud-native, k8s]
categories: [Kubernetes] 

lightgallery: true

toc:
  auto: false
---

kubernetes版本升级

<!--more-->

## 准备工作

kubernetes新版本都会优化很多功能，完善系统稳定性，本文将指导升级k8s到新版本

### 切换yum源

```sh
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

## 镜像准备

|`master`|`Node`|
|:---:|:---:|
|kube-apiserver|
|kube-scheduler|
|kube-controller-manager|
|kube-proxy|

由于国内拉取不到谷歌镜像，但是可以通过阿里云或者Docker Huber拉取镜像，并修改tag就行了

我们从Docker Huber拉取镜像，kubesphere这个owner下会定期同步官方镜像，基本上很全面了

```sh
sudo docker pull kubesphere/kube-proxy:v1.15.1
sudo docker pull kubesphere/kube-scheduler:v1.15.1
sudo docker pull kubesphere/kube-apiserver:v1.15.1
sudo docker pull kubesphere/kube-controller-manager:v1.15.1

# 使用阿里的仓库
sudo docker pull registry.aliyuncs.com/google_containers/kube-proxy:v1.15.1
sudo docker pull registry.aliyuncs.com/google_containers/kube-scheduler:v1.15.1
sudo docker pull registry.aliyuncs.com/google_containers/kube-apiserver:v1.15.1
sudo docker pull registry.aliyuncs.com/google_containers/kube-controller-manager:v1.15.1
```

```sh
sudo docker tag registry.aliyuncs.com/google_containers/kube-proxy:v1.15.1 k8s.gcr.io/kube-proxy:v1.15.1
sudo docker tag registry.aliyuncs.com/google_containers/kube-scheduler:v1.15.1 k8s.gcr.io/kube-scheduler:v1.15.1
sudo docker tag registry.aliyuncs.com/google_containers/kube-apiserver:v1.15.1 k8s.gcr.io/kube-apiserver:v1.15.1
sudo docker tag registry.aliyuncs.com/google_containers/kube-controller-manager:v1.15.1 k8s.gcr.io/kube-controller-manager:v1.15.1
```

```sh
sudo docker save k8s.gcr.io/kube-proxy:v1.15.1 > kube-proxy.tar
sudo docker save k8s.gcr.io/kube-scheduler:v1.15.1 > kube-scheduler.tar
sudo docker save k8s.gcr.io/kube-apiserver:v1.15.1 > kube-apiserver.tar
sudo docker save k8s.gcr.io/kube-controller-manager:v1.15.1 > kube-controller-manager.tar
```

## kubeadm 

检测能升级的版本

`yum list --showduplicates kubeadm --disableexcludes=kubernetes`

升级1.18.8版本

`yum install kubeadm-1.15.1-0 --disableexcludes=kubernetes`

yum install kubeadm-1.16.1-0 --disableexcludes=kubernetes

查看是否升级成功

`kubeadm version`

kubeadm upgrade plan
`kubeadm upgrade apply v1.15.1`
yum install -y kubelet-1.15.1-0 kubectl-1.15.1-0 --disableexcludes=kubernetes
systemctl daemon-reload
systemctl restart kubelet

yum install -y kubelet-1.15.1-0  --disableexcludes=kubernetes
yum install -y kubeadm-1.15.1-0 --disableexcludes=kubernetes
yum install -y kubectl-1.15.1-0 --disableexcludes=kubernetes

kubeadm upgrade node 

unset http_proxy
unset https_proxy

### 升级master节点

kubeadm upgrade apply v1.15.1
yum install -y kubelet-1.15.1-0 kubectl-1.15.1-0 --disableexcludes=kubernetes
systemctl daemon-reload
systemctl daemon-reload

### 升级worker节点

yum install -y kubeadm-1.15.1-0 --disableexcludes=kubernetes
yum install -y kubelet-1.15.1-0  --disableexcludes=kubernetes

## 自建CRD

https://www.servicemesher.com/blog/kubernetes-crd-quick-start/

## 自建CRD

https://www.servicemesher.com/blog/kubernetes-crd-quick-start/

## 注意

主要组件是跟者版本走的，但是插件是有兼容性的，比如coredns，可能1.14到1.15都可以用一个版本，但是kube-proxy不行，kube-proxy必须跟者版本走
如果当前coredns不满足条件了，那么应该根据kubeadm升级计划的提示准备对应的镜像

kube-scheduler 和 kube-controller-manager 可以以集群模式运行，通过 leader 选举产生一个工作进程，其它进程处于阻塞模式。

kube-apiserver 可以运行多个实例，但对其它组件需要提供统一的访问地址，本章节部署 Kubernetes 高可用集群实际就是利用 HAProxy + Keepalived 配置该组件

## 参考

https://www.jianshu.com/p/977a7169bf66

https://www.jianshu.com/p/d42ef0eff63f

https://www.jianshu.com/p/fe847170d050

## 高可用方案

https://blog.csdn.net/horsefoot/article/details/52247277

http://dockone.io/article/8721

百度 kubernetes 高可用集群




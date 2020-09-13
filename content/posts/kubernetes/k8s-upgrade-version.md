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
docker pull kubesphere/kube-proxy:v1.18.8
docker pull kubesphere/kube-scheduler:v1.18.8
docker pull kubesphere/kube-apiserver:v1.18.8
docker pull kubesphere/kube-controller-manager:v1.18.8
```

```sh
docker tag kubesphere/kube-proxy:v1.18.8 k8s.gcr.io/kube-proxy:v1.18.8
docker tag kubesphere/kube-scheduler:v1.18.8 k8s.gcr.io/kube-scheduler:v1.18.8
docker tag kubesphere/kube-apiserver:v1.18.8 k8s.gcr.io/kube-apiserver:v1.18.8
docker tag kubesphere/kube-controller-manager:v1.18.8 k8s.gcr.io/kube-controller-manager:v1.18.8
```

## kubeadm 

检测能升级的版本

`yum list --showduplicates kubeadm --disableexcludes=kubernetes`

升级1.18.8版本

`yum install kubeadm-1.18.8-0 --disableexcludes=kubernetes`

查看是否升级成功

`kubeadm version`

## 自建CRD

https://www.servicemesher.com/blog/kubernetes-crd-quick-start/

## 


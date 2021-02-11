---
weight: 1104
title: "K8s 版本升级实践总结"
subtitle: "K8s Upgrade Version"
date: 2020-09-11T09:49:00+08:00
lastmod: 2020-09-11T09:49:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "K8s Upgrade Version"
# featuredImagePreview: "images/Kubernetes/upgrade.jpg"
# featuredImage: "/images/Kubernetes/upgrade.jpg"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Cloud-Native, K8s]
categories: [Kubernetes]

lightgallery: true

toc:
  auto: false
---

kubernetes新版本都会优化很多功能，完善系统稳定性，本文将指导升级k8s到新版本，升级流程与踩坑实践

<!--more-->

## 准备工作

kubernetes新版本都会优化很多功能，完善系统稳定性，本文将指导升级k8s到新版本

我们采用kubeadm来升级k8s

{{< admonition danger 特别注意 >}}
要特别注意的一点就是，kubeadmin只能一个版本一个版本的升级，就是假如你是1.14.1只能先升级到1.15.1，然后升级1.16.1，不能一次夸很大的版本
{{< /admonition >}}

注意自己机器的配置，遇到过swp没有设置永久生效，导致升级重启后可能docker无法运行镜像的

kubectl -n kube-system get cm kubeadm-config -oyaml  查看配置文件

我们需要解决的就是Linux软件包源，以及docker镜像源，即可完成安装

### 切换yum源

相关软件，比如kubelet，kubeadm都是需要科学上网的，可以换用阿里云的源

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

或者从阿里云拉取第三方仓库

## kubeadm 

检测能升级的版本(--disableexcludes=kubernetes 参数能查看全部版本，不然只列出最新的)

`yum list --showduplicates kubeadm --disableexcludes=kubernetes`

升级1.19.5版本

`yum install kubeadm-1.19.5-0 --disableexcludes=kubernetes`

查看是否升级成功

`kubeadm version`

查看升级计划 `kubeadm upgrade plan` (这个方式只能一个小版本的升级，如果需要跨很大的版本不方便)
会显示能升级的版本，然后升级到这个版本 `kubeadm upgrade apply v1.15.1`

### 升级master节点

yum install -y kubeadm-1.18.0-0 kubelet-1.18.0-0 kubectl-1.18.0-0 --disableexcludes=kubernetes

sudo systemctl daemon-reload
sudo systemctl restart kubelet

可能旧版本安装会失败，可以试试加上--setopt=obsoletes=0参数，或者yum install -y kubelet-1.14.0-0，单独安装kubelet，一般这个的依赖容易出问题

yum install -y kubeadm-1.14.0-0 kubectl-1.14.0-0 --disableexcludes=kubernetes --setopt=obsoletes=0

### 升级worker节点

yum install -y kubelet-1.19.5-0 --disableexcludes=kubernetes


## kubeadm 从私有仓库拉取镜像

我们可以在kubeadm执行命令的时候指定配置文件，配置文件中去指定的仓库拉取镜像，这样就避免了去拉取谷歌镜像

`kubeadm config images list`

## 删除和更新docker

rpm -qa | grep docker – – 列出包含docker字段的软件的信息

yum remove 列出的软件包，docker 包含 docker-ce  docker-cli  cli的版本就是我们通常说的docker版本

`下面我们安装`

yum install -y yum-utils device-mapper-persistent-data lvm2  # 依赖

yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo # 加入源

yum clean all

yum makecache # 重建缓存完成

yum repolist

yum list docker-ce --showduplicates | sort -r # 列出安装包

yum -y install docker-ce-18.09.5-3.el7 docker-ce-cli-19.03.6-3.el7 # 安装，由于k8s目前只支持到19.03，直接安装ce可能会安装最新的cli 20，所以我们指定 cli 的版本，这样linux就不会安装最新的依赖。或者通过 curl -fsSL https://get.docker.com/ | sh 脚本安装

systemctl restart docker
systemctl enable docker

#部分系统安装后未生成daemon.json，请执行以下命令
mkdir -p /etc/docker
touch /etc/docker/daemon.json 
vim /etc/docker/daemon.json

#配置仓库信息如下：
{
    "registry-mirrors": ["https://wnk1ohoc.mirror.aliyuncs.com"],
    "insecure-registries":["hub.eos-ts.h3c.com","hub.eos.h3c.com"],
    "exec-opts": ["native.cgroupdriver=systemd"]
}

#保存后重启docker

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

## 集群备份和恢复

https://cloud.tencent.com/developer/article/1653649?from=10680

https://cloud.tencent.com/developer/article/1638818?from=10680

## 出现问题

没有这个文件

open /run/flannel/subnet.env: no such file or directory

## 问题参考

https://blog.csdn.net/reachyu/article/details/105263983

https://cloud.tencent.com/developer/article/1680083

## 多master

https://blog.csdn.net/double_happy111/article/details/105943912

## calico 网络

https://blog.csdn.net/lswzw/article/details/103044179

https://www.kubernetes.org.cn/1165.html

https://docs.projectcalico.org/getting-started/kubernetes/self-managed-onprem/onpremises

https://www.cnblogs.com/Christine-ting/p/12837250.html

## k8smeetup

http://www.k8smeetup.com/

## 博客

https://blog.gmem.cc/category/work

## kubeadm 

https://kubernetes.io/zh/docs/reference/setup-tools/kubeadm/kubeadm-init/

## 更新证书

https://platformengineer.com/fix-kubernetes-bootstrap-client-certificate-expired-error/

## 内存问题(重要)

https://cloud.tencent.com/developer/article/1637682

## k8s故障处理

https://www.cnblogs.com/rancherlabs/p/12330916.html

高版本可用直接使用 kubeadm alpha certs renew all 更新，具体命令查看文档

kubeadm reset

rm -rf /var/lib/calico
rm -rf /etc/cni/net.d

systemctl restart kubelet

单master部署

kubeadm init --kubernetes-version=1.19.5 --image-repository registry.aliyuncs.com/google_containers --apiserver-advertise-address=10.90.16.112 --pod-network-cidr=192.168.0.0/16 --service-cidr=10.10.0.0/16

多master部署，需要指定控制平面的地址(必须，这样kubeadm会给出加入master的命令)

kubeadm init --kubernetes-version=1.18.0 --apiserver-advertise-address=10.90.16.112 --control-plane-endpoint=10.90.16.112:6443 --image-repository registry.aliyuncs.com/google_containers --service-cidr=10.10.0.0/16 --pod-network-cidr=192.168.0.0/16 --upload-certs


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

yum install kubeadm-1.18.0-0 --disableexcludes=kubernetes

查看是否升级成功

`kubeadm version`

kubeadm upgrade plan
`kubeadm upgrade apply v1.15.1`
yum install -y kubelet-1.18.0-0 kubectl-1.18.0-0 --disableexcludes=kubernetes
yum install -y kubelet-1.16.1-0 kubectl-1.16.1-0 --disableexcludes=kubernetes

sudo systemctl daemon-reload
sudo systemctl restart kubelet

yum install -y kubelet-1.15.1-0 --disableexcludes=kubernetes

yum install -y kubeadm-1.15.1-0 --disableexcludes=kubernetes
yum install -y kubectl-1.15.1-0 --disableexcludes=kubernetes

sudo yum install -y kubeadm-1.16.1-0 kubelet-1.16.1-0 --disableexcludes=kubernetes
sudo yum install -y kubectl-1.16.1-0 --disableexcludes=kubernetes

kubeadm upgrade node 

unset http_proxy
unset https_proxy

### 升级master节点

kubeadm upgrade apply v1.15.1
yum install -y kubelet-1.15.1-0 kubectl-1.15.1-0 --disableexcludes=kubernetes
yum install -y kubelet-1.16.1-0 kubectl-1.16.1-0 --disableexcludes=kubernetes
sudo systemctl daemon-reload
sudo systemctl restart kubelet


### 升级worker节点

yum install -y kubeadm-1.15.1-0 --disableexcludes=kubernetes
yum install -y kubelet-1.15.1-0  --disableexcludes=kubernetes

## kubeadm 从私有仓库拉取镜像

我们可以在kubeadm执行命令的时候指定配置文件，配置文件中去指定的仓库拉取镜像，这样就避免了去拉取谷歌镜像

`kubeadm config images list`

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

高版本可用直接使用 kubeadm alpha certs renew all 更新，具体命令查看文档

```
apiVersion: v1
data:
  ClusterConfiguration: |
    apiServer:
      extraArgs:
        authorization-mode: Node,RBAC
      timeoutForControlPlane: 4m0s
    apiVersion: kubeadm.k8s.io/v1beta2
    certificatesDir: /etc/kubernetes/pki
    clusterName: kubernetes
    controllerManager: {}
    dns:
      type: CoreDNS
    etcd:
      local:
        dataDir: /var/lib/etcd
    imageRepository: registry.aliyuncs.com/google_containers
    kind: ClusterConfiguration
    kubernetesVersion: v1.16.1
    networking:
      dnsDomain: cluster.local
      podSubnet: 10.244.0.0/16
      serviceSubnet: 10.1.0.0/16
    scheduler: {}
  ClusterStatus: |
    apiEndpoints:
      cluster1:
        advertiseAddress: 192.168.59.101
        bindPort: 6443
    apiVersion: kubeadm.k8s.io/v1beta2
    kind: ClusterStatus
kind: ConfigMap
metadata:
  creationTimestamp: "2019-09-23T15:50:48Z"
  name: kubeadm-config
  namespace: kube-system
  resourceVersion: "226113"
  selfLink: /api/v1/namespaces/kube-system/configmaps/kubeadm-config
  uid: e978ab9b-de19-11e9-b73b-525400261060
```

```
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 1.2.3.4
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: cluster1
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: k8s.gcr.io
kind: ClusterConfiguration
kubernetesVersion: v1.16.0
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
scheduler: {}
```
kubeadm reset
kubeadm init --kubernetes-version=1.16.1 --apiserver-advertise-address=10.90.16.112 --pod-network-cidr=3.25.0.0/16

rm -rf /var/lib/calico
rm -rf /etc/cni/net.d
systemctl restart kubelet

kubeadm init --kubernetes-version=1.18.0 --image-repository registry.aliyuncs.com/google_containers --apiserver-advertise-address=10.90.16.112 --pod-network-cidr=192.168.0.0/16 --service-cidr=10.10.0.0/16

kubeadm init --kubernetes-version=1.18.0  
 --apiserver-advertise-address=192.168.56.101   
 --image-repository registry.aliyuncs.com/google_containers  
 --service-cidr=10.10.0.0/16 --pod-network-cidr=192.168.0.0/16


---
weight: 1101
title: "K8s Helm"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "K8s Helm"
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



<!--more-->

## 基础

- helm：
一个命令行下客户端工具，主要用于kubernetes应用chart的创建/打包/发布已经创建和管理和远程Chart仓库。
- Tiller：
helm的服务端，部署于kubernetes内，Tiller接受helm的请求，并根据chart生成kubernetes部署文件（helm称为release），然后提交给 Kubernetes 创建应用。Tiller 还提供了 Release 的升级、删除、回滚等一系列功能。
- Chart： 
helm的软件包，采用tar格式，其中包含运行一个应用所需的所有镜像/依赖/资源定义等，还可能包含kubernetes集群中服务定义
- Release：
在kubernetes中集群中运行的一个Chart实例，在同一个集群上，一个Chart可以安装多次，每次安装均会生成一个新的release。
- Repository：
用于发布和存储Chart的仓库

## 渲染模板

helm template 仓库名/包名 > test.yaml 
helm template ./文件夹名 > test.yaml 

## helm3 安装

下载文件 https://helm-v3.5.2-linux-amd64.tar.gz copy 编译后的二进制文件

cd /opt && wget https://cloudnativeapphub.oss-cn-hangzhou.aliyuncs.com/helm-v3.0.0-alpha.1-linux-amd64.tar.gz

tar -xvf helm-v3.0.0-alpha.1-linux-amd64.tar.gz
mv linux-amd64 helm
chown root.root helm -R

cat > /etc/profile.d/helm.sh << EOF
export PATH=$PATH:/opt/helm
EOF

source /etc/profile.d/helm.sh

初始化替换默认仓库，直接执行init可能无法访问默认仓库导致失败（内网环境，先用nexus代理阿里云的默认参考，关闭服务器代理，执行init即可）

helm3 init --stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts

## istio 模板渲染

helm template ./manifests/charts/base --set global.jwtPolicy=first-party-jwt --set global.hub="hub.eos.h3c.com/istio" > istio-base.yaml
helm template ./manifests/charts/istio-control/istio-discovery --set global.jwtPolicy=first-party-jwt --set global.hub="hub.eos.h3c.com/istio" > istio-control.yaml
helm template ./manifests/charts/gateways/istio-ingress --set global.jwtPolicy=first-party-jwt --set global.hub="hub.eos.h3c.com/istio" > istio-ingress.yaml
helm template ./manifests/charts/gateways/istio-egress --set global.jwtPolicy=first-party-jwt --set global.hub="hub.eos.h3c.com/istio" > istio-egress.yaml

## 安装nfs-provisioner

helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
    --set nfs.server=10.90.16.112 \
    --set nfs.path=/home/k8s

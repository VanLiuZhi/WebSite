---
weight: 1
title: "K8s Kong"
subtitle: "K8s Kong"
date: 2020-12-17T09:48:48+08:00
lastmod: 2020-12-17T09:48:48+08:00
draft: true
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "K8s Kong"
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



<!--more-->

docker run  -di --name=eos_postgre1 -p 5433:5432 --restart=always -e TZ=Asia/Shanghai -e POSTGRES_USER=kong -e POSTGRES_PASSWORD=kong -e POSTGRES_DB=kong -v /data/docker/postgres5433/data:/var/lib/postgresql/data hub.eos.h3c.com/base/postgres:9.5 

## ingress


echo "
apiVersion: v1
kind: Service
metadata:
  name: echo
  namespace: eos-demos
spec:
  selector:
    app: eos-demo13-blue
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 8020
  type: NodePort
" | kubectl apply -f -

curl -svX GET http://myapp.h3c.com/ --resolve myapp.h3c.com:80:http://10.90.15.180:32153

## 问题

map映射不对，本来应该是绑定在 80 上的，http协议就不用写端口，现在是 8000。估计改一下map就行了

kong 1.1 版本，ingress如果不设置 class, 不会被管理后台统计到，但是旧的路由依然能用（能访问，不被kong监听到），然后新加入不指定class也是不行(不能访问，不被kong监听到)

k8s 1.14 ingress 的 address 就算集群中没有ingress-controller，address也不会被清空

部署多个Ingress控制器而不指定注释会导致两个控制器都争相满足Ingress。

不指定注释将导致多个入口控制器声明相同的入口（多个kong-controller相同）。设置与任何现有入口控制器的类都不匹配的值将导致所有入口控制器忽略该入口。（kong-controller 的class 和 ingress不匹配，ingress被忽略）
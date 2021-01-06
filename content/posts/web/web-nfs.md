---
weight: 1
title: "Web Nfs"
subtitle: "Web Nfs"
date: 2020-11-03T09:49:50+08:00
lastmod: 2020-11-03T09:49:50+08:00
draft: true
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Web Nfs"
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

Nfs 网络共享存储在 kubernetes 中的运用

<!--more-->

## 概述

nfs Network File System 文件共享服务

需要一台服务器A挂载一个目录作为共享目录，其它服务器B,C,D可以访问这个目录，实现文件共享

A需要提供nfs服务和rpc服务。BCD准备rpc服务即可

涉及到的端口是2049 111等，nfs 配置文件是 `/etc/exports`


NFS工作原理

```
首先服务器端启动RPC服务，并开启111端口

服务器端启动NFS服务，并向RPC注册端口信息

客户端启动RPC（portmap服务），向服务端的RPC(portmap)服务请求服务端的NFS端口

服务端的RPC(portmap)服务反馈NFS端口信息给客户端。

客户端通过获取的NFS端口来建立和服务端的NFS连接并进行数据的传输
```

## 服务端安装(centos-7) 

`yum -y install nfs-utils`

```
yum install -y rpcbind 
实际上需要安装两个包nfs-utils和rpcbind, 不过当使用yum安装nfs-utils时会把rpcbind一起安装上
centos-6和7服务有差异，需要注意
```

配置共享目录

`vim /etc/exports`   

加入

`/public 10.90.15.0/24(rw,sync,fsid=0)` 或者 `/home/k8s *(rw,root_squash,no_all_squash,sync)` 任何ip都访问

```
1. rw表示可读写，ro只读
2. sync ：同步模式，内存中数据时时写入磁盘；async ：不同步，把内存中数据定期写入磁盘中
3. no_root_squash ：加上这个选项后，root用户就会对共享的目录拥有至高的权限控制，就像是对本机的目录操作一样。不安全，不建议使用；root_squash：和上面的选项对应，root用户对共享目录的权限不高，只有普通用户的权限，即限制了root；all_squash：不管使用NFS的用户是谁，他的身份都会被限定成为一个指定的普通用户身份
4. anonuid/anongid ：要和root_squash 以及all_squash一同使用，用于指定使用NFS的用户限定后的uid和gid，前提是本机的/etc/passwd中存在这个uid和gid
5. fsid=0表示将/opt/nfs整个目录包装成根目录
```

设置开机启动

```s
systemctl enable rpcbind.service
systemctl enable nfs-server.service
```

启动服务

```s
systemctl start rpcbind.service
systemctl start nfs.service
```

showmount -e 加当前服务器ip，检查nfs服务，显示NFS服务器的共享列表。返回一下输出说明正常
showmount -a 查看本机的挂载情况(只能在服务端执行)

```s
[root@k8s-01 ~]# showmount -e 10.90.15.xx
Export list for 10.90.15.xx:
/public 10.90.15.0/24
```

如果修改了共享目录，记得重启nfs服务 systemctl restart nfs.service
最好关闭防火墙 systemctl stop firewalld.service 或者配置防火墙规则

让修改过的配置文件生效 `exportfs –arv`

```
使用exportfs命令，当改变/etc/exports配置文件后，不用重启nfs服务直接用这个exportfs即可，它的常用选项为[-aruv].     
-a ：全部挂载或者卸载；      
-r ：重新挂载；      
-u ：卸载某一个目录；      
-v ：显示共享的目录；
```

## 客户端安装

`yum install -y nfs-utils`

客户端不需要启动nfs服务，只需要启动rpcbind服务，安装nfs-utils的时候会带上rpcbind

`systemctl enable rpcbind.service`
`systemctl start rpcbind.service`

`showmount -e 10.90.15.xx` 看看能不能访问到服务端

创建目录并挂载

`mkdir /opt/nfs` 
`mount -t nfs 10.90.15.xx:/opt/nfs/ /opt/nfs/`

检查挂载情况 `showmount -e` 
或者 `df -h`

卸载挂载：`umount /opt/nfs/`

## 测试

在服务端目录写数据，所有客户端被挂载到网络的目录都能查看到结果

## k8s中使用总结

nfs-client-provisioner 插件必须要跑在拥有rpcbind服务的节点上，否则会导致pod无法创建

statefulset pvc模块创建的pod，也必须跑着拥有rpcbind服务的节点上，这样才能在当前宿主机上创建出nfs挂载目录，实现共享

这个要求对于 ceph 也是类似的

当pod创建后，当前宿主机会创建一个nfs挂载。如果pod消亡，这个挂载也会自动卸载

使用插件后，会自动在服务端共享目录创建文件夹，并且权限已经设置，可以进行读写操作

客户端不需要手动挂载了，由k8s帮我们挂载，目录也不需要去手动创建，保证服务端设置正确，k8s插件连接到正确的服务端配置，工作节点都安装了rpc服务，其它交给k8s

## k8s中部署NFS插件，通过StorageClass动态创建

在整个集群中工作节点准备好rpcbind服务，保证能和nfs服务端通信

创建账号，角色，并绑定

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: default        # 根据实际环境设定namespace,下面类同
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-client-provisioner-runner
rules:
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create", "update", "patch"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    # replace with namespace where provisioner is deployed
    namespace: default
roleRef:
  kind: ClusterRole
  name: nfs-client-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
    # replace with namespace where provisioner is deployed
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    # replace with namespace where provisioner is deployed
    namespace: default
roleRef:
  kind: Role
  name: leader-locking-nfs-client-provisioner
  apiGroup: rbac.authorization.k8s.io
```

部署nfs插件

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nfs-client-provisioner
  labels:
    app: nfs-client-provisioner
  namespace: default  #与RBAC文件中的namespace保持一致
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nfs-client-provisioner
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution: # 硬策略
            nodeSelectorTerms:
              - matchExpressions:
                  - key: kubernetes.io/hostname
                    operator: In
                    # 指定在06节点，即挂载nfs硬盘的节点(不是非必须的)
                    values:
                      - k8s-06
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          image: hub.eos-ts.h3c.com/external_storage/nfs-client-provisioner:latest
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: eos-nfs-storage  #provisioner名称, 该名称与 nfs-StorageClass.yaml文件中的provisioner名称保持一致
            - name: NFS_SERVER
              value: 10.90.15.xx   #NFS Server IP地址
            - name: NFS_PATH
              value: /public    #NFS挂载卷
      volumes:
        - name: nfs-client-root
          nfs:
            server: 10.90.15.xx  #NFS Server IP地址
            path: /public    #NFS 挂载卷
```

部署StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-nfs-storage
provisioner: eos-nfs-storage # 这里的名称要和provisioner配置文件中的环境变量PROVISIONER_NAME保持一致 
```


在StateFulSet中通过volumeClaimTemplates模板，动态创建PVC，PV

```yaml
volumeClaimTemplates:                   # 用于持久化的模板
  - metadata:
      name: redis-data
      annotations:
        # 主要是这一句的配置，指定StorageClass
        volume.beta.kubernetes.io/storage-class: "managed-nfs-storage"
    spec:
      accessModes: ["ReadWriteMany"]
      resources:
        requests:
          storage: 500M
```

## StorageClass 配置

PV的回收策略，可以从StorageClass继承来，对于NFS的provisioner，我们可以使用Retain和Delete的回收策略。特别注意，各个策略的支持看对应插件的控制器的实现

下面是删除策略，也就是当我们删除PVC的时候，PV也会被删除，然后在共享存储路径上创建的文件夹也一起被删除

对应NFS的provisioner，可以配置`archiveOnDelete`参数，这里我们设置是非存档。如果是true，那么文件夹不会被删除，二是在文件夹前面加上archive的前缀，做一个数据备份，也就是把数据存档了，之后的PV是不会去绑定这个文件夹的

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-nfs-storage
provisioner: eos-nfs-storage # 这里的名称要和provisioner配置文件中的环境变量PROVISIONER_NAME保持一致 
parameters:
  archiveOnDelete: "false" # 是否存档
reclaimPolicy: Delete
```

## 参考

https://blog.csdn.net/qq_38265137/article/details/83146421

https://blog.csdn.net/u011418530/article/details/89307874

https://www.cnblogs.com/panwenbin-logs/p/12196286.html

https://www.cnblogs.com/panwenbin-logs/p/12196286.html

https://blog.csdn.net/huwh_/article/details/82052191
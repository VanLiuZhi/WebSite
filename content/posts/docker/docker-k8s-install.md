---
weight: 1199
title: "Kubernetes(k8s) 安装过程"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Kubernetes(k8s) 安装过程"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Linux, Docker, Note]
categories: [Kubernetes]

lightgallery: true

toc:
  auto: false
---

Kubernetes(k8s) 安装过程

<!-- more -->

## K8S DashBoard服务

创建服务后通过代理访问

1. 创建服务
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
kubectl create -f kubernetes-dashboard.yaml

2. 开启代理
kubectl proxy 或 kubectl proxy &

访问:
http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/

3. 登录

配置文件

    Mac: $HOME/.kube/config
    Win: %UserProfile%\.kube\config

创建令牌
{% codeblock%}
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
{% endcodeblock %}

## 安装

安装总结：
1. 关闭各种服务，保证安装和集群建立的成功
2. 安装docker和k8s组件，并保证开启启动
3. 每台机器上都是一样的，当然master如果不参与负载，可以不装部分组件
4. 使用kubeadm部署Kubernetes，完成集群初始化

### 关于SeLinux

```s
关闭SeLinux，由于太复杂，关了
SELinux(Security-Enhanced Linux) 是美国国家安全局（NSA）对于强制访问控制的实现，是 Linux历史上最杰出的新安全子系统。NSA是在Linux社区的帮助下开发了一种访问控制体系，在这种访问控制体系的限制下，进程只能访问那些在他的任务中所需要文件。
```

`sed -i 's/SELINUX=*/SELINUX=disabled/' /etc/selinux/config`或修改配置 `/etc/selinux/config` SELINUX=disabled

### 配置cgroup drive

Cgroups是control groups的缩写，是Linux内核提供的一种可以限制、记录、隔离进程组（process groups）所使用的物理资源（如：cpu,memory,IO等等）的机制。最初由google的工程师提出，后来被整合进Linux内核。Cgroups也是LXC为实现虚拟化所使用的资源管理手段，可以说没有cgroups就没有LXC。
保证docker 和 kubelet的 cgroup 一致，Cgroup Driver:Cgroupfs 与 Systemd。查看（dockerk可以看到cgroup，kublete就不一定有了，这种情况就统一成一样的吧）
` cat /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf` 可能会在 `/etc/systemd/system/kubelet.service.d`
`docker info`
修改docker的deamon.json 

```s
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}
```

或者修改kublete，去配置文件里面，环境变量设置加上
`Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=systemd"`

### 安装 kubelet & kubeadm & kubectl

使用yum命令安装

kubelet:运行在cluster所有节点上，负责启动POD和容器
kubeadm:用于初始化cluster，通过它完成k8s的初始化
kubectl:kubectl是kubenetes命令行工具，通过kubectl可以部署和管理应用，查看各种资源，创建，删除和更新组件

## kubespray 安装方法

并不是很推荐的一种方式，应为换源的原因，kubespray的版本往往和镜像不一致，而且你找到的别人的安装总结都是用自己或别人的，总的来说这是一个不错的工具，但是在天朝局域网下特别难用，最好就是锁定kubespray版本，然后使用阿里云制作自己的镜像，然后kubespray里面把谷歌的镜像换成自己制作的。

### 批量换源

grc_image_files=(
kuberspray/extra_playbooks/roles/dnsmasq/templates/dnsmasq-autoscaler.yml
kuberspray/extra_playbooks/roles/download/defaults/main.yml
kuberspray/extra_playbooks/roles/kubernetes-apps/ansible/defaults/main.yml
kuberspray/roles/download/defaults/main.yml
kuberspray/roles/dnsmasq/templates/dnsmasq-autoscaler.yml
kuberspray/roles/kubernetes-apps/ansible/defaults/main.yml
)

grc_image_files=(
kubespray/roles/download/defaults/main.yml
kubespray/roles/kubernetes-apps/ansible/defaults/main.yml
)

grc_image_files=(
kubespray/roles/download/defaults/main.yml
)

for file in ${grc_image_files[@]} ; do
    sed -i 's/gcr.io\/google_containers/registry.cn-hangzhou.aliyuncs.com\/szss_k8s/g' $file
done

for file in ${grc_image_files[@]} ; do
    sed -i 's/gcr.io\/google_containers/registry.cn-hangzhou.aliyuncs.com\/szss_k8s/g' $file
done

quay_image_files=(
./kubespray/extra_playbooks/roles/download/defaults/main.yml
./kubespray/roles/download/defaults/main.yml
)

quay_image_files=(
kubespray/roles/download/defaults/main.yml
)

for file in ${quay_image_files[@]} ; do
    sed -i 's/quay.io\/coreos\//registry.cn-hangzhou.aliyuncs.com\/szss_quay_io\/coreos-/g' $file
    sed -i 's/quay.io\/calico\//registry.cn-hangzhou.aliyuncs.com\/szss_quay_io\/calico-/g' $file
    sed -i 's/quay.io\/l23network\//registry.cn-hangzhou.aliyuncs.com\/szss_quay_io\/l23network-/g' $file
done

安装失败清理 或者用清理命令

sudo rm -rf /etc/kubernetes/
sudo rm -rf /var/lib/kubelet
sudo rm -rf /var/lib/etcd
sudo rm -rf /usr/local/bin/kubectl
sudo rm -rf /etc/systemd/system/calico-node.service
sudo rm -rf /etc/systemd/system/kubelet.service
sudo systemctl stop etcd.service
sudo systemctl disable etcd.service
sudo systemctl stop calico-node.service
sudo systemctl disable calico-node.service
sudo docker stop $(docker ps -q)
sudo docker rm $(docker ps -a -q)
sudo service docker restart

### 其他命令

CONFIG_FILE=./kubespray/inventory/inventory.cfg python36 ./kubespray/contrib/inventory_builder/inventory.py ${IP[*]}
ansible-playbook -i hosts.ini /etc/kubespray/reset.yml

1. 流程总结

虚拟机: vagrant virtual box，虚拟的操作都使用vagrant，最好是使用一个Vagrantfile文件来启动和管理集群，这样比较方便

网络配置: 使用私有网络，所有虚拟机处在相同的子网当中，相互之间能通信，也能和外网通信

系统配置：检查虚拟机网络是否联通，完成相关配置（最好不要用临时配置的方式）

开始安装：下载kubespray，开始安装

参考：https://www.one-tab.com/page/HIr4mKK3SBKJJBDk5bNz_A

2. 虚拟机

使用vagrant来管理虚拟机，下面是一个配置文件，vagrant使用ruby来编写配置文件

```ruby
Vagrant.configure(2) do |config|
  # 如果需要三台就(1..3)的形式
  (1..3).each do |i|
    config.vm.define "cluster#{i}" do |node|
      node.vm.box = "centos_k8s"
      node.vm.hostname = "cluster#{i}"
      node.vm.network "private_network", ip: "192.168.59.#{i}"
      # 映射目录 根据自己实际情况配置
      # node.vm.synced_folder "../var/www/cluster", "/var/www/cluster"
      node.vm.provider "virtualbox" do |v|
        v.name = "cluster#{i}"
        v.memory = 512
        v.cpus = 1
      end
    end
  end
end
```

find /etc/kubespray -name '*.yml' | xargs -n1 -I{} sed -i "s/gcr\.io\/google_containers/gcr\.mirrors\.ustc\.edu\.cn\/google-containers/g" {}
find /etc/kubespray -name '*.yml' | xargs -n1 -I{} sed -i "s/gcr\.io\/google-containers/gcr\.mirrors\.ustc\.edu\.cn\/google-containers/g" {}
find /etc/kubespray -name '*.yml' | xargs -n1 -I{} sed -i 's/quay\.io/quay-mirror\.qiniu\.com/' {}

find /etc/kubespray -name '*.yml' | xargs -n1 -I{} sed -i "s/mirrorgooglecontainers\/google-containers/registry\.cn-hangzhou\.aliyuncs\.com\/google_containers/g" {}
find /etc/kubespray -name '*.yml' | xargs -n1 -I{} sed -i "s/mirrorgooglecontainers\/google-containers/registry\.cn-hangzhou\.aliyuncs\.com\/google_containers/g" {}
find /etc/kubespray -name '*.yml' | xargs -n1 -I{} sed -i 's/quay\.io/quay-mirror\.qiniu\.com/' {}

## kubeadm 安装(精简版)

官方安装工具，推荐使用的一种安装方式，最大的问题可能仍然是镜像，通过换源使用阿里的仓库，阿里的仓库已经有了同名的所需的镜像，而且是不用科学上网的

### 环境准备

所有机器都要执行，这里使用的用户是普通用户，所以命名涉及权限的都要加sudo

1. 设置主机名hostname，管理节点设置主机名为 master，其它设置为node

```s
sudo hostnamectl set-hostname master
```

2. 然后配置/etc/hosts

```s
192.168.59.101 cluster1

192.168.59.102 cluster2

192.168.59.103 cluster3
```

3. 关闭防火墙、selinux和swap

```s
sudo systemctl stop firewalld

sudo systemctl disable firewalld

sudo setenforce 0

sudo sed -i "s/^SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config

sudo swapoff -a

sudo sed -i 's/.*swap.*/#&/' /etc/fstab
```

4. 配置内核参数，将桥接的IPv4流量传递到iptables

```s
sudo touch /etc/sysctl.d/k8s.conf && sudo vim /etc/sysctl.d/k8s.conf

net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1

执行：
sudo modprobe br_netfilter
sysctl -p /etc/sysctl.d/k8s.conf 
sudo sysctl --system 不推荐
使内核参数生效(sysctl --system，这个会加载所有的sysctl配置，sysctl -p默认无参数只加载/etc/sysctl.conf)
```

5. 配置国内yum源

```s
sudo yum install -y wget

sudo mkdir /etc/yum.repos.d/bak && sudo mv /etc/yum.repos.d/*.repo /etc/yum.repos.d/bak

sudo wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.cloud.tencent.com/repo/centos7_base.repo

sudo wget -O /etc/yum.repos.d/epel.repo http://mirrors.cloud.tencent.com/repo/epel-7.repo

sudo yum clean all && sudo yum makecache
```

6. 配置国内Kubernetes源

```s
sudo touch /etc/yum.repos.d/kubernetes.repo && sudo vim /etc/yum.repos.d/kubernetes.repo

[kubernetes]

name=Kubernetes

baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/

enabled=1

gpgcheck=1

repo_gpgcheck=1

gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
```

7. 配置 docker 源

```s
sudo wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
```

至此，环境准备工作完成，除了主机名称设置不一样，其它操作每台机器执行都是一样的

### 软件安装

所有机器都执行

1. 安装docker

这里指定了版本号(docker-ce就默认安装最新的)

```s
sudo yum install -y docker-ce-18.06.1.ce-3.el7

sudo systemctl enable docker && sudo systemctl start docker

查看安装：
sudo docker –version

Docker version 18.06.1-ce, build e68fc7a
```

2. 安装kubeadm、kubelet、kubectl

没有指定版本，默认安装最新的，这里很重要，应为k8s的版本和这里所安装的版本有关，如果你不指定版本，k8s安装也是默认用最新的

```s
sudo yum install -y kubelet kubeadm kubectl
sudo systemctl enable kubelet
```

修改配置

```s
修改cgroups，在末尾加上"--cgroup-driver=cgroupfs"
sudo vim /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --cgroup-driver=cgroupfs"
```

写法是多样的，新版本官方推荐使用systemd而不是cgroupfs

```s
Environment="KUBELET_CGROUP_ARGS=--cgroup-driver=systemd"
```

### 部署master节点

1. 拉取镜像，创建集群(只在master节点执行)

```s
sudo kubeadm init --kubernetes-version=1.15.2 --apiserver-advertise-address=192.168.59.101 --image-repository registry.aliyuncs.com/google_containers --service-cidr=10.1.0.0/16 --pod-network-cidr=10.244.0.0/16
```

指定了网段，镜像仓库，k8s版本，这里的版本要和上面安装的软件对应上，命令执行失败按照提示修改即可，可能会出现的情况，机器配置不足，docker cgroup-driver 警告等

2. 执行流程

```s
init] Using Kubernetes version: v1.15.2
[preflight] Running pre-flight checks
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Activating the kubelet service
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [cluster1 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.1.0.1 192.168.59.101]
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [cluster1 localhost] and IPs [192.168.59.101 127.0.0.1 ::1]
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [cluster1 localhost] and IPs [192.168.59.101 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 23.503947 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.15" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node cluster1 as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node cluster1 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: sqvswo.903bhdss7w01bsac
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy
```

创建成功后，会返回这样的信息，这个信息很重要，节点加入集群需要用到，忘记了可以重新找到(百度)

```s
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.59.101:6443 --token sqvswo.903bhdss7w01bsac \
    --discovery-token-ca-cert-hash sha256:1cbc26f1d257f042d382273250dbfe52275f90be6784c743200738e4142f9e30
```

3. 配置kubectl工具

大致就是需要配置环境变量才能正确使用kubectl

```
方法1:

[root@k8s-master ~]# export KUBECONFIG=/etc/kubernetes/admin.conf
#此处如果没有声明环境变量，是没有加载管理k8s集群的权限的，此时去查看集群，会提示拒绝了该请求。如下：The connection to the server localhost:8080 was refused - did you specify the right host or port?

方法2(推荐):

[root@k8s-master ~]# mkdir -p /root/.kube
[root@k8s-master ~]# cp -i /etc/kubernetes/admin.conf /root/.kube/config 

测试命令，返回Healthy说明master功能正常
[root@k8s-master ~]# kubectl get cs
NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok                  
controller-manager   Healthy   ok                  
etcd-0               Healthy   {"health":"true"}   

[root@k8s-master ~]# kubectl get node
NAME         STATUS     ROLES    AGE     VERSION
k8s-master   NotReady   master   2m10s   v1.15.2
```

4. 部署网络

这一步很重要，k8s通过安装插件的形式来和其它node节点通信，插件可以自选，这里使用flannel由于上面的步骤已经通过换源，更改镜像仓库的形式保证了网络正常，而flannel同样也是需要翻墙的。

流程就是sudo kubectl apply -f kube-flannel.yml即可部署flannel网络

所以我们需要先去下载一个kube-flannel.yml文件，执行就要拉取镜像了，由于网络问题，一般就两种解决方案:

- 修改文件，把镜像换成可以访问的
- 通过其它地址拉取镜像，重新打标签

第二种操作难度要低的很多，这里用别人做好的，如果不能拉取自己通过阿里镜像仓库做一个，拉取了再打标签

```s
mkdir k8s && cd k8s
curl -O https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
sudo docker pull quay-mirror.qiniu.com/coreos/flannel:v0.11.0-amd64
sudo docker tag quay-mirror.qiniu.com/coreos/flannel:v0.11.0-amd64 quay.io/coreos/flannel:v0.11.0-amd64
sudo kubectl apply -f kube-flannel.yml
```

执行命令 sudo kubectl get pods -n kube-system pod都处于Running状态，集群状态即为正常运行

### node节点加入集群

node节点也需要部署flannel、pause、kube-proxy的pod，所以需要预先进行下载镜像，版本如下，由于k8s使用的是1.15.2，flannel在安装的时候是v0.11.0-amd64 

```s
k8s.gcr.io/kube-proxy-amd64:v1.15.2 
quay.io/coreos/flannel:v0.11.0-amd64 
k8s.gcr.io/pause:3.1
```

镜像在需要的时候就回去拉取，由于我们已经换源了，所以到这步，各个node节点可以看到

```s
registry.aliyuncs.com/google_containers/kube-proxy   v1.15.2
registry.aliyuncs.com/google_containers/pause        3.1
```

也就是只差着flannel镜像了，通过拉取可以拉取的重新打标签的方式，如此镜像就准备好了

执行上面集群初始化返回的命令

```s
kubeadm join 192.168.59.101:6443 --token 5r21h1.gv4kimitgx4x07ct --discovery-token-ca-cert-hash sha256:0e1b2352024abad3cf0a0d301e66e0a5eef4147a34745a3843aa52ec01578871
```

一般都是返回成功的，其实还没有完成，拉取镜像的步骤在之后进行，可以在master上sudo kubectl get pods -n kube-system pod查看是否都是running，如果flannel无法拉取，就会卡在那里，所以最好先准备好镜像

kubectl get nodes 检查集群状态

### 总结

1. 准备环境
2. 安装软件
3. 部署master和node

准备环境都是那几步骤，除非你的内核太低了。安装软件不指定版本，默认最新，上面就是1.15.2的，那么后续的都是这个版本为主。部署网络也是，你下载的文件可能使用的镜像不是v0.11.0-amd64，也要注意处理

如果在集群安装过程中有遇到其他问题，可以使用以下命令进行重置：

```s
初始化(所有机器都执行)
$ kubeadm reset
大概是移除网络，不一定需要，如果网络已有了可以执行清理，通过ifconfig查看
$ sudo ifconfig cni0 down && sudo ip link delete cni0
$ sudo ifconfig flannel.1 down && sudo ip link delete flannel.1
删除cni文件
$ sudo rm -rf /var/lib/cni/
```

### 关于网卡问题导致网络通信失败

这是使用vagrant导致的，vagrant使用第一网卡来做ssh，导致flannel通信失败，需要我们创建集群的时候手动指定网卡，这是本地环境搭建，而且使用vagrant的时候会遇到的一个问题

这里列出了v0.11.0-amd64版本的yml

```yml
---
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: psp.flannel.unprivileged
  annotations:
    seccomp.security.alpha.kubernetes.io/allowedProfileNames: docker/default
    seccomp.security.alpha.kubernetes.io/defaultProfileName: docker/default
    apparmor.security.beta.kubernetes.io/allowedProfileNames: runtime/default
    apparmor.security.beta.kubernetes.io/defaultProfileName: runtime/default
spec:
  privileged: false
  volumes:
    - configMap
    - secret
    - emptyDir
    - hostPath
  allowedHostPaths:
    - pathPrefix: "/etc/cni/net.d"
    - pathPrefix: "/etc/kube-flannel"
    - pathPrefix: "/run/flannel"
  readOnlyRootFilesystem: false
  # Users and groups
  runAsUser:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  fsGroup:
    rule: RunAsAny
  # Privilege Escalation
  allowPrivilegeEscalation: false
  defaultAllowPrivilegeEscalation: false
  # Capabilities
  allowedCapabilities: ['NET_ADMIN']
  defaultAddCapabilities: []
  requiredDropCapabilities: []
  # Host namespaces
  hostPID: false
  hostIPC: false
  hostNetwork: true
  hostPorts:
  - min: 0
    max: 65535
  # SELinux
  seLinux:
    # SELinux is unsed in CaaSP
    rule: 'RunAsAny'
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: flannel
rules:
  - apiGroups: ['extensions']
    resources: ['podsecuritypolicies']
    verbs: ['use']
    resourceNames: ['psp.flannel.unprivileged']
  - apiGroups:
      - ""
    resources:
      - pods
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - nodes
    verbs:
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - nodes/status
    verbs:
      - patch
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: flannel
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: flannel
subjects:
- kind: ServiceAccount
  name: flannel
  namespace: kube-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: flannel
  namespace: kube-system
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: kube-flannel-cfg
  namespace: kube-system
  labels:
    tier: node
    app: flannel
data:
  cni-conf.json: |
    {
      "name": "cbr0",
      "plugins": [
        {
          "type": "flannel",
          "delegate": {
            "hairpinMode": true,
            "isDefaultGateway": true
          }
        },
        {
          "type": "portmap",
          "capabilities": {
            "portMappings": true
          }
        }
      ]
    }
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: kube-flannel-ds-amd64
  namespace: kube-system
  labels:
    tier: node
    app: flannel
spec:
  selector:
    matchLabels:
      app: flannel
  template:
    metadata:
      labels:
        tier: node
        app: flannel
    spec:
      hostNetwork: true
      nodeSelector:
        beta.kubernetes.io/arch: amd64
      tolerations:
      - operator: Exists
        effect: NoSchedule
      serviceAccountName: flannel
      initContainers:
      - name: install-cni
        image: quay.io/coreos/flannel:v0.11.0-amd64
        command:
        - cp
        args:
        - -f
        - /etc/kube-flannel/cni-conf.json
        - /etc/cni/net.d/10-flannel.conflist
        volumeMounts:
        - name: cni
          mountPath: /etc/cni/net.d
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      containers:
      - name: kube-flannel
        image: quay.io/coreos/flannel:v0.11.0-amd64
        command:
        - /opt/bin/flanneld
        args:
        - --ip-masq
        - --kube-subnet-mgr
        - --iface=eth1
        resources:
          requests:
            cpu: "100m"
            memory: "50Mi"
          limits:
            cpu: "100m"
            memory: "50Mi"
        securityContext:
          privileged: false
          capabilities:
             add: ["NET_ADMIN"]
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        volumeMounts:
        - name: run
          mountPath: /run/flannel
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      volumes:
        - name: run
          hostPath:
            path: /run/flannel
        - name: cni
          hostPath:
            path: /etc/cni/net.d
        - name: flannel-cfg
          configMap:
            name: kube-flannel-cfg
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: kube-flannel-ds-arm64
  namespace: kube-system
  labels:
    tier: node
    app: flannel
spec:
  selector:
    matchLabels:
      app: flannel
  template:
    metadata:
      labels:
        tier: node
        app: flannel
    spec:
      hostNetwork: true
      nodeSelector:
        beta.kubernetes.io/arch: arm64
      tolerations:
      - operator: Exists
        effect: NoSchedule
      serviceAccountName: flannel
      initContainers:
      - name: install-cni
        image: quay.io/coreos/flannel:v0.11.0-arm64
        command:
        - cp
        args:
        - -f
        - /etc/kube-flannel/cni-conf.json
        - /etc/cni/net.d/10-flannel.conflist
        volumeMounts:
        - name: cni
          mountPath: /etc/cni/net.d
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      containers:
      - name: kube-flannel
        image: quay.io/coreos/flannel:v0.11.0-arm64
        command:
        - /opt/bin/flanneld
        args:
        - --ip-masq
        - --kube-subnet-mgr
        resources:
          requests:
            cpu: "100m"
            memory: "50Mi"
          limits:
            cpu: "100m"
            memory: "50Mi"
        securityContext:
          privileged: false
          capabilities:
             add: ["NET_ADMIN"]
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        volumeMounts:
        - name: run
          mountPath: /run/flannel
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      volumes:
        - name: run
          hostPath:
            path: /run/flannel
        - name: cni
          hostPath:
            path: /etc/cni/net.d
        - name: flannel-cfg
          configMap:
            name: kube-flannel-cfg
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: kube-flannel-ds-arm
  namespace: kube-system
  labels:
    tier: node
    app: flannel
spec:
  selector:
    matchLabels:
      app: flannel
  template:
    metadata:
      labels:
        tier: node
        app: flannel
    spec:
      hostNetwork: true
      nodeSelector:
        beta.kubernetes.io/arch: arm
      tolerations:
      - operator: Exists
        effect: NoSchedule
      serviceAccountName: flannel
      initContainers:
      - name: install-cni
        image: quay.io/coreos/flannel:v0.11.0-arm
        command:
        - cp
        args:
        - -f
        - /etc/kube-flannel/cni-conf.json
        - /etc/cni/net.d/10-flannel.conflist
        volumeMounts:
        - name: cni
          mountPath: /etc/cni/net.d
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      containers:
      - name: kube-flannel
        image: quay.io/coreos/flannel:v0.11.0-arm
        command:
        - /opt/bin/flanneld
        args:
        - --ip-masq
        - --kube-subnet-mgr
        resources:
          requests:
            cpu: "100m"
            memory: "50Mi"
          limits:
            cpu: "100m"
            memory: "50Mi"
        securityContext:
          privileged: false
          capabilities:
             add: ["NET_ADMIN"]
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        volumeMounts:
        - name: run
          mountPath: /run/flannel
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      volumes:
        - name: run
          hostPath:
            path: /run/flannel
        - name: cni
          hostPath:
            path: /etc/cni/net.d
        - name: flannel-cfg
          configMap:
            name: kube-flannel-cfg
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: kube-flannel-ds-ppc64le
  namespace: kube-system
  labels:
    tier: node
    app: flannel
spec:
  selector:
    matchLabels:
      app: flannel
  template:
    metadata:
      labels:
        tier: node
        app: flannel
    spec:
      hostNetwork: true
      nodeSelector:
        beta.kubernetes.io/arch: ppc64le
      tolerations:
      - operator: Exists
        effect: NoSchedule
      serviceAccountName: flannel
      initContainers:
      - name: install-cni
        image: quay.io/coreos/flannel:v0.11.0-ppc64le
        command:
        - cp
        args:
        - -f
        - /etc/kube-flannel/cni-conf.json
        - /etc/cni/net.d/10-flannel.conflist
        volumeMounts:
        - name: cni
          mountPath: /etc/cni/net.d
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      containers:
      - name: kube-flannel
        image: quay.io/coreos/flannel:v0.11.0-ppc64le
        command:
        - /opt/bin/flanneld
        args:
        - --ip-masq
        - --kube-subnet-mgr
        resources:
          requests:
            cpu: "100m"
            memory: "50Mi"
          limits:
            cpu: "100m"
            memory: "50Mi"
        securityContext:
          privileged: false
          capabilities:
             add: ["NET_ADMIN"]
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        volumeMounts:
        - name: run
          mountPath: /run/flannel
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      volumes:
        - name: run
          hostPath:
            path: /run/flannel
        - name: cni
          hostPath:
            path: /etc/cni/net.d
        - name: flannel-cfg
          configMap:
            name: kube-flannel-cfg
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: kube-flannel-ds-s390x
  namespace: kube-system
  labels:
    tier: node
    app: flannel
spec:
  selector:
    matchLabels:
      app: flannel
  template:
    metadata:
      labels:
        tier: node
        app: flannel
    spec:
      hostNetwork: true
      nodeSelector:
        beta.kubernetes.io/arch: s390x
      tolerations:
      - operator: Exists
        effect: NoSchedule
      serviceAccountName: flannel
      initContainers:
      - name: install-cni
        image: quay.io/coreos/flannel:v0.11.0-s390x
        command:
        - cp
        args:
        - -f
        - /etc/kube-flannel/cni-conf.json
        - /etc/cni/net.d/10-flannel.conflist
        volumeMounts:
        - name: cni
          mountPath: /etc/cni/net.d
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      containers:
      - name: kube-flannel
        image: quay.io/coreos/flannel:v0.11.0-s390x
        command:
        - /opt/bin/flanneld
        args:
        - --ip-masq
        - --kube-subnet-mgr
        resources:
          requests:
            cpu: "100m"
            memory: "50Mi"
          limits:
            cpu: "100m"
            memory: "50Mi"
        securityContext:
          privileged: false
          capabilities:
             add: ["NET_ADMIN"]
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        volumeMounts:
        - name: run
          mountPath: /run/flannel
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      volumes:
        - name: run
          hostPath:
            path: /run/flannel
        - name: cni
          hostPath:
            path: /etc/cni/net.d
        - name: flannel-cfg
          configMap:
            name: kube-flannel-cfg
```
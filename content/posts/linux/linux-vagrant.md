---
weight: 1
title: "vagrant 运用"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "vagrant 运用"
resources:
- name: "base-image"
  src: "base-image.jpg"

tags: [Linux, Note]
categories: [操作系统]

lightgallery: true

toc:
  auto: false
---

虚拟机是学习分布式道路上逃不过的一环，即使你用Mac来开发，掌握虚拟机，把服务跑在虚拟机里面仍然是很有必要的，有些公司就以虚拟机为工作环境，保证线上和线下的一直性。vagrant是虚拟机API工具，可以很方便的管理和使用虚拟机。通过不断的销毁和创建虚拟机，总结了一些经验。总之，还是涉及到的知识太多，以及对Linux和网络相关的知识不熟悉，利用虚拟机API，可以帮助你更快的进行实践，不过硬盘表示很遭罪

<!-- more -->

## 常用命令

vagrant init 加 box name 通过box name 在当前目录创建VagrantFile，如果本地没有box就使用服务器上的
box 下载后的目录 `~/.vagrant.d/boxes`，这个box就是基础镜像，只要有这个box，就可以创建多个虚拟机，虚拟机保存在`VirtualBox VMs`目录下(Mac系统)

vagrant package 对当前环境进行打包
`vagrant box add hahaha ~/box/package.box  # 添加 package.box 镜像并命名为 hahaha`

这里添加的box可以在 `~/.vagrant.d/boxes` 找到

vagrant up 使用已有的文件创建虚拟机，后面加上文件夹路径

vagrant box 加其它命令，管理box，例如 vagrant box list

```s
$ vagrant init      # 初始化
$ vagrant up        # 启动虚拟机
$ vagrant halt      # 关闭虚拟机
$ vagrant reload    # 重启虚拟机
$ vagrant ssh       # SSH 至虚拟机
$ vagrant suspend   # 挂起虚拟机
$ vagrant resume    # 唤醒虚拟机
$ vagrant status    # 查看虚拟机运行状态
$ vagrant destroy   # 销毁当前虚拟机

#box管理命令
$ vagrant box list    # 查看本地box列表
$ vagrant box add     # 添加box到列表
$ vagrant box remove  # 从box列表移除
```

## 解决无法共享目录的问题

一般在官方下载别人做好的box，发现有些box无法共享目录，有些可以。在Windows上竟然没有报错，Mac上会有错误提示。升级内核后，也会导致无法共享目录，应该也是增强软件因为升级的关系没了。

可能和系统有关，ubuntu解决方案

    sudo apt-get update
    sudo apt-get install virtualbox-guest-utils

解决过程很简单，貌似我下了Ubuntu box没有无法共享目录的情况

centos 先进入虚拟机，安装软件，可能是为了编译用吧

    sudo yum update
    sudo yum install gcc
    sudo yum install kernel-devel

然后加载镜像，并挂载它，执行（box文件是vagrant官方下载比较多的一个版本，就发现了这个无法共享目录的问题）

```
sudo find / -name VBoxGuestAdditions.iso 在mac中，找到镜像，复制到容易发现的目录，比如下载目录
mount VBoxGuestAdditions.iso的目录(虚拟机里面要有这个文件)，这一步是挂载命令，挂载由于本身目录就不共享了，所以也可以用virtualBox来加载镜像（加载在mac中找到的镜像，镜像位于软件安装包中，可以用find命令查找具体路径），记得先关闭虚拟机

通过命令或virtualBox，完成镜像挂载后，重新进入虚拟机

通过命令查看挂载情况
lsscsi (适用于centos7)  yum install lsscsi 没有就安装一下
lsblk  (适用于ubuntu)

结果
[0:0:0:0]    disk    ATA      VBOX HARDDISK    1.0   /dev/sda
[0:0:1:0]    cd/dvd  VBOX     CD-ROM           1.0   /dev/sr0

可以看到，镜像已被挂载到/dev/sr0这个设备了

sudo mount /dev/sr0 /media/cdrom

/dev/sr0 是通过命令看到的，把sr0挂载到自己创建的目录/media/cdrom

cd 到该目录

sudo ./VBoxLinuxAdditions.run

退出虚拟机，重启即可(启动虚拟，没有报错，文件也能共享就是成功了)
```
最好把软件安装了再加载镜像，可能有些box已经有了软件不需要安装。

## 解决网络问题，无法解析主机

通过vagrant下载的centos Box，默认设置，在mac上可以访问网络，在windows上就不行，不知道是不是公司网络的原因

通过设置dns解决了，在/etc/resolv.conf添加一个nameserver 8.8.8.8，执行sudo yum clean all && sudo yum makecache

加了8.8.8.8还不行，nameserver 114.114.114.114 加上

确实是网络问题导致dns无法解析，上面的配置也不一定每次都好使

## 网络配置

默认情况下，采用Provider的NAT网络模式，在虚拟机中可以访问宿主机，也可以使用宿主机的外网路由上网。
virtual Box 可以设置很多的网络模式，不过这里要从vagrant来看，分为三种

1. Forwarded port(端口映射)
config.vm.network "forwarded_port", guest: 80, host: 8080
采用端口映射从主机特定端口访问虚拟机

2. Private network(私有网络)
config.vm.network "private_network", ip: "192.168.1.4" # 固定IP
config.vm.network "private_network", type: "dhcp" # 由路由器分配

只有主机可以访问虚拟机，如果多个虚拟机设置定在同一个网段也可以相互访问，虚拟机也是可以访问外部网络

3. Public network(公有网络)

config.vm.network "public_network", ip: "192.168.1.4"
config.vm.network "public_network", type: "dhcp"

相当于把虚拟机加入当前子网络

测试：修改配置文件，使用私有网络，然后利用python启动文件服务器 `python -m SimpleHTTPServer` or `python -m http.server 8080(py3)` 在外部通过设置的ip加默认文件服务的端口8000访问，成功代表网络设置正确

虚拟机通过 ip address 可以看到，拥有三个网络了

## 使用root用户

sudo passwd root

输入密码，不要太短，两次后确认（vagrant默认的root用户密码是vagrant，如果不更改密码，可以不执行这一步）

su root 

切换到root用户

测试使用配置文件修改用户为root等设置后，无法启动系统，不知道是不是vagrant BUG，如果要用root用户，连接后切换，如果是非windows系统，配置好root用户后，通过ssh来连接放弃vagrant ssh的方式

## 关于ssh相关问题

既然是为了学习分布式，很多工具都需要通过ssh控制目标机，保证ssh的连接正确性是关键的第一步。

ssh登录处在相同子网的其它服务器，直接复制公钥到目标机就行了，然而这一招在root用户下不管用了（之所以直接复制，也是一开对机制不熟悉，如果有上百台机器，肯定只能用命令），这里猜测还是流程上不标准吧，正确的处理应该是修改需要连接的目标机的sshd，即ssh服务端配置，允许使用公钥的方式，配置文件不满足要求的，就去修改它

有一个很重要的概念：
当主机中开启openssh服务，那么就对外开放了远程连接的接口，ssh为openssh服务的客户端，sshd为openssh服务的服务端
ssh和sshd的差别，一定要牢记，不要混淆，别人连接你，你要修改的是服务端的配置
注意sshd只有root用户组的才有读写权限，其它都没有

相关命令

```s
systemctl restart sshd.service

systemctl status sshd.service #查看ssh服务的状态

systemctl start sshd.service  #开启ssh服务

sytemctl enable sshd.service #ssh服务随开机启动，还有个disabled

systemctl stop sshd.ervice #停止

正常情况下应该是Active：active（running）
```

权限配置
```s
chmod 700 /home/Hadoop/.ssh
chmod 644 /home/Hadoop/.ssh/authorized_keys
如果权限不对，会导致文件无法读取导致失败，还有这些配置是跟着各个用户走的，它们的用户组也必须正确
```

关于配置文件，一般关注的是服务端的配置文件，以及几个常用的参数

PermitRootLogin no
"PermitRootLogin”设置是否允许root通过ssh登录。这个选项从安全角度来讲应设成"no"。

RSAAuthentication yes
"RSAAuthentication”设置是否允许只有RSA安全验证。

PasswordAuthentication yes
"PasswordAuthentication”设置是否允许口令验证。

PermitEmptyPasswords no
"PermitEmptyPasswords”设置是否允许用口令为空的帐号登录。

AllowUsers admin   #限定登录的用户名
"AllowUsers”的后面可以跟任意的数量的用户名的匹配串，这些字符串用空格隔开。主机名可以是域名或IP地址。

以上是关于ssh相关的知识，正常来说，ssh连接目标机器，然后输入密码就行了，这是正常情况，如果不行，那么可能是配置文件被修改了，或是权限的问题，如果这些处理了，肯定是一些吊毛情况，建议卸载了重新安装，另外通过手动复制公钥到目标机的做法不一定每次都行，我就遇到过通过命令无法实现，然后手动复制可以了，但是对于root用户就不行。

重装方法

yum -y remove openssh-server
yum -y install openssh-server

执行 /usr/sbin/sshd 启动服务（使用此方法重装后，systemctl和服务相关的命令有问题，通过测试发现，如果要修改配置，修改后执行/usr/sbin/sshd就会生效，整个服务默认是开机启动的，测试的不是很全面，可以查找相关资料了解一下）

ssh-keygen
ssh-copy-id root@192.168.59.102

不同身份的用户都需要执行这两步，上面只是让root用户可以连接，vagrant也需要执行一次，就是说不同的用户要用自己的公钥


---
weight: 6600
title: "Web Confluence"
subtitle: "Web Confluence"
date: 2020-09-21T10:41:18+08:00
lastmod: 2020-09-21T10:41:18+08:00
draft: true
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Web Confluence"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Note]
categories: [Clear-Mind]

lightgallery: true

toc:
  auto: false
---

工作协调管理程序

<!--more-->

# confluence

工作协调管理程序

## 部署

docker run -d --name confluence -p 8090:8090 --user root:root cptactionhank/atlassian-confluence:latest

## 破解

docker exec -it confluence /bin/sh

拷贝文件到本地，注意版本
docker cp confluence:/opt/atlassian/confluence/confluence/WEB-INF/lib/atlassian-extras-decoder-v2-3.4.1.jar atlassian-extras-decoder-v2-3.4.1.jar
替换名称，然后传给破解程序
mv ~/atlassian-extras-decoder-v2-3.4.1.jar  ~/atlassian-extras-2.4.jar
mv ~/atlassian-extras-2.4.jar ~/atlassian-extras-decoder-v2-3.4.1.jar
把破解好的文件名字改回来，然后复制回容器覆盖
docker cp atlassian-extras-decoder-v2-3.4.1.jar confluence:/opt/atlassian/confluence/confluence/WEB-INF/lib/atlassian-extras-decoder-v2-3.4.1.jar

/opt/atlassian/application-data/confluence/confluence.cfg.xml

步骤：
1. 启动镜像
2. 复制一个jar到本地，用破解程序破解后传回容器
3. 完成破解
4. mysql连接要处理字符集，排序，事务隔离级别问题

使用如下命令创建数据库，然后连接的时候指定事务隔离级别即可

create schema confluence character set utf8 collate utf8_bin

jdbc:mysql://xx:3306/confluence?sessionVariables=tx_isolation='READ-COMMITTED'&useUnicode=true&characterEncoding=utf8
jdbc:mysql://xx:3306/confluence?sessionVariables=tx_isolation='READ-COMMITTED'&useUnicode=true&characterEncoding=utf8

## docker部署存在问题

系统对MySQL的配置有要求，可能需要调整

http://xx:8090/plugins/servlet/troubleshooting/view?source=notification&healthCheck=innoDBLogFileSizeCheck、

## 使用教程

https://blog.csdn.net/u011456337/article/details/103084838


## 部署mysql

docker run -p 3306:3306 --name mysql-5.7-docker \
-v /data/mysql/log:/var/log/mysql \
-v /data/mysql/data:/var/lib/mysql \
-v /data/mysql/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=h3cqdb45EX  \
-d repository/mysql:5.7 \
--default-time_zone='+8:00'

docker run -p 5056:3306 --name confluence \
-v /data/mysql2/log:/var/log/mysql \
-v /data/mysql2/data:/var/lib/mysql \
-v /data/mysql2/conf:/etc/mysql \
-e MYSQL_ROOT_PASSWORD=h3cqdb45EX  \
-d mysql:5.7

docker run -p 8090:8090 --name confluence --user root:root \
-v /data/confluence/backups:/var/atlassian/confluence/backups \
-e CATALINA_OPTS=-Duser.timezone=GMT+08  \
-d repository/cptactionhank/atlassian-confluence:latest

## 账户

admin 123456
admin h3c!agl895AZ

## LDAP配置

用自己的ldap账号，需要配置连接的地址，然后用户名，用户名是通过这种标签去定位的，然后用自己的密码去登录
一般大家的账号都有获取数据的权限，我们把数据同步过来即可，不要去修改ldap服务上的数据

服务地址 xx.com  
用户名 CN=liuzhi xx,OU=partnerusers,DC=xx,DC=xx,DC=com
CN=liuzhi ys3415,OU=partnerusers,DC=h3c,DC=huawei-3com,DC=com
CN=App EOS,OU=AppUsers,DC=h3c,DC=huawei-3com,DC=com
基本DN(减少数据筛选范围) DC=xx,DC=xxcom,DC=com

ADExplorer.exe 一个连接ldap的工具，可以查看数据

## world 文件乱码问题

C:\Windows\Fonts 目录下找到 simsun.ttc(新宋体；常规)

上传到服务器

注意：confluence官方镜像中默认已安装字体命令，所以/usr/share/fonts目录已经存在，你的若是没有该目录，那么你首先需要先进行字体命令的安装

进入容器创建目录
docker exec -it confluence /bin/bash 
mkdir /usr/share/fonts/chinese/
拷贝文件
docker cp simsun.ttc confluence:/usr/share/fonts/chinese/ 

编辑配置，进入容器
vi /opt/atlassian/confluence/bin/setenv.sh

加入这行配置
CATALINA_OPTS="-Dconfluence.document.conversion.fontpath=/usr/share/fonts/chinese/ ${CATALINA_OPTS}"

清空confluence缓存文件目录

cd /var/atlassian/confluencevi

删除viewfile目录和shared-home/dcl-document目录里的所有缓存文档文件

注：如果你不进行此操作，预览旧文件时，还是会出现乱码，只有新上传文件预览才正常

## 时区问题

environment:
    - 'CATALINA_OPTS= -Duser.timezone=GMT+08'

volumes:
  - /etc/localtime:/etc/localtime:ro

## 搜索不到用户的问题

有时候想把通过LDAP同步来的账户添加到当前命名空间，但是用户存在却搜索不了的情况

解决办法: 要在全局权限中，先把用户组添加进去，搜不到的用户对应的用户组没有在里面

## 性能

environment:
    - 'JVM_MINIMUM_MEMORY=4096m'
    - 'JVM_MAXIMUM_MEMORY=8192m'


    liuzhi ys3415

    huawei-3com


export JVM_MINIMUM_MEMORY="2096m"
export JVM_MAXIMUM_MEMORY="8192m"

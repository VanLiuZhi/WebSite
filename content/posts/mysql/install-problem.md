---
weight: 1
title: "MySQL install-problem"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "MySQL install-problem"
resources:
- name: "base-image"
  src: "base-image.jpg"

tags: [DataBase, Note]
categories: [DataBase]

lightgallery: true

toc:
  auto: false
---

安装MySQL遇到的问题

<!-- more -->

# 安装遇到的问题

安装软件总会遇到很多问题，很多问题仅在当时情况下出现，其实解决的思路都是大致的，多看报错和官方文档。

## mysql.sock 无法被创建

mysql 的链接需要借助这个套接字，如果不小心删除了，会导致服务无法运行

mac mysql 8.0.12安装教程

本次中Mac上使用非root账户来安装mysql，终于装好了。主要原因在于8.0版本配置（说是没有配置，但还说有，通过brew安装的可以在 /usr/local/etc 目录下找到配置文件，linux包管理软件安装的东西一般都在这个目录）没有配置东西，用的是默认的，这导致要创建链接的socket的时候没有指定目录使用了默认的 /tmp，没有执行/tmp的权限或者是用户所属等问题导致创建失败（并不是很建议去操作这个文件，有些人直接运行成功了，可能和个人的系统有很大关系）。无法创建mysql服务

大概解决思路
mysql安装会在data目录下生成很多东西，如果没有或生成的不对，可以把目录删除了，通过启动服务来生成（我就说这么干的），然后配置了socket的客户端和服务端的路径，路径指向data即可（以为mysql的data是有执行权限的），这样服务就启动成功了，然后安装brew指导，创建root用户。

配置

```s
[client]
socket=/usr/local/lnmp/mysql/data/mysql.sock
[mysqld]
socket=/usr/local/lnmp/mysql/data/mysql.sock
```
两个都要指定

mysql.sock 无法创建，问题可能是多方面的，在data目录下，会生成 .err后缀的文件，可以在里面查看错误信息，然后结合经验来解决问题。  还有就是data目录下数据不对，这个目录应该有很多文件的，但是我一直没有，我通过删除，再执行服务成功生成了，感觉并不是很正确的解决思路。

mysqld --help --verbose | less
这个目录可以列出mysql的一些信息，包括配置文件可能的路径，一些已经配置的参数。

最后，使用编译源代码的方式，估计是最后的活路。

## 报错：MySQL Illegal mix of collations for operation 'like'

MySQL Illegal mix of collations for operation 'like'

在 MySQL 5.5 以上, 若字段类型 Type 是 time,date,datetime 

在 select时如果使用 like '%中文%' 会出现 Illegal mix of collations for operation 'like'

在编程时要对每个字段进行查找，在执行时可能就会出现时间字段 like '%中文%' 这种语法，在旧版的 MySQL 是不会出现错误的。

升到 MySQL 5.5 以上, 必需改成 like binary '%中文%' 即可避免出现错误。

在python代码中，解决方案为使用cast(fields, CHAR)转换类型后再做like.(高mysql版本中使用like的使用，非字符类型和字符类型混合在filter条件中做like，就会导致这个问题)

## mac python2.7 安装 Mysql-python

Mac安装Mysql-python遇到的坑，被这俩报错反复摩擦：'my_config.h' file not found 和 IndexError: string index out of range

    brew install mysql
    brew unlink mysql
    brew install mysql-connector-c
    sed -i -e 's/libs="$libs -l "/libs="$libs -lmysqlclient -lssl -lcrypto"/g' /usr/local/bin/mysql_config
    pip install MySQL-python
    brew unlink mysql-connector-c
    brew link --overwrite mysql

解决参考[Stackoverflow](https://stackoverflow.com/questions/12218229/my-config-h-file-not-found-when-intall-mysql-python-on-osx-10-8/12233148)

如果MySQL安装了，那么安装mysql-connector-c后，执行 `sed -i -e 's/libs="$libs -l "/libs="$libs -lmysqlclient -lssl -lcrypto"/g' /usr/local/bin/mysql_config` 差不多就可以安装成功了。

本质就是读取配置文件错误，通过文本替换命令sed修改配置。


## 解决Access denied for user ''@'localhost' to database 'mysql

Access denied for user ''@'localhost' to database '

出现原因是MySQL的密码有问题，使用yum安装的时候，没设置密码进入交互环境后不能创建用户

用mysql匿名用户可以进入数据库，但是看不见mysql数据库.

解决办法：

具体操作步骤：

关闭mysql:

1. service mysqld stop

然后:

2. mysqld_safe --skip-grant-tables

启动mysql:

3. service mysqld start

mysql -u root

mysql> use mysql

mysql> UPDATE user SET Password=PASSWORD('xxx') WHERE user='root';

mysql> flush privileges;

到这里密码已经修改成功，

mysql -u root -p

mysql -udbappwgs -p -h127.0.0.1 有些时候要知道host才能连接上

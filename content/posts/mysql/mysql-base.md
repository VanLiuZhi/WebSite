---
weight: 1000
title: "Mysql 数据库基本"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Mysql 数据库基本"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [DataBase, Note]
categories: [DataBase]

lightgallery: true

toc:
  auto: false
---

mysql学习笔记

<!-- more -->

## 基础

一般对于命令是不区分大小写的，为了区分保留关键字，一般保留关键字大写，变量和数据小写。

    sudo service mysql start

    mysql -u root

    mysql -u root -p

    show databases;

    use dataname;

    show tables;

    create database; 库名    //创建数据库

    drop database; 库名      //删除数据库

/app/mysql/5.7.18/bin/mysql --socket=/app/mysql/5.7.18/dirstats/mysqld.sock -uroot -h134.108.3.196  -p

请先到 /etc/mysql   配置  my.cnf  避免中文插入有误

```s
[client]  
default-character-set=utf8  
  
[mysqld]  
character-set-server=utf8  
collation-server=utf8_general_ci
```

mysql --verbose --help | grep -A 1 'Default options' 

## 理解数据库和实例，数据库引擎

关于引擎：引擎是数据库进行读写，保存，执行事务等一系列行为时，如何进行这些行为，就是由引擎来决定的。mysql默认会使用InnoDB引擎，引擎可以修改，可以对表指定不同的引擎（数据库应该要支持这种引擎），例如对于需要大量数据访问的，指定查询能力较强的引擎，对于需要事务处理的表，指定能处理事务的引擎。有些引擎是不支持事务的，InnoDB是功能较为齐全的引擎，各方面能力均衡。

索引：引擎不同，索引的实现也不用，保存的索引文件也不同（对于数据库来说，会有表结构文件，表数据文件，索引文件等，引擎不同也文件也不同，因为存储方式不一样）。
索引类型：B-Tree索引，哈希索引，空间数据索引（R-Tree），全文索引（数据库引擎要支持，这种索引用来做海量搜索，但是如果需求高，还是用专业的搜索引擎）。

主键索引，唯一索引，普通索引

mysql锁：大致分为表级锁，行级锁，页面锁。理解这三种锁很简单，从字面就可以看出锁的锁定粒度，也就知道他们的区别了。其中这个页面锁的粒度介于另外两者之间。同样，锁怎么去控制数据库，也是和引擎有关的。

乐观和悲观：常会看到乐观锁，悲观锁，是一类概念的统称，比如假定对数据库的操作都会产生资源竞争的问题，这个时候就要锁库，悲观的做法就是都锁，实现这样的锁也可以被称为悲观锁。乐观的概念就是相反的，你假定不会出现资源竞争的情况。

悲观的做法

悲观的做法表明，您应该完全锁定资源，直到完成它。如果没有人可以在您处理对象时获取对象上的锁定，那么可以确保对象没有被更改。
我们使用数据库锁有几个原因：
1. 数据库非常擅长管理锁并保持一致性。
2. 数据库是访问数据的最低级别 - 获取最低级别的锁也会防止其他进程尝试修改数据。 例如，DB中的直接更新，cron作业，清理任务等。
3. Django应用程序可以在多个进程 （例如工作者）上运行。 在应用程序级别维护锁将需要大量（不必要的）工作。

要在Django中锁定一个对象，我们使用 `select_for_update`。
   
主键 id int primary key not null auto_increment

外键 CONSTRAINT emp_fk FOREIGN KEY (in_dpt) REFERENCES department(dpt_name) CONSTRAINT 后面的名字在一张表里面不能重复

插入数据  INSERT INTO tablename (column, column)  VALUES （values）

**数据类型**

除了我们常用的，补充  可变字符VARCHAR， ENUM单选（必须是定义时枚举的值之一） SET多选 

**SQL约束**

通过对表的行为或列的数据做出限制，来确保表的数据的完整性、唯一性

**约束类型**

主键  默认值  唯一  外键 非空

主键 ：对于主键还有复合主键，由两个字段来确定唯一性，比如一个学生成绩表，有学号，课程号，成绩。通过学号和课程号我们可以得到他的成绩，这个两个字段就组成复合主键，联合主键就是多个字段来决定主键。

唯一 ： 这个约束就是这个字段的这一列的值是唯一的，在执行INSERT 语句的时候，如果插入重复的值则会失败。例如 UNIQUE （phone） 对这个字段进行唯一约束

非空约束： age INT(10) NOT NULL, 在创建的表的语句中出现了这个，就是非空约束，如果插入数据的时候不填，会警告，不会报错。（可能有些mySQL版本会报错）

select  fields from table where 限制条件

限制条件比如大于小于，在什么范围。 where in (3, 10)  where not in (5, 20)

where age between 20 and 25  年龄在20到25包括20和25

通配符  where fieldname like '12_'  可以匹配到123 125等   用  %  代表不定个未指定字符

## 概念内容

合并排序：一种将数据排序的算法，在数据库排序的时候使用，这种算法比较灵活，比如你不必把数据完全读取出来，这种方法称为 原地排序

二维阵列：最简单的数据结构，就是一张表

二叉查找树：用来做索引。数据在保存的时候，如果使用了索引，则这个索引信息将建立一个二叉查找树。当然的，你每次更新新的数据都要维护和更新这个查找数，这个算法比起从头到尾查找数据，将会快很多。

B+索引树：二叉的升级版，新的索引结构方式，支持范围查找，比如1到5，你只要找到1，在结果里面，1下面对应的数据一直到5，我们需要的就得到了。如果是二叉树，那么你要找1，然后找2，2还不一定有，接着找3...

哈希表：是一种数据结构，保存的数据是键值对类型的。散列函数（哈希函数）是把键转换成哈希码的函数，散列表（哈希表）是存放记录的数组（这个记录空间是一片连续的空间），数据的键通过散列函数得到一样结果的，都分类在一个数组里面。  概述就是对数据保存进行哈希，得到一个哈希表，去查找数据的时候，对数据的键进行哈希函数运算（得到哈希码后，可能还要进行求模运算才得到记录数组的下标），得到的结果就是记录里面数组的下标，在这个数组里面去找值。

一些概念扩展：数据库是数据库和实例结合的，数据库服务被启动后，用户需要链接到服务上，我们去链接数据库后，系统为你分配了各种资源来操作数据库，就是你得到了数据库的一个实例。这个链接由客户端管理器来处理

多实例：理解了实例和数据库后，我们可以使用多实例，你只需要部署一次数据库应用（在linux上装一个MySql server）然后通过多个实例进行链接，生成的数据库文件也是多个的，这样可以实现很多功能，比如主从数据库。

数据库是多个组件构成的：一般有查询管理器，数据管理器，工具，核心组件

核心组件：
进程管理器（process manager）：很多数据库具备一个需要妥善管理的进程/线程池。再者，为了实现纳秒级操作，一些现代数据库使用自己的线程而不是操作系统线程。
网络管理器（network manager）：网路I/O是个大问题，尤其是对于分布式数据库。所以一些数据库具备自己的网络管理器。
文件系统管理器（File system manager）：磁盘I/O是数据库的首要瓶颈。具备一个文件系统管理器来完美地处理OS文件系统甚至取代OS文件系统，是非常重要的。
内存管理器（memory manager）：为了避免磁盘I/O带来的性能损失，需要大量的内存。但是如果你要处理大容量内存你需要高效的内存管理器，尤其是你有很多查询同时使用内存的时候。
安全管理器（Security Manager）：用于对用户的验证和授权。
客户端管理器（Client manager）：用于管理客户端连接。
……

工具：
备份管理器（Backup manager）：用于保存和恢复数据。
复原管理器（Recovery manager）：用于崩溃后重启数据库到一个一致状态。
监控管理器（Monitor manager）：用于记录数据库活动信息和提供监控数据库的工具。
Administration管理器（Administration manager）：用于保存元数据（比如表的名称和结构），提供管理数据库、模式、表空间的工具。
……

查询管理器：
查询解析器（Query parser）：用于检查查询是否合法
查询重写器（Query rewriter）：用于预优化查询
查询优化器（Query optimizer）：用于优化查询
查询执行器（Query executor）：用于编译和执行查询
数据管理器：
事务管理器（Transaction manager）：用于处理事务
缓存管理器（Cache manager）：数据被使用之前置于内存，或者数据写入磁盘之前置于内存
数据访问管理器（Data access manager）：访问磁盘中的数据

获取数据和联接数据

如何获取数据？

全扫描：完全读取一个表或者索引

范围扫描：where语句

唯一扫描：索引中获取一个值

存取路径：

1、问题的提出

数据库必须支持多个用户的多种应用，因而也就必须提供对数据访问的多个入口，也就是说对同一数据的存储要提供多条存取路径。数据库物理设计的任务之一就是确定应建立哪些存取路径。存取路径即索引结构，因为索引结构提供了定位和存取数据的一条路径。存取方法是快速存取数据库中数据的技术。数据库管理系统一般都提供多种存取方法。常用的存取方法有三种：
- 索引方法；
- 簇集方法；
- HASH方法；

B+树索引方法是数据库中经典的索引存取方法，使用最普遍。

2、存取路径的特点

在关系数据库中存取路径具有以下特点：
- 存取路径和数据是分离的，对用户来说是不可见的；
- 存取路径可以由用户建立、删除，也可以由系统动态地建立、删除。例如，在执行查询时DBMS的查询优化器会根据优化策略自动地建立索引，以提高查询效率；
- 存取路径的物理组织通常采用顺序文件、Ｂ+树文件和散列文件结构等等。

## 表结构

innodb 表，数据和索引是一个文件
myisim 表，数据，索引，三个分开

## 为什么直接跳到8

MySQL 5.5 -> MySQL 5
MySQL 5.6 -> MySQL 6
MySQL 5.7 -> MySQL 7
MySQL 8.0 -> MySQL 8

## DQL DML DDL DCL

SQL语言共分为四大类：数据查询语言DQL，数据操纵语言DML，数据定义语言DDL，数据控制语言DCL。

1. 数据查询语言DQL
数据查询语言DQL基本结构是由SELECT子句，FROM子句，WHERE
子句组成的查询块：
SELECT <字段名表>
FROM <表或视图名>
WHERE <查询条件>

2. 数据操纵语言DML data manipulation language
数据操纵语言DML主要有三种形式：
1) 插入：INSERT
2) 更新：UPDATE
3) 删除：DELETE

3. 数据定义语言DDL
数据定义语言DDL用来创建数据库中的各种对象-----表、视图、
索引、同义词、聚簇等如：
CREATE TABLE/VIEW/INDEX/SYN/CLUSTER
| | | | |
表 视图 索引 同义词 簇

DDL操作是隐性提交的！不能rollback 

4. 数据控制语言DCL
数据控制语言DCL用来授予或回收访问数据库的某种特权，并控制
数据库操纵事务发生的时间及效果，对数据库实行监视等。如：
1) GRANT：授权。

2) ROLLBACK [WORK] TO [SAVEPOINT]：回退到某一点。
回滚---ROLLBACK
回滚命令使数据库状态回到上次最后提交的状态。其格式为：
SQL>ROLLBACK;

3) COMMIT [WORK]：提交。
在数据库的插入、删除和修改操作时，只有当事务在提交到数据
库时才算完成。在事务提交前，只有操作数据库的这个人才能有权看
到所做的事情，别人只有在最后提交完成后才可以看到(不是绝对的，看当前的事务隔离级别)。
提交数据有三种类型：显式提交、隐式提交及自动提交。下面分
别说明这三种类型。


(1) 显式提交
用COMMIT命令直接完成的提交为显式提交。其格式为：
SQL>COMMIT；

(2) 隐式提交
用SQL命令间接完成的提交为隐式提交。这些命令是：
ALTER，AUDIT，COMMENT，CONNECT，CREATE，DISCONNECT，DROP，
EXIT，GRANT，NOAUDIT，QUIT，REVOKE，RENAME。

(3) 自动提交
若把AUTOCOMMIT设置为ON，则在插入、修改、删除语句执行后，
系统将自动进行提交，这就是自动提交。其格式为：
SQL>SET AUTOCOMMIT ON；

## 事务隔离级别

SELECT @@tx_isolation 查看数据库隔离级别，版本不一致命令也不同，8.0 为 select @@transaction_isolation

start transaction;
update abc ts set ts.name='abc' where ts.id='1';

ACID 原子性 一致性 隔离性 持久性

事务隔离级别	               脏读	 不可重复读	幻读
读未提交（read-uncommitted）	是	  是	   是
不可重复读(读已提交)（read-committed）	否	  是	   是
可重复读（repeatable-read）	   否	  否	   是
串行化（serializable）	       否	  否	   否

理解事务隔离级别产生的问题: 脏读，不可重复读，幻读

并发带来的问题: 丢失更新

幻读举例，MySQL8.0 默认事务隔离级别，不可重复读

开启两个会话

假设表是abc，有两条数据，表的字段是id name

会话1
start transaction;
select * from abc where id=5;

会话2
start transaction;
insert into abc (name) values ('zxc');
commit;

会话1
mysql> insert into abc (id,name) values (3,'zxc');
ERROR 1062 (23000): Duplicate entry '3' for key 'PRIMARY'
mysql> select * from abc where id = 3;
Empty set (0.00 sec)

出现了幻读，会话2的数据已经插入了，而会话1读取不到（都要在事务中才有这种效果，并且数据库隔离级别不是串行化）

## 事务流程与日志模块

通过日志模块来支持事务：binlog（归档日志）和redo log（重做日志）

## 三范式和反范式

1. 第一范式
确保数据表中每列（字段）的原子性。
如果数据表中每个字段都是不可再分的最小数据单元，则满足第一范式。
例如：user用户表，包含字段id,username,password

2. 第二范式
在第一范式的基础上更进一步，目标是确保表中的每列都和主键相关。
如果一个关系满足第一范式，并且除了主键之外的其他列，都依赖于该主键，则满足第二范式。
例如：一个用户只有一种角色，而一个角色对应多个用户。则可以按如下方式建立数据表关系，使其满足第二范式。
user用户表，字段id,username,password,role_id
role角色表，字段id,name
用户表通过角色id（role_id）来关联角色表

3. 第三范式
在第二范式的基础上更进一步，目标是确保表中的列都和主键直接相关，而不是间接相关。
例如：一个用户可以对应多个角色，一个角色也可以对应多个用户。则可以按如下方式建立数据表关系，使其满足第三范式。
user用户表，字段id,username,password
role角色表，字段id,name
user_role用户-角色中间表，id,user_id,role_id
像这样，通过第三张表（中间表）来建立用户表和角色表之间的关系，同时又符合范式化的原则，就可以称为第三范式。

4. 反范式化
反范式化指的是通过增加冗余或重复的数据来提高数据库的读性能。
例如：在上例中的user_role用户-角色中间表增加字段role_name。
反范式化可以减少关联查询时，join表的次数。

## InnoDB 架构线程

采用的是多线程模型，主要包括:

1. Master Thread

核心的后台线程，主要负责把缓冲池的数据异步刷新到磁盘，保证数据一致性

2. IO Thread

存储引擎使用大量的异步IO处理IO请求，IO Thread主要负责这些IO请求的回调

又可以细分到 write read insert buffer log 4中 IO Thread，线程数量不是单一的，读写线程比较多，版本演进也发生了变化

3. Purge Thread

事务被提交后，使用到的undolog可能不再需要，需要Purge Thread来回收已经使用过的undo页。随着版本的演进也发生了变化

4. Page Cleaner Thread

用来处理脏页刷新的，减轻Master Thread的压力

## 聚蔟索引和非聚蔟索引

或者叫聚集索引也行

非聚集索引索引项顺序存储，但索引项对应的内容却是随机存储的；

表中主键id是该表的聚集索引、name为非聚集索引；
表中的每行数据都是按照聚集索引id排序存储的；
比如要查找name='Arla'和name='Arle'的两个同学，他们在name索引表中位置可能是相邻的，但是实际存储位置可能差的很远(B+树存储)
name索引表节点按照name排序，检索的是每一行数据的`主键`
聚集索引表按照主键id排序，检索的是每一行数据的`真实内容`

也就是说查询name='Arle'的记录时，首相通过name索引表查找到Arle的主键id（可能有多个主键id，因为有重名的同学），再根据主键id的聚集索引找到相应的行记录；

聚集索引一般是表中的主键索引，如果表中没有显示指定主键，则会选择表中的第一个不允许为NULL的唯一索引，如果还是没有的话，就采用Innodb存储引擎为每行数据内置的6字节ROWID作为聚集索引

每张表只有一个聚集索引，因为聚集索引在精确查找和范围查找方面良好的性能表现（相比于普通索引和全表扫描），聚集索引就显得弥足珍贵，聚集索引选择还是要慎重的（一般不会让没有语义的自增id充当聚集索引）

sql举例，1用了主键索引，2用了非聚蔟索引。1就是顺序查找，然后直接得到数据。2就name找id，id可能是离散存储不是顺序的，然后通过id取到数据。总体来说多了一层IO，也就是回表，而且不能顺序读取，也会有一定的性能损坏

（1）select * from student where id >5000 and id <20000;

（2）select * from student where name > 'Alie' and name < 'John';

## 疑问

悲观锁用for_update
乐观锁

锁

共享锁
排它锁

意向共享锁
意向排它锁

间隙锁
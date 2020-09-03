---
weight: 1
title: "mybatis 配置使用总结"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "mybatis 配置使用总结"
resources:
- name: "base-image"
  src: "base-image.jpg"

tags: [Java, Note]
categories: [Java]

lightgallery: true

toc:
  auto: false
---

mybatis 学习笔记和基础概念，在spring boot中的使用配置总结

<!-- more -->

## JDBC 配置参数

官方地址 https://dev.mysql.com/doc/connector-j/5.1/en/connector-j-reference-configuration-properties.html

发现很多人都是抄一样的，有很多参数配置了是不对或者无效的，上面是官方参数说明地址，很详细，全文搜索一下你就知道各个参数的作用了

## 基础

总共有两种使用方式：1. 无配置文件注解版，这种模式就不需要XML文件了；2. 使用传统的XML文件

2 类型的使用流程总结

流程：

1. 添加依赖

2. 配置spring boot

    整体概述：
    配entity，是一个包，包含实体类
    配mapper，映射文件
    mybatis设置，例如map-underscore-to-camel-case-repository 开启驼峰功能，实体类的驼峰字段和数据的下划线风格做映射 （不过一般自己定义resultMap来处理）
    src：要有一个实体类，一个DAO接口类(mapper类) -> 由XML做映射


Spring Boot 会自动加载 spring.datasource.* 相关配置，数据源就会自动注入到 sqlSessionFactory 中，sqlSessionFactory 会自动注入到 Mapper 中。
在启动类中添加对 mapper 包扫描 `@MapperScan`，指定mapper类

@MapperScan 和 @Mapper，@Mapper注解表明这个类是一个映射类，要对应到xml中，生成一个实体类，这样每个映射类都要加@Mapper，所以一般用@MapperScan指明扫描路径
省去了@Mapper注解

spring boot配置文件

```s
mybatis:
  mapper-locations: classpath:/mybatis/mapper/*.xml
  config-location:  classpath:/mybatis/config/mybatis-config.xml
```

配置包括，mapper映射文件，和mybatis配置文件，配置文件是和数据库相关的。这里都是xml相关，mapper映射文件，映射会有多个，映射是接口类的装配，在映射文件中，必须`<mapper namespace="com.yukong.chapter5.repository.UserMapper">`的形式指明接口类是哪一个。

在mybatis的xml配置文件中

```s
<typeAliases>
    <package name="com.yukong.chapter4.entity"/>
</typeAliases>
```

这个指明了实体类的包路径，这样在映射xml中就不用写全名了。mybatis配置文件不是必须的，不显示的指定配置参数，那就使用默认的。

接下来就是src源文件了，包括entity实体类(或者取名为model)包，和mapper接口类包，实体类描述了表的字段，接口类方法描述了对表的操作CURD，具体的操作实现由xml映射文件来实现。

至此，整个配置流程就完成了，需要手动创建数据库，然后就可以使用了。

操作实例：

```java
@Autowired
    private UserMapper userMapper;

@Test
public void save() {
    User user = new User();
    user.setUsername("zzzz");
    user.setPassword("bbbb");
    user.setSex(1);
    user.setAge(18);
    Assert.assertEquals(1, userMapper.save(user));
}
```

通过注解自动装配userMapper，然后给实体类实例赋值，调用userMapper的save方法保存数据

## xml传递参数

接口的实现由xml来编写，接口传递参数可以由xml接收到，xml中用 `#{}` 的形式引用参数

1. 单一参数

```
接口定义参数 String id

在 xml 中
where ID = #{id,jdbcType=VARCHAR}

接口定义参数是对象 Object User

在 xml 中
values (#{id,jdbcType=VARCHAR}, #{name,jdbcType=VARCHAR}, #{code,jdbcType=VARCHAR})

直接访问对象的属性就行了，对象要是一个Bean
```

要指明parameterType，即参数类型，有些资料又说不需要，可能和版本有关

## xml标签

1. select

select – 书写查询sql语句
select中的几个属性说明：
id属性：当前名称空间下的statement的唯一标识。必须。要求id和mapper接口中的方法的名字一致。
resultType：将结果集映射为java的对象类型(记得用完整路径该对象)。必须（和 resultMap 二选一）
parameterType：传入参数类型。可以省略

2. resultMap

MyBatis中在查询进行select映射的时候，返回类型可以用resultType，也可以用resultMap，
resultType是直接表示返回类型的，而resultMap则是对外部ResultMap的引用，但是resultType跟resultMap不能同时存在

```xml
<resultMap id="UserWithRoleMap" type="com.zoctan.api.model.User" extends="UserMap">
    <result column="role_id" jdbcType="BIGINT" property="roleId"/>
    <result column="role_name" jdbcType="VARCHAR" property="roleName"/>
</resultMap>
```
resultMap 通过extends继承另一个resultMap

## 相关注解

注解多为使用纯Java代码映射的形式，做法也很简单
1. 定义实体类
2. 写接口，接口要有注解，方法也要有注解，这样就ok了
参考: https://blog.csdn.net/qq_33238935/article/details/85336429

## mybatis扩展

mybatis是在国内非常通用的MySQL，有很多延伸扩展

### mybatis-generator

简称MBG，是mybatis的逆向工程，由数据库表创建相关代码

---
weight: 4400
title: "mybatis 配置使用总结"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "mybatis 配置使用总结"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Java, Note]
categories: [Java-MicroServices]

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

## mybatis-plus

```xml
<!--数据库依赖-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.1</version>
</dependency>
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.3.1</version>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.21</version>
</dependency>
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.3.0</version>
</dependency>
```

查询，使用lambda

```java
LambdaQueryWrapper<EosPaasMonitor> eq = Wrappers.<EosPaasMonitor>lambdaQuery()
                .eq(EosPaasMonitor::getProject, project)
                .eq(EosPaasMonitor::getName, name);
        EosPaasMonitor eosPaasMonitor = eosPaasMonitorDao.selectOne(eq);

eosPaasMonitorDao.selectOne(Wrappers.<EosPaasMonitor>lambdaQuery().select(EosPaasMonitor::getProject).eq(EosPaasMonitor::getName, name))
```

常规用法

```java
QueryWrapper<EosPaasRedis> wrapper = new QueryWrapper<>();
            wrapper.eq("project", project)
                    .eq("name", name)
                    .eq("delete_flag", Constants.NORMAL);
            EosPaasRedis eosPaasRedis = eosPaasRedisDao.selectOne(wrapper);
```

更新数据，根据指定条件查出后更新

```java
UpdateWrapper<EosPaasPort> updateWrapper = new UpdateWrapper<>();
                updateWrapper
                        .set("is_used", Constants.PORT_USE)
                        .eq("id", eosPaasPortItem.getId());
                eosPaasPortDao.update(eosPaasPortItem, updateWrapper);
```

使用Map作为查询条件，查询类似

```java
Map<String, Object> map = new HashMap<>();
        map.put("project", rabbitmqConfigDTO.getProject());
        map.put("name", rabbitmqConfigDTO.getName());
        eosPaasMonitorDao.deleteByMap(map);
```

lambda查询，如果字段不存在则查询条件不生效

```java
queryWrapper.lambda()
        .eq(ObjectUtil.isNotEmpty(template.getCode()), MsgTemplate::getCode, template.getCode())
        .eq(ObjectUtil.isNotEmpty(template.getType()), MsgTemplate::getType, template.getType())
        .orderByAsc(MsgTemplate::getSort); 
```

使用分页，引入分页依赖，注入分页拦截Bean

```java
@Bean
public PaginationInterceptor paginationInterceptor() {
    return new PaginationInterceptor();
}
```

```java
int current = 2;//当前页
int row = 1;// 每页条数
Page<EosPaasRedis> page = new Page<>(current, row);
QueryWrapper<EosPaasRedis> wrapper = new QueryWrapper<>();
// wrapper.like("password", "8");
Page<EosPaasRedis> selectPage = eosPaasRedisDao.selectPage(page, wrapper);
System.out.println("selectPage.getSize() = " + selectPage.getSize());
System.out.println("selectPage.getTotal() = " + selectPage.getTotal());
System.out.println("selectPage.getOrders() = " + selectPage.getOrders());
System.out.println("selectPage.getCurrent() = " + selectPage.getCurrent());
System.out.println("selectPage.getRecords() = " + selectPage.getRecords());
```

## mybatis中的collection如何使用dao的传值

针对一对多的查询场景，我们可以使用collection去组合数据，这里没有使用外连接，而是使用子查询的方式

这里我们用模型和模型组的概念来举例，多个模型组合成一个模型组，所以是 `模型组` **一对多** `模型`

关键点：

1. `<collection property="models" column="{id=id,search_name=search_name}" select="selectModelList2">` property 定义的属性在
ModelListResp中要有，并且是list的，然后我们通过column传值，column可以把当前表的字段或者通过DAO过来的值传递给子查询的 select
这里就是把 ModelListResp 这个模型的 id 传递 给 `select id="selectModelList2" resultType="ModelEntity"` ，就可以在里面拿到id

2. 在子查询中通过id过滤数据，差不多类似做到了外连接的 on 条件 (子查询相当于需要这条数据每次去数据库查询，比较耗费性能)

3. 像一些DAO传递过来的参数，我们通过 `#{name} as search_name`的方式，把变量映射到结果集中，然后通过 collection 的 column 传递 过去
`{id=id,search_name=search_name}` 的写法是数据库字段和模型字段的映射关系，第一个id是数据库的字段，第二id是模型的字段，都一样可以写成 id, search_name

```xml
<resultMap id="ListResultMap" type="ModelListResp">
    <id column="id" jdbcType="VARCHAR" property="id"/>
    <result column="name" jdbcType="VARCHAR" property="name"/>
    <result column="code" jdbcType="VARCHAR" property="code"/>
    <result column="color" jdbcType="VARCHAR" property="color"/>
    <collection property="models" column="{id=id,search_name=search_name}" select="selectModelList2">
    </collection>
</resultMap>

<select id="selectModelList" parameterType="string" resultMap="ListResultMap">
    SELECT *,#{name} as search_name  FROM `model_group`
</select>

<select id="selectModelList2"
        resultType="ModelEntity">
    SELECT * FROM `model` WHERE model_group_id=#{id}
    <if test="search_name !=null and search_name != ''">AND `name` like concat('%', #{search_name}, '%')</if>
</select>
```

参考：

https://blog.csdn.net/qq_39247788/article/details/105380249

https://www.cnblogs.com/dreamyoung/p/12466656.html

https://www.cnblogs.com/yshyee/p/12713679.html

https://my.oschina.net/u/4466912/blog/4679187

https://www.cnblogs.com/wllbdml/p/11455143.html

https://my.oschina.net/u/2427561/blog/3138316

## mybatis-plus 分页

mapper List<User> getUserList(Page<User> page);

接口：Page<User> getUserList(Page<User> page);
实现：page.setRecords(this.baseMapper.getUserList(page));

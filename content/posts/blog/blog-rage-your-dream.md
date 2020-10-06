---
weight: 6600
title: "Rage Your Dream"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Rage Your Dream"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Technology]
categories: [Clear-Mind]

lightgallery: true

toc:
  auto: false
---

Rage Your Dream 项目开发笔记

<!-- more -->

## 流程

1. 初步实现
2. 风格确定
3. 代码审查

## 知识补充

- Vue.prototype

当你在main.js里声明了Vue.prototype.a = 1后，因为你的每一个vue组件都是一个Vue对象的实例，所以即使你没有在组件内部使用data(){return{……}}声明a，你依然可以在组件中通过this.a来访问。
当然，你也可以在组件中添加一个变量a，这时你访问的就是你在组件中添加的a，而不再是之前在原型中添加的a了，当然你对组件的a继续修改即不会影响原型中的a和其他组建中的a，就类似于下面这段代码（Form是一个自定义对象类型，Vue也可以看作一个自定义对象类型，而每个.vue文件就是一个对象的实例）

kafaka、keeplive、docker、influxdb、zookeerper，hive、kfka、storm、promethues、pinpoint、grafana、elk、hdfs、kibana、nexus、harbor、grafana、ambari、cerebro、otter、Weave Scope

- mock请求

mock的请求不会在浏览器被记录的，它是一种js代码控制的行为，当然不知道是不是有其它办法，总之用了mock需要调试的话，使用console
mock的初始化要先，保证被mock记录的URL都能被拦截到

## 问题记录

1. 关于集成通用tk.mybatis

旧版本是不会在控制台有告警的，但是无法解析驼峰命名字段，使用注解也不行，升级了新版本后，字段解析正常，控制台会有告警
依赖的使用也有争议
```xml
<!--mapper-->
<!-- https://mvnrepository.com/artifact/tk.mybatis/mapper -->
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper</artifactId>
    <version>4.1.5</version>
</dependency>

<!-- https://mvnrepository.com/artifact/tk.mybatis/mapper-spring-boot-starter -->
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper-spring-boot-starter</artifactId>
    <version>2.1.5</version>
</dependency>
```
有关该问题的描述记录，在readme文件中查找

## 规范

注释：

```
<!-- 用户管理视图
@author vanliuzhi
@create 2019-08-15
-->
```

```
/**
 * @description 系统管理，用户管理API
 * @author vanliuzhi
 * @create 2019-08-15
 */
```

数据：
接口数据走json，格式如下，后端也统一成该风格，数据封装在result中

```
code: 0
message: ""
result: {}
timestamp: 1565707913336
```

## vue-cli3.0

全新的脚手架

### vue-cli3.0 环境变量与模式

vue-cli3.0移除了配置文件目录：config和build文件夹(最外层已经没有这两个文件夹了)

总共提供了四种方式来制定环境变量：

1. 在根目录添加.env文件，配置所有情况下都会用到的配置（不知道这个存在的意义，所有的都需要的也就不需要配置了吧）。
2. 在根目录添加.env.local 文件，配置所有情况下都会用到的配置，与.env的区别是只会在本地，该文件不会被git跟踪。
3. 在根目录添加.env.[mode] 文件，配置对应某个模式下的配置,比如：.env.development来配置开发环境的配置。
4. 在根目录添加.env.[mode].local文件，配置对应某个模式下的配置,与.env.[mode]的区别也只是会在本地生效，该文件不会被git跟踪。

修改环境变量后，记得手动重启，让环境变量重新写入系统上下文

## layouts布局组件

根目录layouts文件下存放了所有布局组件，路由可以配置使用的组件，所以一些页面默认是带了布局的

- UserLayout

该组件用于`'/user'`路由下，即登录注册等

## 数据管理

暂时使用sql文件的方式，对表的改动都基于sql文件

## main.js

程序主入口，一般会导入各种配置文件，前端的路由和后端的接口URL不是一个概念，路由的URL跳转不会去和后端交互

### permission

权限控制部分，通过路由守卫来完成，根据有无token来决定下一步。目前绝大部分项目的思路都很类似，比较的工程化，最好也参照这个流程来

1. 有

如果`to.path`是登录页面，一般跳转到首页去，否则，获取用户信息等操作

2. 无

白名单直接登录，否则跳转到登录页面，登录页面表单接受用户名和密码，调用`store`中`user`的`actions`中的`Login`，在Login中去请求真正的登录接口

## table组件

table组件使用总结

```html
<a-table :columns="columns" :dataSource="data" :rowKey="record => record.id"></a-table>
```

- columns配置列表数据，dataSource为数据源
需要配置唯一key，columns和dataSource都需要，columns 配置dataIndex或key，数据源最好包含一个唯一值，在`:rowKey="record => record.id`指定

- columns中的customRender属性，`Function(text, record, index)` 参数分别为当前行的值，当前行数据，行索引，可以用这个对数据做复杂处理，比如把整形状态值变成文本值
JSX语法：可以在customRender中使用JSX语法返回HTML，这样就不需要用scopedSlots在标签中写代码了

- 插槽：并不是很理解(语言本身理解的还不够深入)，不过2.6要废弃这个属性了，说明一下用法（大概的总结就是下面的内容，可以自行测试）
slots: { title: 'customTitle' } 这个是用法是`<span slot="customTitle"><a-icon type="smile-o" /> Name</span>`，title是表格的属性，指明表头名称的，通过插槽可以自定义这个名称，这里自定义就是加了一个icon (总结就是可以自定义一下表格本身的属性，目前已知可以修改标题，还可以filterIcon，自定义 fiter 图标)
scopedSlots: { customRender: 'name' } 一般就这么定义，name就是当前的字段，上面的customTitle是自定义的，然后`<span slot="name" slot-scope="text, record">`或`<span slot="name" slot-scope="name">`，这里的参数对应上面customRender属性，只用一个参数，比如这里的`name`就是当前字段的值 (总结就是可以获取到当前字段和行，索引的数据，方便自定义字段的渲染)

这里的两个属性都是对象的形式，如果要自定义字段，首先就是要知道插槽到底是指向哪一个字段，slots是可以不用配置的，默认名称就是当前字段名，然后配置scopedSlots: { customRender: 'name' }，`<span slot="name" slot-scope="text">{{text}}</span>`，slot-scope才会生效

- 关于表格的对齐，有标题和内容，测试发现对齐属性配置在columns中，对标题和内容都生效，可以全局定义css来覆盖，需要的地方再使用属性来配置

- 分页，框架默认前端分页，分页可以通过配置来自定义，不过这个配置文档描述相当不友好，这个分页依赖组件，个人认为最好的形式是禁用table的分页，自己使用分页组件
总之这一块设计的很不好，应为你用table默认的分页，就要去配置属性，然而配置属性的文档是模糊的，比如onShowSizeChange才是table配置中的每页数目改变的事件回调，而组件中确是showSizeChange，这些不确定的配置导致很多问题，下面是一个配置用例，注意onShowSizeChange，其它属性可参考分页组件(猜错这可能是遗留的BUG)，个人猜测defaultCurrent是组件需要读取的，但是用pageNo也可以，这里新加了两个属性，方便读取数据的时候知道当前页数和条目

总结：存在bug肯定是逃不掉的了，或者说是泛用性很强，但是onShowSizeChange是你看文档看不出来的，ant-vue-pro框架也提供了封装的方法，不过不是所有的数据请求加载都是在页面这一环的，我想更自由一点，只能自己梳理文档

```js
pagination: {
    // defaultCurrent: 1, // 默认的当前页数
    // defaultPageSize: 10, // 默认每页显示数量
    pageNo: 1,
    pageSize: 20,
    showSizeChanger: true, // 显示可改变每页数量
    showQuickJumper: true, // 显示页数跳转
    pageSizeOptions: ['10', '20', '30', '50', '100'], // 每页数量选项
    showTotal: total => `共 ${total} 条数据`, // 显示总数
    onShowSizeChange: (current, pageSize) => {
        // console.log(current) // current 当前页
        this.pagination.pageSize = pageSize
        this.pagination.pageNo = current
        this.getTableData(current, pageSize)
    }, // 改变每页数量时更新显示，并请求接口
    onChange: (page, pageSize) => {
        this.pagination.pageSize = pageSize
        this.pagination.pageNo = page
        this.getTableData(page, pageSize)
    }, // 点击页码事件
    total: 0 // 总条数
}
```

后端返回分页数据格式

```js
{
    data: []
    pageNo: 3
    pageSize: 10
    totalCount: 23
    totalPage: 3
}
```

## form 表单组件

表单组件有其独特的用法，不再是单纯的双向绑定，自己提交参数的形式，使用form-create后，可以自动的收集和校验数据，不使用此方式就用传统的双向绑定
双向绑定的方式就是常规的用法了，使用新的模式可以使用框架提供的API，建议用此模式

可能会遇到的错误`Warning: You cannot set a form field before rendering a field associated with the value.`，出现这种情况一般是dom还没渲染，或者是字段没注册，没被组件接管不能用API

- 数据赋值

## 后端技术栈

grafana  可视化监控界面

OpenResty Nginx配合lua的高性能web服务器

elasticsearch-head  是用于监控 Elasticsearch 状态的客户端插件，使用node运行

kafka 

zookeeper

logstash

filebeat

elasticsearch

Redis-Desktop-Manager

redis

ZooInspector zk数据查看工具

## 后端框架版本

spring boot 2.1.12.RELEASE

spring cloud Greenwich.SR5

## 微服务带来的挑战

服务部署的挑战
服务伸缩的挑战
服务高可用
服务容错
依赖关系
服务监控

## DevOps

DevOps（Development和Operations的组合词）是一组过程、方法与系统的统称，用于促进开发（应用程序/软件工程）、技术运营和质量保障（QA）部门之间的沟通、协作与整合。

## 权限设计

RBAC设计思想

基于角色的访问控制（Role-Based Access Control）
有两种正在实践中使用的RBAC访问控制方式：隐式(模糊)的方式和显示(明确)的方式。

隐式：假如用于属于管理员，在对账户操作的时候，就去判断用于是否是管理员角色，因为在常规的认知中，管理员一般都是能操作账号的，但是实际是否分配的这样的权限是未知的，这就是隐式的，是一种假设。
显示：就是明确的权限判断

### 设计实现

1. 用户管理
    - 新建用户
    - 分配角色
    - 删除用户
    - 获取用户列表
2. 角色管理
    - 新建角色
    - 分配权限
    - 获取角色列表
3. 权限管理
    - 新建权限 （把权限完全最小粒度化，只有权限名，不做更多的关联，通过更为详细的权限命名来扩展）
    - 获取权限列表
    - 删除权限


### 表结构

1. user 用户表
2. role 角色表
3. permission 权限表
4. user_role 用户和角色关联表
5. role_permission 角色和权限关联表

## 数据库与ORM

`主要分析，数据配置，与orm框架结合，连接池，多数据源配置等如何使用`

### mybatis

数据库ORM使用mybatis，通常情况，引入mybatis的spring boot风格的依赖，还有MySQL connect依赖就会自动装配，相关bean就会注入容器，此时如果不配置MySQL，是会有错误日志的

```xml
<!--mybatis Spring boot 支持-->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

数据源配置比较简单，这里使用默认的连接池配置

```yaml
spring:
  datasource:
    url: jdbc:mysql://${MYSQL_HOST:localhost}:${MYSQL_PORT:5506}/${DB_AUTH:rrd_store}?useUnicode=true&allowMultiQueries=true&serverTimezone=Asia/Shanghai&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull&useSSL=false
    username: ${MYSQL_USERNAME:root}
    password: ${MYSQL_PASSWORD:root123}
    type: com.zaxxer.hikari.HikariDataSource
    driver-class-name: com.mysql.jdbc.Driver
```

### 连接池

当然这样用的太基础了，实际还要加上分页插件，连接池等配置，还有数据库性能监控，或者使用mybatis-plus增强功能

Springboot默认的连接池Hikari，实际情况，会考虑使用Druid

依赖

```xml
<dependency>
   <groupId>com.alibaba</groupId>
   <artifactId>druid-spring-boot-starter</artifactId>
   <version>1.1.10</version>
</dependency>
```

还有一种依赖是这个，使用这种的话要自己创建配置类，所以还是用spring-boot风格的吧，就是上面一种

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.3</version>
</dependency>
```

```yaml
spring:
  datasource:
    druid:
      initial-size: 5
      min-idle: 5
      max-active: 20
      # 省略...
```

也就是把参数写在druid下面，对于type的配置，可以不写，引入依赖就用Druid，具体需要源码验证，不放心就显示配置

druid自带了监控页面，以及防止sql注入配置，多数据源配置，具体看文档

druid监控页面 去广告https://blog.csdn.net/chouya3495/article/details/100661794

### mybatis-plus

把依赖替换，注意mybatis-plus的配置名称和mybatis不同，但是配置的内容大同小异

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>      
    <version>3.0.7</version>
</dependency>
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus</artifactId>
    <version>3.0.7</version>
</dependency>
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.0.7</version>
</dependency>
```

### 多数据

多数据源指的是使用多个数据库

不知道你有没有注意到一点，假设我的数据源配置是 jdbc:mysql://127.0.0.1:9906/rrd_store
我在mybatis中是 from `honor-user-center`.sys_user，能正常访问到数据库

rrd_store 和 honor-user-center 都是 9906连接下的两个数据库，也就是说连接上后，是可以去访问不同的数据库的

但是假如我有多个连接，比较9906，7706，做读写分离，那么上面的小技巧就没用了，这种情况就是多数据源，需要我们去连接不同地址的数据库

具体做法大致有2种，涉及到很多底层的东西，先知道一个概念，就是数据源对象

1. 配置多个连接对象，每个连接对象都是不同的配置，直接和mybatis映射绑定

2. 配置一个连接对象，但是里面有多个数据源对象，通过注解切换数据源对象




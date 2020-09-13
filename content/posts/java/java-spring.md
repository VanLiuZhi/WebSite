---
weight: 1000
title: "Spring"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Spring"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Java, Note]
categories: [Java]

lightgallery: true

toc:
  auto: false
---

Spring 基础概念学习总结

<!-- more -->

## 应用服务器

SpringBoot已经内置了Servlet容器，包括tomcat，jetty，Undertow

## Spring基础

IoC 控制反转(Inversion of Control，缩写为IoC)：Spring所倡导的开发方式就是如此，所有的类都会在spring容器中登记，告诉spring你是个什么东西，你需要什么东西，然后spring会在系统运行到适当的时候，把你要的东西主动给你，同时也把你交给其他需要你的东西。所有的类的创建、销毁都由spring来控制，也就是说控制对象生存周期的不再是引用它的对象，而是spring。对于某个具体的对象而言，以前是它控制其他对象，现在是所有对象都被spring控制，所以这叫控制反转。
IoC的一个重点是在系统运行中，动态的向某个对象提供它所需要的其他对象。这一点是通过DI（Dependency Injection，依赖注入）来实现的。比如对象A需要操作数据库，以前我们总是要在A中自己编写代码来获得一个Connection对象，有了 spring我们就只需要告诉spring，A中需要一个Connection，至于这个Connection怎么构造，何时构造，A不需要知道。在系统运行时，spring会在适当的时候制造一个Connection，然后像打针一样，注射到A当中，这样就完成了对各个对象之间关系的控制。A需要依赖 Connection才能正常运行，而这个Connection是由spring注入到A中的，依赖注入的名字就这么来的。
`Spring的IoC容器就是实现控制反转和依赖注入`

- JavaBean
JavaBean 是一种JAVA语言写成的可重用组件。为写成JavaBean，类必须是具体的和公共的，并且具有无参数的构造器。JavaBean 通过提供符合一致性设计模式的公共方法将内部域暴露成员属性，set和get方法获取。众所周知，属性名称符合这种模式，其他Java 类可以通过自省机制(反射机制)发现和操作这些JavaBean 的属性。

- POJO
POJO（Plain Ordinary Java Object）简单的Java对象，实际就是普通JavaBeans，是为了避免和EJB混淆所创造的简称。
使用POJO名称是为了避免和EJB混淆起来, 而且简称比较直接. 其中有一些属性及其getter setter方法的类,没有业务逻辑，有时可以作为VO(value -object)或dto(Data Transform Object)来使用.当然,如果你有一个简单的运算属性也是可以的,但不允许有业务方法,也不能携带有connection之类的方法

- RMI
RMI(Remote Method Invocation，远程方法调用)是用Java在JDK1.2中实现的，它大大增强了Java开发分布式应用的能力。Java作为一种风靡一时的网络开发语言，其巨大的威力就体现在它强大的开发分布式网络应用的能力上，而RMI就是开发百分之百纯Java的网络分布式应用系统的核心解决方案之一。其实它可以被看作是RPC的Java版本。但是传统RPC并不能很好地应用于分布式对象系统。而Java RMI 则支持存储于不同地址空间的程序级对象之间彼此进行通信，实现远程对象之间的无缝远程调用。

- impl
全称implement是用来实现接口的

## Starters

Starters是一系列极其方便的依赖描述，通过在你的项目中包含这些starter，你可以一站式获得你所需要的服务，而无需像以往那样copy各种示例配置及代码，然后调试，真正做到开箱即用；比如你想使用Spring JPA进行数据操作，只需要在你的项目依赖中引入spring-boot-starter-data-jpa即可

## Spring properties 配置文件

通过配置文件，可以修改框架配置，或为框架提供数据，比如生成一个随机数，和框架配合后可以很方便的获取一个随机数，而不需要写更多的代码

在配置中这么写:
shiyanlou.springboot=Hello_shiyanlou

在控制器对象中通过注解定义shiyanlou，变量shiyanlou的值就是配置中的值Hello_shiyanlou

    @Value("${shiyanlou.springboot}")
    private String shiyanlou;


## 模板引擎

freemarker
Thymeleaf
velocity

## 关于controller层

有两种风格的代码，主要是注解的运用，具体看源码注释

1. 风格一
- 对主类使用@RequestMapping("/movies") @RestController
- 对方法使用@GetMapping("/users/{id}")

2. 风格二
- 对主类使用@RequestMapping("/brand") @Controller
- 对方法使用@RequestMapping(value = "/delete/{id}", method = RequestMethod.GET) @ResponseBody

使用了RestController，就包含了Controller，ResponseBody
使用了GetMapping，包含了RequestMapping


## AOP 在 spring中的运用

OOP面向对象，允许开发者定义纵向的关系，但并适用于定义横向的关系，导致了大量代码的重复，而不利于各个模块的重用。
AOP，一般称为面向切面，作为面向对象的一种补充，用于将那些与业务无关，但却对多个对象产生影响的公共行为和逻辑，抽取并封装为一个可重用的模块，这个模块被命名为“切面”（Aspect），减少系统中的重复代码，降低了模块间的耦合度，同时提高了系统的可维护性。

可用于权限认证、日志、事务处理。

AOP实现的关键在于 代理模式，AOP代理主要分为静态代理和动态代理。静态代理的代表为AspectJ；动态代理则以Spring AOP为代表。

（1）AspectJ是静态代理的增强，所谓静态代理，就是AOP框架会在编译阶段生成AOP代理类，因此也称为编译时增强，他会在编译阶段将AspectJ(切面)织入到Java字节码中，运行的时候就是增强之后的AOP对象。
（2）Spring AOP使用的动态代理，所谓的动态代理就是说AOP框架不会去修改字节码，而是每次运行时在内存中临时为方法生成一个AOP对象，这个AOP对象包含了目标对象的全部方法，并且在特定的切点做了增强处理，并回调原对象的方法。

Spring AOP中的动态代理主要有两种方式，JDK动态代理和CGLIB动态代理：

1. JDK动态代理只提供接口的代理，不支持类的代理。核心InvocationHandler接口和Proxy类，InvocationHandler通过invoke()方法反射来调用目标类中的代码，动态地将横切逻辑和业务编织在一起；接着，Proxy利用 InvocationHandler动态创建一个符合某一接口的的实例, 生成目标类的代理对象。

2. 如果代理类没有实现 InvocationHandler 接口，那么Spring AOP会选择使用CGLIB来动态代理目标类。CGLIB（Code Generation Library），是一个代码生成的类库，可以在运行时动态的生成指定类的一个子类对象，并覆盖其中特定方法并添加增强代码，从而实现AOP。CGLIB是通过继承的方式做的动态代理，因此如果某个类被标记为final，那么它是无法使用CGLIB做动态代理的。

（3）静态代理与动态代理区别在于生成AOP代理对象的时机不同，相对来说AspectJ的静态代理方式具有更好的性能，但是AspectJ需要特定的编译器进行处理，而Spring AOP则无需特定的编译器处理。
InvocationHandler 的 invoke(Objectproxy,Methodmethod,Object[] args)：proxy是最终生成的代理实例;method 是被代理目标实例的某个具体方法;args 是被代理目标实例某个方法的具体入参, 在方法反射调用时使用。

面向切面编程

在Spring中的AOP是依靠动态代理来实现切面编程的.
而这两者又是有区别的.

JDK是基于反射机制,生成一个实现代理接口的匿名类,然后重写方法,实现方法的增强.
它生成类的速度很快,但是运行时因为是基于反射,调用后续的类操作会很慢.
而且他是只能针对接口编程的.

CGLIB是基于继承机制,继承被代理类,所以方法不要声明为final,然后重写父类方法达到增强了类的作用.
它底层是基于asm第三方框架，是对代理对象类的class文件加载进来,通过修改其字节码生成子类来处理.
生成类的速度慢,但是后续执行类的操作时候很快.
可以针对类和接口。

因为jdk是基于反射,CGLIB是基于字节码.所以性能上会有差异.
在老版本CGLIB的速度是JDK速度的10倍左右,但是CGLIB启动类比JDK慢8倍左右,但是实际上JDK的速度在版本升级的时候每次都提高很多性能,而CGLIB仍止步不前.

在对JDK动态代理与CGlib动态代理的代码实验中看，1W次执行下，JDK7及8的动态代理性能比CGlib要好20%左右。

具体应用:
如果目标对象实现了接口,默认情况下是采用JDK动态实现AOP
如果目标对象没有实现接口,必须采用CGLIB库.

## 注解

注解说明

### @Resource和@Autowired

共同点

@Resource和@Autowired都可以作为注入属性的修饰，在接口仅有单一实现类时，两个注解的修饰效果相同，可以互相替换，不影响使用。

不同点，在有多个实现的时候使用要注意

@Resource是Java自己的注解，@Resource有两个属性是比较重要的，分是name和type；Spring将@Resource注解的name属性解析为bean的名字，而type属性则解析为bean的类型。所以如果使用name属性，则使用byName的自动注入策略，而使用type属性时则使用byType自动注入策略。如果既不指定name也不指定type属性，这时将通过反射机制使用byName自动注入策略。
@Autowired是spring的注解，是spring2.5版本引入的，Autowired只根据type进行注入，不会去匹配name。如果涉及到type无法辨别注入对象时，那需要依赖@Qualifier或@Primary注解一起来修饰。

参考原文：https://blog.csdn.net/magi1201/article/details/82590106 

### @SpringBootApplication

@SpringBootApplication注解相当于同时使用@EnableAutoConfiguration、@ComponentScan、@Configurations三个注解  

### @EnableAutoConfiguration

@EnableAutoConfiguration用于打开SpringBoot自动配置，而其余注解为Spring注解，这里不再赘述

### @RestController

@RestController相当于同时使用@Controller和@ResponseBody注解

### @Configuration

@Configuration表示该类为配置类，该注解可以被@ComponentScan扫描到

### @ImportResource

// 通过@ImportResource加载xml配置文件 配置文件放在资源目录下，注解作用于主函数入口
@ImportResource(value = "classpath:config.xml")

### @Transactional

@Transactional 用于事务的注解

## 扩展

如何使用Spring Boot/Spring Cloud 实现微服务应用
spring Cloud是一个基于Spring Boot实现的云应用开发工具，它为基于JVM的云应用开发中的配置管理、服务发现、断路器、智能路由、微代理、控制总线、全局锁、决策竞选、分布式会话和集群状态管理等操作提供了一种简单的开发方式。

Spring Cloud与Dubbo对比
提到Dubbo，我想顺便提下ESB，目前央视新华社也在用ESB来做任务编排，这里先比较下Dubbo和ESB：

ESB（企业数据总线），一般采用集中式转发请求，适合大量异构系统集成，侧重任务的编排，性能问题可通过异构的方式来进行规避，无法支持特别大的并发。
Dubbo（服务注册管理），采用的是分布式调用，注册中心只记录地址信息，然后直连调用，适合并发及压力比较大的情况；其侧重服务的治理，将各个服务颗粒化，各个子业务系统在程序逻辑上完成业务的编排。

回归主题，Spring Cloud和Dubbo又有什么不同那，首先，我们看下有什么相同之处，它们两都具备分布式服务治理相关的功能，都能够提供服务注册、发现、路由、负载均衡等。说到这，Dubbo的功能好像也就这么多了，但是Spring Cloud是提供了一整套企业级分布式云应用的完美解决方案，能够结合Spring Boot，Docker实现快速开发的目的，所以说Dubbo只有Spring Cloud的一部分RPC功能，而且也谈不上谁好谁坏。不过，Dubbo项目现已停止了更新，淘宝内部由hsf替代dubbo，我想这会有更多人倾向Spring Cloud了。

从开发角度上说，Dubbo常与Spring、zookeeper结合，而且实现只是通过xml来配置服务地址、名称、端口，代码的侵入性是很小的，相对Spring Cloud，它的实现需要类注解等，多少具有一定侵入性。

ESB（Enterprise Service Bus）全名：企业服务总线，是SOA架构思想的一种实现思路

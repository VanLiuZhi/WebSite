---
weight: 1000
title: "Spring Cloud 微服务实践"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Spring Cloud 微服务实践"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Java, Note]
categories: [Java]

lightgallery: true

toc:
  auto: false
---

![image](/images/Java/Java-spring.jpg)

## 版本简介

spring cloud的版本比较特殊，它不是常见的版本号形式，而是由英文单词+SR+数字的形式，英文单词由伦敦地铁命名。这种版本命名也不是什么新鲜事，比如英伟达就用物理学家名字命名自己的GPU架构

Spring Cloud版本发布记录可详见：https://github.com/spring-cloud/spring-cloud-release/releases ，从中我们可清晰看到Spring Cloud版本发布的时间及先后顺序

Spring Cloud版本演进计划：https://github.com/spring-cloud/spring-cloud-release/milestones ，从中我们可了解Spring Cloud的版本演进计划，例如计划什么时间点发布什么版本等

## SOA

面向服务的架构（SOA）Service-Oriented Architecture，它将应用程序的不同功能进行拆分，并通过服务之间定义的接口联系起来，一般来说接口定义应该是中立的，即独立于平台和编程语言的，这样使得不同语言的服务能有一种统一的方式进行交互

## 各种组件

`Eureka`: Spring Cloud用来提供服务发现和注册的组件
`Ribbon`: 提供服务负载均衡的组件
`Feign`: 封装服务直接的请求调用，不用指定point，这样服务ip端口变化时仍然能够访问
`Hystrix`: 提供服务熔断, Turbine做集群监控
`Zuul`: 提供服务网关
`Config`: 提供统一服务配置
`Sleuth`: 提供服务调用链，用于排查问题，和ZipKin配合使用，ZipKin提供可视化页面
`Consul`: 是一个服务网格（微服务间的 TCP/IP，负责服务之间的网络调用、限流、熔断和监控）解决方案，它是一个一个分布式的，高度可用的系统，而且开发使用都很简便。它提供了一个功能齐全的控制平面，主要特点是：服务发现、健康检查、键值存储、安全服务通信、多数据中心
`Gateway`: Spring-cloud 的网关服务

## 如何使用多实例

这个也是我从接口api中摸索出来的，如果有更好的方式可以改进

首先服务是支持多实例的，以consul为例，我们可以配置服务名称和实例名称，服务名称通过下面两种方式设置

```
spring.cloud.consul.discovery.service-name  显示设置
spring.application.name  默认
```

服务名称相同的归为同一个服务，然后有实例名称

```
spring.cloud.consul.discovery.instance-id 显示设置，不设置，默认是服务名称加port
```

当我们只有一个服务的时候，实例只有一个，当我们启动多个实例的时候，服务名称相同的归在一起

1. 如何启动多个实例？可以用profiles的形式

```yml
spring:
  application:
    name: rrdtool-store-service-star
  cloud:
    consul:
      host: ${CONSUL_HOST:localhost}
      port: ${CONSUL_PORT:8500}
      discovery:
        instance-id: ${spring.application.name}-${server.port}
---
server:
  port: 8920
spring:
  profiles: node1
---
server:
  port: 8921
spring:
  profiles: node2

```

但是这样会有问题

Consul把InstanceId作为唯一标识，而Spring Cloud Consul默认的InstanceId是 `${spring.application.name}-${server.port}` 。

这样导致的问题是：某个微服务即使有多个实例，只要端口相同，那么Consul上依然只会保留1条数据！也就是说我们如果在不同的机器上用一样的端口，就会有这个问题。要想解决这个问题，只需要让不同实例，拥有不同的InstanceId即可。

解决方案: 再加一个唯一标识，或者自定义实例id，当然这个做法就不推荐了，除非有必要(ConsulAutoRegistration相关代码可以做)

`${spring.cloud.client.ip-address}` 或 `${spring.cloud.client.hostname}`

2. 多个实例有什么用？

在feign请求服务的时候，我们设置的是服务名称，然后负载均衡策略会轮询调用具体的实例

## Eureka

如何使用: 启动一个eureka service，各个微服务通过配置client把自己注册到eureka service

eureka service主要配置，起多个服务实现高可用，注意它是AP架构。下面通过一个工程的不同profiles启动多个service实现高可用，如果是部署在一台主机上，hostname不能少

依赖: spring-cloud-starter-netflix-eureka-client spring-cloud-starter-netflix-eureka-server，在启动类上加上`@EnableEurekaServer`注解

service端配置

```yaml
eureka:
  client:
    # 由于是高可用，所以要同步数据
    # 是否要注册到其他Eureka Server实例
    register-with-eureka: true
    # 是否要从其他Eureka Server实例获取数据
    fetch-registry: true
    service-url:    
      defaultZone: http://localhost:8761/eureka/, http://localhost:8762/eureka/
---
spring:
  profiles: peer1
server:
  port: 8761
eureka:
  instance:
    hostname: peer1
---
spring:
  profiles: peer2
server:
  port: 8762
eureka:
  instance:
    hostname: peer2
```

client端加入服务

```yaml
eureka:
  client:
    service-url:
      # 指定eureka server通信地址，注意/eureka/小尾巴不能少，如果是有多个service，建议配置多个地址
      defaultZone: http://localhost:8761/eureka/
  instance:
    # 是否注册IP到eureka server，如不指定或设为false，那就会注册主机名到eureka server
    prefer-ip-address: true
```

## Ribbon

Ribbon不在本节讨论

依赖: spring-cloud-starter-netfilx-ribbon

可以把ip+端口用服务名来代替，Ribbon可实现精确到目标服务的细粒度配置

Ribbon的负载均衡和Nginx的区别：Nginx是集中式的，把所有请求发到Nginx上，然后分发到服务消费者。ribbon是在客户端做的负载均衡，然后去请求服务消费者。Nginx多了一层客户端发到Nginx的过程

## Feign

1. 依赖: spring-cloud-starter-openfeign

2. 添加注解到启动类上

```java
// 扫描api包里的FeignClient
@EnableFeignClients(basePackages = {CommonConstant.BASE_PACKAGE})
```

4. 编写Feign Client：

```java
@FeignClient(name = "microservice-provider-user")
public interface UserFeignClient {
  @GetMapping("/users/{id}")
  User findById(@PathVariable("id") Long id);
}
```

这样一个Feign Client就完成了，其中，@FeignClient 注解中的microservice-provider-user是想要请求服务的名称，这是用来创建Ribbon Client的（Feign整合了Ribbon）。在本例中，由于使用了Eureka，所以Ribbon会把microservice-provider-user 解析成Eureka Server中的服务。

除此之外，还可使用url属性指定请求的URL（URL可以是完整的URL或主机名），例如@FeignClient(name = "abcde", url = "http://localhost:8000/") 。此时，name可以是任意值，但不可省略，否则应用将无法启动！

5. 通过Feign Client访问服务接口

```java
@RequestMapping("/movies")
@RestController
public class MovieController {
  @Autowired
  private UserFeignClient userFeignClient;

  @GetMapping("/users/{id}")
  public User findById(@PathVariable Long id) {
    return this.userFeignClient.findById(id);
  }
}
```


## 应用容错

什么是容错，就是当服务中的某个服务挂了的时候，允许这种情况，并有对应的补救。如果没有容错机制，那么A请求B，B挂了，但是A还要一直请求B，导致系统大量的资源被占用无法释放

1. 超时

超时是一种比较容易的机制，比如就设置2秒，2秒没返回就释放资源

2. 舱壁模式

先了解一下船舱构造——一般来说，现代的轮船都会分很多舱室，舱室之间用钢板焊死，彼此隔离。这样即使有某个/某些船舱进水，也不会影响其他舱室，浮力够，船不会沉

这种机制在程序上就是分配资源，比如A服务和B服务都只能使用系统资源的20%，这样B服务挂了也就会影响系统20%的资源

或者这样：A调用a，B调用b。AB共享同样的资源，比如线程池，当a挂了，那么B服务就被A拖死了，如果AB使用独立的线程池，那么顶多就是A把自己的线程池占满了，不会影响B

3. 断路器

软件世界的断路器可以这样理解：实时监测应用，如果发现在一定时间内失败次数/失败率达到一定阈值，就“跳闸”，断路器打开——此时，请求直接返回，而不去调用原本调用的逻辑。

跳闸一段时间后（例如15秒），断路器会进入半开状态，这是一个瞬间态，此时允许一次请求调用该调的逻辑，如果成功，则断路器关闭，应用正常调用；
如果调用依然不成功，断路器继续回到打开状态，过段时间再进入半开状态尝试——通过”跳闸“，应用可以保护自己，而且避免浪费资源；
而通过半开的设计，可实现应用的“自我修复“。

## Hystrix

监控端点配置

```yaml
management:
  endpoint:
    health:
      show-details: always
```

注解用法: @HystrixCommand(fallbackMethod = "findByIdFallback") 用注解的形式加到方法上，不过一都是配合Feign使用，不推荐单独使用

默认Feign是不启用Hystrix的，需要添加如下配置启用Hystrix，这样所有的Feign Client都会受到Hystrix保护

```yaml
feign:
  hystrix:
    enabled: true
```

`断路器实现`

在注解上配置，有fallback和fallbackFactory，工厂多了一层封装(就是为了工厂模式嘛)，工厂模式有异常(不知道是不是要用异常一定要用工厂模式，fallback没有传递异常的地方)

`@FeignClient(value = ServiceConstant.USER_SERVICE, configuration = CustomFeignConfig.class, fallbackFactory = UserServiceClientFallbackFactory.class)`

Feign本身已经整合了Hystrix，可直接使用@FeignClient(value = "microservice-provider-user", fallback = XXX.class) 来指定fallback类，fallback类继承@FeignClient所标注的接口

启动类加@EnableCircuitBreaker，引入spring-cloud-starter-hystrix(整合hystrix，其实feign中自带了hystrix，引入该依赖主要是为了使用其中的hystrix-metrics-event-stream，用于dashboard)

Hystrix服务监控点暴露

```yaml
management:
  endpoints:
    web:
      exposure:
        include: 'hystrix.stream'
```

访问任意Feign Client接口的API后，再访问http://IP:PORT/actuator/hystrix.stream ，就会展示一大堆Hystrix监控数据了

**特别注意：如果没有指定fallback，接口报500，但是我们可以修改默认的设置，这样就不用每个接口都在不可用的情况下返回异常了**

1. fallback

```java
public class ExaminationServiceClientFallbackImpl implements ExaminationServiceClient {
    // 对 ExaminationServiceClient 的各个接口实现，做短路时的处理
}
```

2. fallbackFactory

可以看到实现方法`T create(Throwable cause);`可以获取到异常

```java
public class ExaminationServiceClientFallbackFactory implements FallbackFactory<ExaminationServiceClient> {

    @Override
    public ExaminationServiceClient create(Throwable throwable) {
        ExaminationServiceClientFallbackImpl examinationServiceClientFallback = new ExaminationServiceClientFallbackImpl();
        examinationServiceClientFallback.setThrowable(throwable);
        return examinationServiceClientFallback;
    }
}
```

从源码中可以看到，不做配置，默认实现类是抛出FINE级别日志，当然前提是配置了日志

```java
public interface FallbackFactory<T> {

  /**
   * Returns an instance of the fallback appropriate for the given cause
   *
   * @param cause corresponds to {@link com.netflix.hystrix.AbstractCommand#getExecutionException()}
   *        often, but not always an instance of {@link FeignException}.
   */
  T create(Throwable cause);

  /** Returns a constant fallback after logging the cause to FINE level. */
  final class Default<T> implements FallbackFactory<T> {
    // jul to not add a dependency
    final Logger logger;
    final T constant;

    public Default(T constant) {
      this(constant, Logger.getLogger(Default.class.getName()));
    }

    Default(T constant, Logger logger) {
      this.constant = checkNotNull(constant, "fallback");
      this.logger = checkNotNull(logger, "logger");
    }

    @Override
    public T create(Throwable cause) {
      if (logger.isLoggable(Level.FINE)) {
        logger.log(Level.FINE, "fallback due to: " + cause.getMessage(), cause);
      }
      return constant;
    }

    @Override
    public String toString() {
      return constant.toString();
    }
  }
}
```

### 监控可视化

添加依赖spring-cloud-starter-netflix-hystrix-dashboard，启动类加注解：@EnableHystrixDashboard

但是这个只能监控一个服务，还可以配合Turbine，监控数据聚合-Turbine

添加依赖

```xml
<!--监控可视化-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>

<!--监控数据聚合-Turbine-->
<!-- 因为用了consul，需要排除eureka依赖 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-turbine</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

配置要监控的服务，这样在这个服务下就能监控到其它服务了

```yml
server:
  port: 9186

# 监控数据聚合
turbine:
  appConfig: consul,auth-service,exam-service,user-service,gateway-service,msc-service
  aggregator:
    clusterConfig: CONSUL,AUTH-SERVICE,EXAM-SERVICE,USER-SERVICE,GATEWAY-SERVICE,MSC-SERVICE
```

## Consule

服务网格是把现有的微服务直接的通信部分整合，更加的细致

如何使用: 由于consule是go编写的，CP架构，部署完成后，Spring只需要关注clien如何加入即可完成集成

consule的功能比较丰富，也可以用它来代替config-service，依赖 `spring-cloud-starter-consul-config`

### 部署

consul agent -server -client=0.0.0.0 -bootstrap-expect=3 -data-dir=/Users/liuzhi/cloud/data/ -node=server1

consul agent -dev

## Spring Cloud Config

是一种统一配置管理的方案，简单来说要这样使用: 部署一个config service，这个服务有各个微服务需要用到的配置文件信息，然后可以通过restful或client与config service进行交互，这样各个微服务就能获取到配置参数了

对于这个config service它是可以动态修改的，可以高可用；另外还有一个概念就是配置文件的存储方式，有git，mysql，本地文件系统等形式

如何使用: 要启动一个cloud service服务，然后其它服务去这里读取配置文件，基本就是这么一个流程。

关于高可用: 
1. 存储仓库的高可用，如果是第三方git，本身高可用(除非你网络有问题)，当然实际肯定用自己搭建的，就需要准备高可用，存储也是，文件系统高可用

2. 服务高可用，可以使用注册中心，把多个service注册到服务中心，clien也是从注册中心读取配置。如果不用注册中心，就需要负载均衡器做高可用。当然soa架构，肯定是有注册中心的 

关于动态刷新: 这里我有个疑问，假如是通过`@Value`读取的配置文件，那么刷新后用新的值，确实没问题，但是像数据库配置这种，刷新后应该是要重启才行吧(简单属性和比较复杂的配置属性)，未完待续

数据库配置是已经装配好bean工厂了，理论需要重启，没去式了。config服务会缓存配置信息，所以修改后其它服务取到的都是未修改之前的数据，所以热刷新还是有必要的，开发阶段就不用一直重启配置服务了(因为jrebel无法对资源文件监控，所以还是要重启，推荐尽快跟上热刷新)

实现动态刷新:
1. 引入依赖
2. 开启监控点
3. 加上注解

bus，是通过消息广播，一个客户端刷新后，通知其它服务刷新

注意，在使用了security后，刷新接口会报403异常，需要加入访问权限

### git

项目结构规划

config-service 的client配置的连接service端的配置文件要写bootstrap中，原因如下:

1. bootstrap.yml文件中的内容不能放到application.yml中，否则config部分无法被加载
2. 因为config部分的配置先于application.yml被加载，而bootstrap.yml中的配置会先于application.yml加载
3. bootstarp配置文件是从云端加载配置文件。优先级高于application，项目启动时，会先去加载自带的配置文件(框架自己的)，然后加载bootstrap配置文件，将加载到的内容放入到application中

上面是什么意思呢？这是一个很重要的点，首先对于config-service来说，正常部署使用即可；而对于依赖于config-service配置的其它微服务来说，由于系统启动的时候就需要加载配置文件，绑定config-server的URL，然后在加载application配置。如果bootstrap文件找不到或者没有配置server的URL，系统会默认URL为http://127.0.0.1:8888(当然是你使用spring-cloud-starter-config依赖才是这个逻辑)

可以看到引入依赖后，不配置的默认输出

```log
2020-02-20 17:13:40.418  INFO 36775 --- [  restartedMain] c.c.c.ConfigServicePropertySourceLocator : Fetching config from server at : http://localhost:8888
2020-02-20 17:13:40.554  INFO 36775 --- [  restartedMain] c.c.c.ConfigServicePropertySourceLocator : Connect Timeout Exception on Url - http://localhost:8888. Will be trying the next url if available
2020-02-20 17:13:40.554  WARN 36775 --- [  restartedMain] c.c.c.ConfigServicePropertySourceLocator : Could not locate PropertySource: I/O error on GET request for "http://localhost:8888/application/default": Connection refused (Connection refused); nested exception is java.net.ConnectException: Connection refused (Connection refused)
```

我们直接配置在application中就会导致读取不到配置文件，这个时候的报错大多数都是读取不到配置文件的错误，另外可能有人把配置文件在application也准备一份，这种情况不会报错(因为读取不到，使用了本地的，编译通过)，但是client是没有使用config-service的，这种情况也是要注意的

当我们在代码中使用配置文件的值绑定到属性的时候，service端有用service，没有用本地，本地没有报错

## spring boot admin

用于管理和监控SpringBoot应用程序

应用程序作为Spring Boot Admin Client向为Spring Boot Admin Server注册（通过HTTP）或使用SpringCloud注册中心（例如Eureka，Consul）发现
UI是的AngularJs应用程序，展示Spring Boot Admin Client的Actuator端点上的一些监控

如何使用

1. 首先我们要单独部署一个服务，这个服务就是admin的服务，作为service端，其它spring boot应用作为client端。依赖spring-boot-admin-starter-server，然后启动类加上注解 @EnableAdminServer
2. 其它spring boot应用作为client端加入到admin服务中，这里需要注意，具体如下

如果没用注册中心，客户端是要配置下面的参数的，就是通过http去注册

```yaml
boot:
    admin:
      client:
        url: http://${ADMIN_HOST:localhost}:${ADMIN_PORT:8520}/admin
        username: ${ADMIN_USERNAME:admin}
        password: ${ADMIN_PASSWORD:exVan1234}
        instance:
          service-base-url: http://${USER_SERVICE_HOST:localhost}:${server.port}
          metadata:
            tags:
              environment: dev
```

如果配置了注册中心，可以不需要这个配置，甚至依赖都不需要。但是这种情况发现admin有些许差异

推荐：使用注册中心的话，也把依赖和配置加上

`注意`: Spring Boot Admin 不是 Spring Boot starter 风格的，需要注意版本号，启动失败，大概率是版本不兼容的问题

关于存储: 发现监控服务的一些配置添加后，下次启动还有效，换浏览器无效，猜测是用了浏览器来缓存配置和添加的监控点设置吧，不深入了

关于security: 配置后需要鉴权才能访问admin服务

## 触发自我保护机制

部署起spring cloud相关组件后，在Eureka页面有时会报这个异常

`EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY'RE NOT. RENEWALS ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUST TO BE SAFE.`

Euerka Service和Euerka Client之间，默认30秒进行一次心跳，出现这个异常是因为client少于一定的阈值，server不会删除注册信息，这就有可能导致在调用微服务时，实际上服务并不存在。 这种保护状态实际上是考虑了client和server之间的心跳是因为网络问题，而非服务本身问题，不能简单的删除注册信息


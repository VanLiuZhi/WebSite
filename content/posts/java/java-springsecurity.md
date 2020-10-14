---
weight: 4400
title: "springSecurity安全框架"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "springSecurity安全框架"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Java, Note]
categories: [Java-MicroServices]

lightgallery: true

toc:
  auto: false
---

springSecurity安全框架学习笔记，使用配置总结

<!-- more -->

## Security

如果要对Web资源进行保护，最好的办法莫过于Filter，要想对方法调用进行保护，最好的办法莫过于AOP

Security主要思想就是通过filter来实现的，整个流程包含各种过滤器

1. 主要过滤器  

WebAsyncManagerIntegrationFilter 
SecurityContextPersistenceFilter 
HeaderWriterFilter 
CorsFilter 
LogoutFilter
RequestCacheAwareFilter
SecurityContextHolderAwareRequestFilter
AnonymousAuthenticationFilter
SessionManagementFilter
ExceptionTranslationFilter
FilterSecurityInterceptor
UsernamePasswordAuthenticationFilter
BasicAuthenticationFilter        

2. 框架的核心组件

SecurityContextHolder 提供对SecurityContext的访问
SecurityContext 持有Authentication对象和其他可能需要的信息
AuthenticationManager 认证管理器，其中可以包含多个AuthenticationProvider
ProviderManager 对象为AuthenticationManager接口的实现类
AuthenticationProvider 主要用来进行认证操作的类 调用其中的authenticate()方法去进行认证操作
Authentication Spring Security方式的认证主体
GrantedAuthority 对认证主题的应用层面的授权，含当前用户的权限信息，通常使用角色表示
UserDetails 构建Authentication对象必须的信息，可以自定义，可能需要访问DB得到
UserDetailsService 通过username构建UserDetails对象，通过loadUserByUsername根据userName获取UserDetail对象 （可以在这里基于自身业务进行自定义的实现  如通过数据库，xml,缓存获取等）

## UserDetailsService

这是一个接口，实现后需要返回一个框架自身的User类型对象，User又是UserDetails的实现

主要实现loadUserByUsername方法，就是如何通过用户名获取用户对象，这个用户对象必须是框架提供的类型，并且包括了相关的权限信息

```java
/**
 * @Description
 * @Author VanLiuZhi
 * @Date 2020-02-18 15:01
 */
@Service
public class MyUserDetailsUserService implements UserDetailsService {
    @Override
    public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
        return new User(
                "User",
                PasswordEncoderFactories.createDelegatingPasswordEncoder().encode("123456"),
                AuthorityUtils.commaSeparatedStringToAuthorityList("admin"));
    }
}
```

上面是一个模拟，通过用户名加载到都是一样的User对象，实际应该去数据库查询

**关于密码加密的形式，我找到两种说法，这里做个记录。在我的比较新的版本中，已经不会出现方式一的问题了**

不过我也测试出了很多有价值的东西，在配置文件中配置`passwordEncoder`，使用`PasswordEncoderFactories.createDelegatingPasswordEncoder()`，和`userDetailsService`实现类中，也使用同样的(看上图的代码)，那么就是正常的。猜测原来默认不配置应该是使用`PasswordEncoderFactories.createDelegatingPasswordEncoder()`进行加密，后来改了，新版本的默认加密编码就用`BCryptPasswordEncoder()`

总结就是要统一，测试版本5.1.7，推荐使用方式二

```java
// config文件中
@Override
public void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.userDetailsService(userDetailsService).
            passwordEncoder(passwordEncoder());
}

@Bean
public PasswordEncoder passwordEncoder() {
    return PasswordEncoderFactories.createDelegatingPasswordEncoder();
}
```

{% blockquote %}

方式一:

不能直接创建 `BCryptPasswordEncoder` 对象来加密， 这种加密方式 没有 `{bcrypt}` 前缀
会导致在 matches 时导致获取不到加密的算法出现 `java.lang.IllegalArgumentException: There is no PasswordEncoder mapped for the id "null"`  问题。
问题原因是 Spring Security5 使用 DelegatingPasswordEncoder(委托)  替代 NoOpPasswordEncoder，并且 默认使用  BCryptPasswordEncoder 加密（注意 DelegatingPasswordEncoder 委托加密方法 BCryptPasswordEncoder 加密前 添加了加密类型的前缀）
https://blog.csdn.net/alinyua/article/details/80219500

方式二: 

1. 在配置中声名一个bean

@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}

2. UserDetailsService实现类注入

@Autowired
private PasswordEncoder passwordEncoder;

3. 加密

passwordEncoder.encode("123456")
{% endblockquote %}

两种方式加密的结果，确实是多了个前缀，具体可以看源码

$2a$10$oUpt1tFddYCQXy0V3EEt6uM1V2BzRnYrGbcBhpOUFq1egzS2ocYZO
`{bcrypt}`$2a$10$CZWyE3FxdYfvYRGdTBVYue2i5PISShMxtgSnVJ4zs0RPyb9QLkJbW


## Security Config

Security配置，是继承覆盖父类的形式，围绕着两个方面来

1. 安全策略配置自定义
2. 用户详情和密码加密方法自定义

最简单的实现就是覆盖两个方法，一个负责对请求资源的权限设置，比如哪些路径需要登录。还有另外一个方法，配置认证，就是配置怎么去认证，输入的账号密码是否正确

刚才对UserDetailService的实现在这里就用到了

### 用到的注解

`@EnableWebSecurity`
启用WebSecurity，都要配置

`@EnableGlobalMethodSecurity(prePostEnabled=true)`
要开启Spring方法级安全，在添加了@Configuration注解的类上再添加@EnableGlobalMethodSecurity注解即可，对请求方法做权限控制，一般都要用

查看注解，发现它还提供了很多类型

`prePostEnabled`： 确定 前置注解[@PreAuthorize,@PostAuthorize,..] 是否启用
`securedEnabled`： 确定安全注解 [@Secured] 是否启用
`jsr250Enabled`： 确定 JSR-250注解 [@RolesAllowed..]是否启用

可以启用多种类型的注解，但是在方法中只能生效一种

```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true))
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    ...
}
```

在方法中使用，多数是在controller层用

```java
public interface UserService {
  List<User> findAllUsers();

  @PreAuthorize("hasAnyRole('user')")
  void updateUser(User user);

    // 下面不能设置两个注解，如果设置两个，只有其中一个生效
    // @PreAuthorize("hasAnyRole('user')")
  @Secured({ "ROLE_user", "ROLE_admin" })
  void deleteUser();
}
```

总结就是启用方法权限控制，通过设置的权限和用户的权限表做匹配决定是否能调用方法，这里用户的权限表就是通过UserDetailService来获取的。有三种模式，PreAuthorize用的比较广，有专门的表达式来编写，比如`hasAnyRole('user')`就是用了表达式，这个不是乱写的

参考
https://docs.spring.io/spring-security/site/docs/4.0.1.RELEASE/reference/htmlsingle/#el-common-built-in

https://www.jianshu.com/p/77b4835b6e8e


### 配置类

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private UserDetailsService userDetailsService;

    /**
     * 这是Security的安全认证策略,默认的是所有请求都可以在授权之后访问
     */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .formLogin()
                .and()
                .authorizeRequests()
                .anyRequest()
                .authenticated();
    }
```

其它例子

```java
// 不会拦截/user,因为只会匹配"/api/**"，可以写多个http，不要冲突

http.requestMatchers().antMatchers("/api/**")
        .and().authorizeRequests().antMatchers("/user","/api/user").authenticated()
        .anyRequest().authenticated()
        .and().formLogin().loginPage("/login");
        //需要人证
        // http.authorizeRequests().antMatchers("/user").hasRole("Admin");
        // .and().formLogin().loginPage("/login");
        // api请求都不需要权限认证
        // http.authorizeRequests().antMatchers("/api").permitAll();
        // data/** Get请求不需要权限人证
        // http.authorizeRequests().antMatchers(HttpMethod.GET,"/data/**").permitAll();
```

方法使用说明，大致就是各种路由设置，鉴权设置

authorizeRequests() 开始请求权限配置
antMatchers() 使用Ant风格的路径匹配
permitAll() 用户可任意访问
anyRequest() 匹配所有路径
authenticated() 用户登录后可访问

- URL匹配
requestMatchers() 配置一个request Mather数组，参数为RequestMatcher 对象，其match 规则自定义，需要的时候放在最前面，对需要匹配的的规则进行自定义与过滤
authorizeRequests() URL权限配置
antMatchers() 配置一个request Mather 的 string数组，参数为 ant 路径格式， 直接匹配url
anyRequest 匹配任意url，无参 ,最好放在最后面

- 保护URL
authenticated() 保护UrL，需要用户登录
permitAll() 指定URL无需保护，一般应用与静态资源文件
hasRole(String role) 限制单个角色访问，角色将被增加 “ROLE_” .所以”ADMIN” 将和 “ROLE_ADMIN”进行比较. 另一个方法是hasAuthority(String authority)
hasAnyRole(String… roles) 允许多个角色访问. 另一个方法是hasAnyAuthority(String… authorities)
access(String attribute) 该方法使用 SPEL, 所以可以创建复杂的限制 例如如access("permitAll"), access("hasRole('ADMIN') and hasIpAddress('123.123.123.123')")
hasIpAddress(String ipaddressExpression) 限制IP地址或子网

- 登录login
formLogin() 基于表单登录
loginPage() 登录页
defaultSuccessUrl 登录成功后的默认处理页
failuerHandler登录失败之后的处理器
successHandler登录成功之后的处理器
failuerUrl登录失败之后系统转向的url，默认是this.loginPage + "?error"

- 登出logout
logoutUrl 登出url ， 默认是/logout， 它可以是一个ant path url
logoutSuccessUrl 登出成功后跳转的 url 默认是"/login?logout"
logoutSuccessHandler 登出成功处理器，设置后会把logoutSuccessUrl 置为null


特别

```java
// HttpSecurity 下的
public HttpSecurity and() {
    return HttpSecurity.this;
}
```

```java
// SecurityConfigurerAdapter 下的

public B and() {
    return getBuilder();
}

protected final B getBuilder() {
    if (securityBuilder == null) {
        throw new IllegalStateException("securityBuilder cannot be null");
    }
    return securityBuilder;
}
```

有2个and方法，有些方法会返回getBuilder，也和and一样，最后指向securityBuilder

## OAuth 2.0

项目源码数据库模型地址: https://github.com/spring-projects/spring-security-oauth/blob/master/spring-security-oauth2/src/test/resources/schema.sql

学习一个新技术，最好的方式莫过于直接看源码

security的使用还是比较简单的，复杂的是Oauth2

OAuth 的核心就是向第三方应用颁发令牌，OAuth 2.0 规定了四种获得令牌的流程，可以选择一种方式颁发令牌

授权码（authorization-code）
隐藏式（implicit）
密码式（password）
客户端凭证（client credentials）

Authorization Code（授权码模式）：正宗的OAuth2的授权模式，客户端先将用户导向认证服务器，登录后获取授权码，然后进行授权，最后根据授权码获取访问令牌；
Implicit（简化模式）：和授权码模式相比，取消了获取授权码的过程，直接获取访问令牌；
Resource Owner Password Credentials（密码模式）：客户端直接向用户获取用户名和密码，之后向认证服务器获取访问令牌；
Client Credentials（客户端模式）：客户端直接通过客户端认证（比如client_id和client_secret）从认证服务器获取访问令牌。

这里说下自己的理解: 

授权码模式安全级别最高，是因为，认证服务器，授权服务器都是自己信任的，而涉及到客户端获取用户账号密码，是不可信的。

简单来说，假如我开发了一套大型系统，我提供给各个企业用，企业可以实现自己的访问客户端，但是用户数据在我这里，那么企业自己的客户端就是不可信的

我们可以通过常见的qq第三方登录来说明。比如通过qq登录，客户端是第三方开发的，它支持qq登录，但是你不会直接把qq密码在客户端上输入，这是不安全的，qq提供认证服务器给客户端，客户端引导用户去qq的认证登录页面，由于这个页面是qq做的，所以它是安全的，你输入账号密码后，得到授权码，之后客户端通过授权码去获取令牌，这样后续qq的接口，比如获取用户详情等，客户端通常需要这些数据来创建自己的用户，之后的qq接口通过授权码访问即可。你会发现，客户端全程拿不到你的密码，所以授权码模式级别高，但是也复杂。只要理解了授权码模式，理解其它模式就简单了


`在程序中，需要做相关的配置，使用上面的英文`

注意，不管哪一种授权方式，第三方应用申请令牌之前，都必须先到系统备案，说明自己的身份，然后会拿到两个身份识别码：客户端 ID（client ID）和客户端密钥（client secret）。这是为了防止令牌被滥用，没有备案过的第三方应用，是不会拿到令牌的

`在程序中也有这个配置，声名哪些服务能获取令牌`

在架构设计中，一般存在三种角色

1. 客户应用
2. 资源服务器
3. 授权服务器  

authorization 授权 authentication 认证

authentication证明你是你，authorization证明你有这个权限

简单来说，身份验证是验证您的身份的过程，而授权是验证您有权访问的过程

参考 https://www.lagou.com/lgeduarticle/44562.html

客户应用要从资源服务器获取数据，资源服务器会提供API，但是这样的话不安全，所以客户服务器需要提供一个Access Token，资源服务器会校验这个token，来确认这个客户应用是否有访问权限

那么问题也来了，客户应用的Access Token是怎么来的？这就需要授权服务器了，客户应用去授权服务器上申请Access Token

**AuthenticationManager，AccessDecisionManager，AbstractSecurityInterceptor属于spring security框架的基础铁三角**

### 涉及到的术语，概念

复杂体现到术语和概念比较多，首先要理解这些东西，我们直接从代码去理解

### 1. 资源服务器配置

```java
@EnableResourceServer
ResourceServerConfigurerAdapter
```

通常我们会做上面的配置，使用注解，继承一个类，通过这两步来完成资源服务器的配置，下面一步一步的分析

@EnableResourceServer引入 OAuth2AuthenticationProcessingFilter 过滤器，import的类ResourceServerConfiguration

`主要覆盖方法`

public void configure(ResourceServerSecurityConfigurer resources)

public void configure(HttpSecurity http)

```java
@Configuration
public class ResourceServerConfiguration extends WebSecurityConfigurerAdapter implements Ordered
```

### 2. 授权服务配置

```java
@EnableAuthorizationServer
AuthorizationServerConfigurerAdapter
```

`主要覆盖方法`

public void configure(ClientDetailsServiceConfigurer clients)

public void configure(AuthorizationServerSecurityConfigurer oauthServer)

ClientDetailsServiceConfigurer：用来配置客户端详情服务，客户端详情信息在这里进行初始化，你能够把客户端详情信息写死在这里或者是通过数据库来存储调取详情信息。客户端就是指第三方应用
AuthorizationServerSecurityConfigurer：用来配置令牌端点(Token Endpoint)的安全约束.
AuthorizationServerEndpointsConfigurer：用来配置授权（authorization）以及令牌（token）的访问端点和令牌服务(token services)。


### 3. Security配置

@EnableWebSecurity
WebSecurityConfigurerAdapter

`主要覆盖方法`

protected void configure(HttpSecurity http)

Spring Security 中的配置优先级高于资源服务器中的配置，即请求地址先经过 Spring Security 的 HttpSecurity，再经过资源服务器的 HttpSecurity

### refresh_token

刷新模式，就是获取令牌的接口如果支持刷新模式，那么直接使用refresh_token就可以获取到令牌，该模式就不需要前面的繁琐流程，直接获取到令牌token，前提是你要有refresh_token，并且它没过期

refresh_token一般是获取access token的时候返回的

### 简单使用

在一个服务上，可以同时实现 资源服务器配置 授权服务配置 Security配置

一定要理解资源服务器，授权服务器，和security三个配置的职责

`资源服务器`: 负责控制资源

`授权服务器`: 发放令牌的

`security`: 安全校验，比如登录校验，权限校验

开始用的时候，总感觉有些是重合的，但是其实是各司其职的

然后根据不同的模式(4种)去做

授权码模式相对来说比较繁琐，密码模式比较简单

直接使用cloud的依赖，比较适合cloud的项目

```xml
<!--oauth2依赖-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-oauth2</artifactId>
</dependency>
<!--security依赖-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-security</artifactId>
</dependency>
```

## Security Oauth2知识点总结

1. 报错Access is denied

```json
{
    "error": "access_denied",
    "error_description": "Access is denied"
}
```

这通常是token校验通过了，但是你没有访问权限，所以被拒绝了(比如当前角色不允许访问这个资源)

2. 不要轻易调用父类方法

Security Oauth2的集成中，有很多方法需要我们重写，IDEA快捷键创建的方法默认会调用父类的方法
有很多配置是不能调用父类方法的，比如`super.configure(HttpSecurity http)`，这也是非常容易出错的地方，因为父类方法是全部拦截的

3. 基于内存和基于Redis的普通token存储

资源服务器也是需要拿着token去授权服务器校验的(JWT就不需要了，资源服务器可以自己校验)，如果是基于内存的，服务重启就失效了，基于Redis自然就不会(服务怎么重启只要不失效都可以去Redis中获取)

但是基于Redis的，需要资源服务器配置token的store为去Redis读取。很多Hello World教程都不告诉你这个的

```java
@Override
public void configure(ResourceServerSecurityConfigurer resources) {
    // 配置资源id，这里的资源id和授权服务器中的资源id一致
    resources.resourceId("myResource")
            // 设置这些资源仅基于令牌认证
            .stateless(true);
    // 配置去Redis获取token，才能和传入的token做验证
    resources.tokenStore(new RedisTokenStore(redisConnectionFactory));
}
```

4. jwt扩展

TokenEnhancer，使用这种方式，是获取jwt的时候，会附加信息。附加信息不添加到载荷中()
DefaultAccessTokenConverter，使用这种方式，是扩展jwt的载荷信息，资源服务器可以解析到载荷中附加的数据

## 参考

https://blog.csdn.net/u012702547/article/details/107530246

http://blog.didispace.com/spring-security-oauth2-xjf-3/

https://blog.csdn.net/andy_zhang2007/article/details/90023901





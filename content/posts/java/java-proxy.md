---
weight: 1
title: "Java Proxy 与 AOP"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Java Proxy 与 AOP"
resources:
- name: "base-image"
  src: "base-image.jpg"

tags: [Java, Note]
categories: [Java]

lightgallery: true

toc:
  auto: false
---

## 静态代理和动态代理

静态代理就是直接编码的形式，比如有一个UserDao，提供查询和修改数据库功能，假如我要扩展它，让每个数据库操作都在事务中执行，势必要修改代码。 可以代理这个UserDao，由代理类来处理事务，查询和修改的调用还是在UserDao中

参考 https://blog.csdn.net/hon_3y/article/details/70655966

例子

```java
package com.liuzhidream.rrdtool.store.service.proxy;

/**
 * @Description
 * @Author VanLiuZhi
 * @Date 2020-03-11 14:54
 */

// 服务接口
interface IUserDao {
    public int query(Integer pk);

    public int delete(Integer pk);
}

// 服务实现

class UserDao implements IUserDao {

    @Override
    public int query(Integer pk) {
        System.out.println("查询的数据id是" + pk);
        return pk;
    }

    @Override
    public int delete(Integer pk) {
        System.out.println("删除的数据id是" + pk);
        return pk;
    }
}

// 如果要扩展事务，需要修改代码
//public int delete(Integer pk) {
//    System.out.println("transaction staring");
//    System.out.println("删除的数据id是" + pk);
//    System.out.println("transaction end");
//    return pk;
//}

// 使用静态代理

public class StaticProxy implements IUserDao {

    private UserDao userDao;

    public StaticProxy(UserDao userDao) {
        this.userDao = userDao;
    }

    @Override
    public int query(Integer pk) {
        System.out.println("transaction staring");
        int res = userDao.query(pk);
        System.out.println("transaction end");
        return res;
    }

    @Override
    public int delete(Integer pk) {
        System.out.println("transaction staring");
        int res = userDao.delete(pk);
        System.out.println("transaction end");
        return res;
    }
}
```

缺点:

1. 代理类也是对接口的实现，如果接口改了，也要跟着改
2. 代理对象需要实现和目标对象一样的接口，会有很多代理类，类太多

动态代理: 可以使用动态代理，代理对象，不需要实现接口，就不会有太多的代理类。动态代理是在内存中创建对象的，利用JDKAPI

动态代理约束: **目标对象一定是要有接口的，没有接口就不能实现动态代理**

动态代理举例，使用Proxy.newProxyInstance创建目标对象

```java
package com.liuzhidream.rrdtool.store.service.proxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

/**
 * @Description
 * @Author VanLiuZhi
 * @Date 2020-03-11 14:02
 */
public class ProxyFactory {
    XiaoMing xiaoMing = new XiaoMing();

    public Person getProxy() {
        /*
         * loader 生成代理对象使用哪个类装载器【一般我们使用的是代理类的装载器】
         * interfaces 目标对象的接口。生成哪个对象的代理对象，通过接口指定【指定要代理类的接口】
         * h 生成的代理对象的方法里干什么事【实现handler接口，我们想怎么实现就怎么实现】
         *
         * public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)
         */
        return (Person) Proxy.newProxyInstance(ProxyFactory.class.getClassLoader(), xiaoMing.getClass().getInterfaces(), new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                // 这里只实现了其中一个方法，就是代理对象实例调用getProxy后，再去调用方法，就由这里的逻辑来处理
                if (method.getName().equals("sing")) {
                    System.out.println("1000");
                    method.invoke(xiaoMing, args);
                }
                return null;
            }
        });
    }

    public static void main(String[] args) {
        ProxyFactory proxyFactory = new ProxyFactory();
        Person proxy = proxyFactory.getProxy();
        proxy.sing("你好");
    }

}

interface Person {
    void sing(String name);

    void dance(String name);
}

class XiaoMing implements Person {

    @Override
    public void sing(String name) {

        System.out.println("小明唱" + name);
    }

    @Override
    public void dance(String name) {

        System.out.println("小明跳" + name);

    }
}
```

## cglib

引入cglib – jar文件，spring core包含了对应的代码，可以用spring的

cglib在内存中动态构建子类

代理的类不能为final，否则报错(在内存中构建子类来做扩展，当然不能为final，有final就不能继承了)

目标对象的方法如果为final/static, 那么就不会被拦截，即不会执行目标对象额外的业务方法

代码举例

```java
package com.liuzhidream.rrdtool.store.service.proxy;

import org.springframework.cglib.proxy.Enhancer;
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

/**
 * @Description
 * @Author VanLiuZhi
 * @Date 2020-03-11 15:16
 */
public class CGlibProxyFactory implements MethodInterceptor {

    // 维护目标对象
    private Object target;

    public CGlibProxyFactory(Object target) {
        this.target = target;
    }

    // 给目标对象创建代理对象
    public Object getProxyInstance() {
        //1. 工具类
        Enhancer en = new Enhancer();
        //2. 设置父类(目标对象)
        en.setSuperclass(target.getClass());
        //3. 设置回调函数
        en.setCallback(this);
        //4. 创建子类(代理对象)
        return en.create();
    }


    @Override
    public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        System.out.println("开始事务.....");

        // 执行目标对象的方法
        Object returnValue = method.invoke(target, args);

        System.out.println("提交事务.....");
        return returnValue;
    }
}

class User {

    public int query(Integer pk) {
        System.out.println("query pk is" + pk);
        return pk;
    }

    public final int delete(Integer pk) {
        System.out.println("delete pk is" + pk);
        return pk;
    }

    public static void main(String[] args) {
        User user = (User) new CGlibProxyFactory(new User()).getProxyInstance();
        user.query(21);

        // class com.liuzhidream.rrdtool.store.service.proxy.User$$EnhancerByCGLIB$$fa2d90ed
        // 对象已经不是原对象了
        System.out.println(user.getClass()); 
        
        System.out.println(user.delete(21)); // 不会被拦截
    }
}

```

## 动态代理和cglib

动态代理是jdk通过API创建的的(利用的是反射)，cglib是asm字节码操作框架实现的(利用字节码)。都是动态代理

但是jdk动态代理，`目标对象要实现接口`，有一定的局限性。cglib `目标对象不用实现接口`

JDK是基于反射机制,生成一个实现代理接口的匿名类,然后重写方法,实现方法的增强.
它生成类的速度很快,但是运行时因为是基于反射,调用后续的类操作会很慢.
而且他是只能针对接口编程的.

CGLIB是基于继承机制,继承被代理类,所以方法不要声明为final,然后重写父类方法达到增强了类的作用.
它底层是基于asm第三方框架，是对代理对象类的class文件加载进来,通过修改其字节码生成子类来处理.
生成类的速度慢,但是后续执行类的操作时候很快.
可以针对类和接口。

## AOP

参考: 链接：https://juejin.im/post/5aa8edf06fb9a028d0432584

AOP这块，首先要把一些术语先记住，其实AOP个人感觉不是什么特别神奇的东西，和Python的装饰器很像，就像很多语言都有反射一样，但是这些术语不知道会
阻碍你学习AOP，所以先过一遍相关术语

关注点代码和核心代码: 

- 所谓关注点代码就是重复的代码，就行最开始举例中的事务相关的代码，它的特点就是重复出现(这个点也叫切面，横切关注点)

- 核心代码就是业务代码，比如查询用户(核心关注点)

使用AOP的作用: `关注点代码和核心代码分离`

这样做的好处: 1. 关注点代码只用写一次  2. 开发者把工作放到核心代码上 3. 运行时期，执行核心业务代码时候动态植入关注点代码，也就是代理

另外，这里聊的AOP主要是在spring中去运用，spring借鉴了Aspectj框架的概念和思想，但是spring没有依赖这个框架，只有一个注解@Aspect，看着好像是有用到Aspectj的东西，其实不是，它只是一个注解，刚好名字接近了

下面是概念：

切面(aspect):类是对物体特征的抽象，切面就是对横切关注点的抽象

横切关注点:对哪些方法进行拦截，拦截后怎么处理，这些关注点称之为横切关注点。

连接点(joinpoint):被拦截到的点，因为 Spring 只支持方法类型的连接点，所以在 Spring中连接点指的就是被拦截到的方法，实际上连接点还可以是字段或者构造器。

切入点(pointcut):对连接点进行拦截的定义

通知(advice):所谓通知指的就是指拦截到连接点之后要执行的代码，通知分为前置、后置、 异常、最终、环绕通知五类。

目标对象:代理的目标对象

织入(weave):将切面应用到目标对象并导致代理对象创建的过程

引入(introduction):在不修改代码的前提下，引入可以在运行期为类动态地添加一些方法 或字段。

### 如何使用

1. 加入依赖

2. 定义切面，它是一个bean

3. 在切面中去定义连接点，就是要对哪个方法进行代理

方法的代理涉及到通知类型，有前置通知，后置通知，异常通知，最终通知，环绕通知

通知是注解的方式，用在方法上，注解参数是切入点表达式，切入点表达式指定哪些方法会被代理

环绕通知比较特别，它能传递参数，需要我们在方法体内执行代理方法，并返回，然后围绕这个方法执行做一些事，比如方法执行时间计算

其它通知都不能传递参数(是不是一定不能传未知，因为大部分都是用环绕通知)

### 相关注解

1. @Aspect	

指定一个类为切面类

2. @Pointcut("execution( cn.itcast.e_aop_anno..(..))")  

指定切入点表达式，把这个注解运用到方法上。这样通知注解就不用写切入点表达式了，方法没有方法体

3. @Before("pointCut_()")				

前置通知: 目标方法之前执行，这里的参数就是被@Pointcut注解的方法

4. @After("pointCut_()")				

后置通知: 目标方法之后执行（始终执行）

5. @AfterReturning("pointCut_()")		    

返回后通知: 执行方法结束前执行(异常不执行)

6. @AfterThrowing("pointCut_()")	

异常通知: 出现异常时候执行

7. @Around("pointCut_()")				

环绕通知: 环绕目标方法执行

### 切入点表达式

`execution(modifiers-pattern? ret-type-pattern declaring-type-pattern? name-pattern(param-pattern) throws-pattern?)`

符号讲解：

?号代表0或1，可以不写
“*”号代表任意类型，0或多
方法参数为..表示为可变参数

参数讲解：

1. modifiers-pattern?【修饰的类型，可以不写】

这个修饰类型有很多种，一般用execution，

还有@annotation，表明使用这个注解的方法会被代理，还有this，within，具体看文档

2. ret-type-pattern【方法返回值类型，必写】

3. declaring-type-pattern?【方法声明的类型，可以不写】

4. name-pattern(param-pattern)【要匹配的名称，括号里面是方法的参数】

5. throws-pattern?【方法抛出的异常类型，可以不写】

由于很多可以不写，而且这个表达式又长，不必特意记住，有个印象，具体用法查文档吧


举例

```java
<!-- 【拦截所有public方法】 -->
<!--<aop:pointcut expression="execution(public * *(..))" id="pt"/>-->

<!-- 【拦截所有save开头的方法 】 -->
<!--<aop:pointcut expression="execution(* save*(..))" id="pt"/>-->

<!-- 【拦截指定类的指定方法, 拦截时候一定要定位到方法】 -->
<!--<aop:pointcut expression="execution(public * cn.itcast.g_pointcut.OrderDao.save(..))" id="pt"/>-->

<!-- 【拦截指定类的所有方法】 -->
<!--<aop:pointcut expression="execution(* cn.itcast.g_pointcut.UserDao.*(..))" id="pt"/>-->

<!-- 【拦截指定包，以及其自包下所有类的所有方法】 -->
<!--<aop:pointcut expression="execution(* cn..*.*(..))" id="pt"/>-->

<!-- 【多个表达式】 -->
<!--<aop:pointcut expression="execution(* cn.itcast.g_pointcut.UserDao.save()) || execution(* cn.itcast.g_pointcut.OrderDao.save())" id="pt"/>-->
<!--<aop:pointcut expression="execution(* cn.itcast.g_pointcut.UserDao.save()) or execution(* cn.itcast.g_pointcut.OrderDao.save())" id="pt"/>-->
<!-- 下面2个且关系的，没有意义 -->
<!--<aop:pointcut expression="execution(* cn.itcast.g_pointcut.UserDao.save()) &amp;&amp; execution(* cn.itcast.g_pointcut.OrderDao.save())" id="pt"/>-->
<!--<aop:pointcut expression="execution(* cn.itcast.g_pointcut.UserDao.save()) and execution(* cn.itcast.g_pointcut.OrderDao.save())" id="pt"/>-->

<!-- 【取非值】 -->
<!--<aop:pointcut expression="!execution(* cn.itcast.g_pointcut.OrderDao.save())" id="pt"/>-->
```

更多内容参考 官方文档: https://docs.spring.io/spring/docs/5.2.5.RELEASE/spring-framework-reference/core.html#aop-pointcuts-examples







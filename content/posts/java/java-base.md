---
weight: 4400
title: "Java学习笔记"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Java学习笔记"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Java, Note]
categories: [Java-MicroServices]

lightgallery: true

toc:
  auto: false
---

Java 学习笔记，Java是一门面向对象编程语言，不仅吸收了C++语言的各种优点，还摒弃了C++里难以理解的多继承、指针等概念，因此Java语言具有功能强大和简单易用两个特征。Java语言作为静态面向对象编程语言的代表，极好地实现了面向对象理论，允许程序员以优雅的思维方式进行复杂的编程

![image](/images/Java/Java-base.jpg)
                
<!-- more -->

## 名词解释

大部分都是通用的，或者是设计模式，只是第一次接触难免有些生疏晦涩。

构件：其实就是组件，可以按照组件来理解，不过在这个范畴内一般翻译为构件

DAO：数据库访问模型 Data Access Object

DTO：数据传输对象（DTO)(Data Transfer Object)，是一种设计模式之间传输数据的软件应用系统。数据传输目标往往是数据访问对象从数据库中检索数据。数据传输对象与数据交互对象或数据访问对象之间的差异是一个以不具有任何行为除了存储和检索的数据（访问和存取器）。

Bean：它的定义为，描述Java的软件组件模型，EJB是Enterprise Java Bean的缩写。javaBean在MVC设计模型中是model，又称模型层，在一般的程序中，我们称它为数据层，就是用来设置数据的属性和一些行为，然后我会提供获取属性和设置属性的get/set方法JavaBean是一种JAVA语言写成的可重用组件。

JDBC（Java DataBase Connectivity,java数据库连接）是一种用于执行SQL语句的Java API，可以为多种关系数据库提供统一访问，它由一组用Java语言编写的类和接口组成。JDBC提供了一种基准，据此可以构建更高级的工具和接口，使数据库开发人员能够编写数据库应用程序，同时，JDBC也是个商标名。

JPA是Java Persistence API的简称，中文名Java持久层API，是JDK 5.0注解或XML描述对象－关系表的映射关系，并将运行期的实体对象持久化到数据库中。
Sun引入新的JPA ORM规范出于两个原因：其一，简化现有Java EE和Java SE应用开发工作；其二，Sun希望整合ORM技术，实现天下归一。

JNDI(Java Naming and Directory Interface,Java命名和目录接口)是SUN公司提供的一种标准的Java命名系统接口，JNDI提供统一的客户端API，通过不同的访问提供者接口JNDI服务供应接口(SPI)的实现，由管理者将JNDI API映射为特定的命名服务和目录系统，使得Java应用程序可以和这些命名服务和目录服务之间进行交互。

- PO：persistent object 持久对象
1 ．有时也被称为Data对象，对应数据库中的entity，可以简单认为一个PO对应数据库中的一条记录
2 ．在hibernate持久化框架中与insert/delet操作密切相关
3 ．PO中不应该包含任何对数据库的操作

- POJO ：plain ordinary java object 无规则简单java对象
一个中间对象，可以转化为PO、DTO、VO

1 ．POJO持久化之后==〉PO
（在运行期，由Hibernate中的cglib动态把POJO转换为PO，PO相对于POJO会增加一些用来管理数据库entity状态的属性和方法。PO对于programmer来说完全透明，由于是运行期生成PO，所以可以支持增量编译，增量调试）
2 ．POJO传输过程中==〉DTO
3 ．POJO用作表示层==〉VO

PO 和VO都应该属于它

- BO：business object 业务对象
业务对象主要作用是把业务逻辑封装为一个对象。这个对象可以包括一个或多个其它的对象

比如一个简历，有教育经历、工作经历、社会关系等等。我们可以把教育经历对应一个PO，工作经历对应一个PO，社会关系对应一个PO
建立一个对应简历的BO对象处理简历，每个BO包含这些PO

这样处理业务逻辑时，我们就可以针对BO去处理
封装业务逻辑为一个对象（可以包括多个PO，通常需要将BO转化成PO，才能进行数据的持久化，反之，从DB中得到的PO，需要转化成BO才能在业务层使用）

关于BO主要有三种概念

1 、只包含业务对象的属性
2 、只包含业务方法
3 、两者都包含

在实际使用中，认为哪一种概念正确并不重要，关键是实际应用中适合自己项目的需要

- VO：value object 值对象 / view object 表现层对象

1 ．主要对应页面显示（web页面/swt、swing界面）的数据对象
2 ．可以和表对应，也可以不，这根据业务的需要

- DTO（TO）：Data Transfer Object 数据传输对象

1 ．用在需要跨进程或远程传输时，它不应该包含业务逻辑
2 ．比如一张表有100个字段，那么对应的PO就有100个属性（大多数情况下，DTO内的数据来自多个表）。但view层只需显示10个字段，没有必要把整个PO对象传递到client，这时我们就可以用只有这10个属性的DTO来传输数据到client，这样也不会暴露server端表结构。到达客户端以后，如果用这个对象来对应界面显示，那此时它的身份就转为VO

- DAO：data access object数据访问对象

1 ．主要用来封装对DB的访问（CRUD操作）
2 ．通过接收Business层的数据，把POJO持久化为PO


## 环境变量

配置环境变量，保证 java javac java -version 都能输出正确信息

## classpath路径

Java项目中classpath路径
1、src不是classpath, WEB-INF/classes、lib、resources才是classpath，WEB-INF/是资源目录, 客户端不能直接访问。

2、WEB-INF/classes目录存放src目录java文件编译之后的class文件，xml、properties等资源配置文件，这是一个定位资源的入口。

3、引用classpath路径下的文件，只需在文件名前加classpath:
`<property name="configLocation" value="classpath:/mybatis/mybatis-config.xml" />`
4、lib和classes同属classpath，两者的访问优先级为: lib>classes。

5、classpath 和 classpath* 区别：
classpath：只会到你的class路径中查找找文件;
classpath*：不仅包含class路径，还包括jar文件中(class路径)进行查找。

## 关于版本

你肯定听说过
Java SE（Java Platform，Standard Edition）
Java EE（Java Platform，Enterprise Edition）
Java ME（Java Platform，Micro Edition）

服务端开发，要用java ee，其实只要下载jdk就行了，jdk(Java SE Development Kit 8 Downloads) Java开发工具包，包含了jre(Java运行时环境，如果只是跑代码，只需要jre就可以了)

JDK与Java SE/EE/ME的区别

jdk是不区分se、ee、me的，所以你在oracle的官网上只要下载java se对应的版本jdk即可，你可能会奇怪，不是ee才是企业级开发吗？为什么下载jdk就可以了？

参考引文 http://javaligang.blog.51cto.com/5026500/1825681

Java刚开始的时候，因为各种应用和生态不成熟，很多东西需要有人牵头制定强制规范引导Java的发展，于是Java EE曾经引领了企业级应用的开发。

但随着时代的进步，以及越来越多的公司和组织参与到Java世界，出现了各种各样的Java EE组件的代替者，比如Hibernate、Spring就是其中两个典型。相反，Java官方制定的各种Java EE规范反而不太受欢迎，他们制定了JSF规范，但实际企业开发喜欢用Struts 2、Spring MVC；他们制定了EJB规范，但实际企业开发往往还是喜欢用Spring；他们制定了JPA规范，但实际企业开发往往还是喜欢直接用Hibernate、MyBatis。

现代企业级应用常用的各种框架和工具，比如Struts 2、Spring、Hibernate、jBPM、Activiti、Lucene、Hadoop、Drools、CXF等这些大家耳熟能详的组件，全部都不是来自Oracle官方，但是却在企业应用中开发经常用到的。

现在企业里面，真正常用的JavaEE规范有什么？Servlet、JSP、JMS、JNDI。这些技术都只是充当了一个程序的入口而已。

Oracle之所以可能考虑放弃Java EE，正体现了Oracle对丧失Java控制权的无奈。企业的本质是逐利，Oracle每年为制定Java EE规范投入不少人力、财力，但制定的规范最终并没有获得市场的青睐，所以Oracle可能放弃这种吃亏不讨好的事情。

但Java不同，2016年6月，Java在商业语言排行榜上的市场份额将近21%，庞大到恐怖的市场份额，背后隐藏着巨大各种专利使用费和盈利商机，任何一个理智的公司都不会放弃这个会下金蛋的母鸡。

由此可见，oracle上提供的java EE是官方指定的javaEE规范，里面都是符合官方指定的javaEE组件，我们用SSM，SSH开发后台时使用到的只有Servlet、JSP、JMS等少量的java EE规范，没有必要使用orcale提供的java EE版本，直接使用jdk就可以（当然还需要maven等管理第三方的jar包来实现功能）

有时会有这样的一个说法，选择jdk1.x的版本还是jdk8的版本这样的，jdk1.x的说法是很多年前遗留下来的说法，而现在我们统称的叫法是jdk8这样子。

## Oracle jdk 和 Open jdk

java -version

(1) 如果是SUN/OracleJDK, 显示信息为:

[root@localhost ~]# java -version
java version "1.8.0_162"
Java(TM) SE Runtime Environment (build 1.8.0_162-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.162-b12, mixed mode)

Java HotSpot(TM) 64-Bit Server VM 表明, 此JDK的JVM是Oracle的64位HotSpot虚拟机, 运行在Server模式下(虚拟机有Server和Client两种运行模式).
Java(TM) SE Runtime Environment (build 1.8.0_162-b12) 是Java运行时环境(即JRE)的版本信息.

(2) 如果OpenJDK, 显示信息为:

[root@localhost ~]# java -version
openjdk version "1.8.0_144"
OpenJDK Runtime Environment (build 1.8.0_144-b01)
OpenJDK 64-Bit Server VM (build 25.144-b01, mixed mode)

## 对象与类

Java是一门面向对象很强的语言

### 基本特点

return 只能返回一个对象，python可以返回多个，面向对象体现更明显，强类型
面向对象部分是Java核心和难点部分，不过语言还是要在实践中学习，而实践又会用框架，等代码感熟练了，慢慢的去看源码的时候，再来体会面向对象更容易掌握，初期可以跳过繁琐的概念，因为看了也记不住。
面向对象集中在7，8，9，10章节(Java编程思想)，有其它语言的基础可以快速过一遍，先理解11章后的内容


对象成员不进行初始化会设定默认值(原始类型才这样，引用类型都是null)，不过对于局部变量不适用，也就是在方法内的变量都必须初始化。

面向对象：字段或数据成员，方法，Python中叫做属性。

方法签名，可能在Python中不怎么提及这个概念，由于Java是静态语言，所以方法签名唯一确定方法。

java.lang 是一个类库，每个Java文件都会默认导入它。

main() 方法的参数是一个String对象的数组，以及一个args，一般都要写这两个，否则编译器报错，因为args要用来存储命令行参数。

注释文档：javadoc是JDK安装的一部分，用于提取注释的工具。该工具提取注释，可以生产html文件。可以对工具输出风格做调整，通过编写自己的被称为 “doclets” 的javadoc处理器来实现。javadoc 有特定的语法。

所有的javadoc语法只能在 “/**  */” 注释中出现，“//” 是不可以的。

使用方式：嵌入HTML，使用文档标签。

### 原始类型和封装类

引用类型和原始类型（或内置类型）。比如:Int是java的原始数据类型，Integer是java为int提供的封装类

8种基本类型

整型：byte 8, short 16, int 32, long 64

字符型：char
char类型是一个单一的 16 位 Unicode 字符；
最小值是 \u0000（即为0）；
最大值是 \uffff（即为65,535）；
char 数据类型可以储存任何字符；
例子：char letter = 'A';。

浮点型：float 32, double 64
布尔型：boolean

  原始类型           封装类   
  boolean           Boolean   
  char              Character   
  byte              Byte   
  short             Short   
  int               Integer   
  long              Long   
  float             Float   
  double            Double  

引用类型和原始类型的区别:

1. 两者的初始化方式不同

```java
int i = 5;                       // 原始类型
Integer j = new Integer(10);     // 对象引用  java 1.5以后支持自动装箱所以   Integer j = 10; 也可以
// 使用原始类型无须调用 new，也无须创建对象。这节省了时间和空间。混合使用原始类型和对象也可能导致与赋值有关的意外结果。 
```

2. 原始类型是类，引用类型是对象

原始类型大小比较用"==", 引用类型大小比较用"equals"

3. 引用类型可以被序列化，原始类型不行。

4. 引用类型提供的方法可以灵活转换，可以扩展，原始类型不行

5. 在集合类中只能使用引用类型，不能使用原始类型

6. 原始类型没有null的概念，引用类型有，某些情况下需要辨别某个参数是否被初始化了，如果使用原始类型，那么0的值不知道是初始值还是没有初始化系统自动给的。

7. 有些时候必须要用封装类

比如你要用request.setAttribute(String key ,Object value);这个方法时，第二个参数为Object类型，而你要放的是一个整数的时候，那就只能放Integer不能放int。

总结:

原始类型和封装类型的行为完全不同，并且它们具有不同的语义。引用类型和原始类型具有不同的特征和用法，它们包括：大小和速度问题，这种类型以哪种类型的数据结构存储，当引用类型和原始类型用作某个类的实例数据时所指定的缺省值。对象引用实例变量的缺省值为null，而原始类型实例变量的缺省值与它们的类型有关。

int(原始类型)   一般做为数值参数就够了   
integer (封装类型)  一般做类型转换的时候用的较

### 大数值

java.math中的 `BigInteger` 和 `BigDecimal` 可以处理包含任意长度数字序列的数值
BigInteger 任意精度的整数运算
BigDecimal 任意精度的浮点数运算

### 算术运算的强制转换

直接常量可以添加标识符，使表达更加清晰
类型转换操作符 long i = (long) j  将整形j转换成长整形并赋值给i，转换总是截尾，要想四舍五入需要使用round库
提升：对基本类型执行按位运算或算术运算，只要类型比int小（比如char,byte,short），那么在运算之前，会自动转换成int。较大的数据类型决定了结果，比如double和float相乘，结果是double
sizeof：c,c++中用来计算数据占的字节，这导致移植代码很头疼，不同的处理器对数据存储所占的字节是不一样的，而Java不会这样，它的数据类型在所有机器上都是一样的

### 初始化与清理

构造器：构造器的命名和类名相同，可以带访问修饰符，不能有返回值。

方法重载：构造器也可以方法重载，方法重载要求函数名相同，参数不一样，参数的顺序不一样也是方法重载，但是一般不建议这么做。一般动态语言不需要方法重载。方法重载一定要写的明确，这样编译器在调用方法的时候才知道是调用哪个方法。

缺省构造器：构造器可以不提供，编译器默认创建一个，这个时候构造器没做任何事情

this关键字：通常不需要显示的写出它来，和python不一样，另外它是关键字

垃圾回收：

1. 对象可能不被垃圾回收 
2. 垃圾回收并不等于析构

### 内部类

内部类是指在一个外部类的内部再定义一个类。内部类作为外部类的一个成员，并且依附于外部类而存在的。内部类可为静态，可用protected和private修饰（而外部类只能使用public和缺省的包访问权限）。内部类主要有以下几类：成员内部类、局部内部类、静态内部类、匿名内部类

内部类的共性:

1. 内部类仍然是一个独立的类，在编译之后内部类会被编译成独立的.class文件，但是前面冠以外部类的类名和$符号

2. 内部类不能用普通的方式访问

3. 内部类声明成静态的，就不能随便的访问外部类的成员变量了，此时内部类只能访问外部类的静态成员变量

4. 外部类不能直接访问内部类的的成员，但可以通过内部类对象来访问

内部类是外部类的一个成员，因此内部类可以自由地访问外部类的成员变量，无论是否是private的。
因为当某个外围类的对象创建内部类的对象时，此内部类会捕获一个隐式引用，它引用了实例化该内部对象的外围类对象。通过这个指针，可以访问外围类对象的全部状态，主要是通过以下几步做到的:

1 编译器自动为内部类添加一个成员变量，这个成员变量的类型和外部类的类型相同， 这个成员变量就是指向外部类对象的引用

2 编译器自动为内部类的构造方法添加一个参数，参数的类型是外部类的类型， 在构造方法内部使用这个参数为1中添加的成员变量赋值

3 在调用内部类的构造函数初始化内部类对象时，会默认传入外部类的引用

那静态内部类与普通内部类有什么区别呢？
问得好，区别如下：

1 静态内部类不持有外部类的引用 在普通内部类中，我们可以直接访问外部类的属性、方法，即使是private类型也可以访问，这是因为内部类持有一个外部类的引用，可以自由访问。而静态内部类，则只可以访问外部类的静态方法和静态属性（如果是private权限也能访问，这是由其代码位置所决定的），其他则不能访问。

2 静态内部类不依赖外部类 普通内部类与外部类之间是相互依赖的关系，内部类实例不能脱离外部类实例，也就是说它们会同生同死，一起声明，一起被垃圾回收器回收。而静态内部类是可以独立存在的，即使外部类消亡了，静态内部类还是可以存在的。

3 普通内部类不能声明static的方法和变量 普通内部类不能声明static的方法和变量，注意这里说的是变量，常量（也就是final static修饰的属性）还是可以的，而静态内部类形似外部类，没有任何限制。

为什么普通内部类不能有静态变量呢？

1 成员内部类 之所以叫做成员 就是说他是类实例的一部分 而不是类的一部分

2 结构上来说 他和你声明的成员变量是一样的地位 一个特殊的成员变量 而静态的变量是类的一部分和实例无关

3 你若声明一个成员内部类 让他成为主类的实例一部分 然后又想在内部类声明和实例无关的静态的东西 你让JVM情何以堪啊

4 若想在内部类内声明静态字段 就必须将其内部类本身声明为静态

例子

```java
package interior;

public class Interior {

    private Integer a = 20;

    Test test;

    Interior(Integer a) {
        this.a = a;
    }

    class Test {
        public void func1() {
            System.out.println(Interior.this.a);
        }
    }

    public void func() {
        Test test = new Test();
        test.func1();
    }


    public static void main(String[] args) {
        Interior integer = new Interior(20);
//        Test test = new Test(); 内部类依托于当前实例，不能在这里创建实例，如果是静态内部类就可以了
    }
}
```

### 面向对象

类之间的关系: 继承，接口实现，依赖，聚合，关联，直接关联

方法签名: 只有方法名，参数类型才能描述方法签名，不包含返回类型，所以不能定义两个返回类型不同方法名参数类型相同的方法

在构造器中调用this()将调用另外的构造器，根据参数来决定

this的常见用法：调用构造器，直接this()。调用方法，指代当前调用对象。由于大括号内封闭作用域，如果形参定义了和对象成员同名的属性，直接使用该名称无法取到对象成员，此时应该用this关键字。

静态块的初始化
```java
private static int nextId;

static
{
    Random generator = new Random();
    nextId = generator.nextInt(1000);
}
```

### finalize方法

可以为类添加finalize方法，在垃圾回收器清除对象之前调用

### 包

补充一个知识点，静态导入: 可以导入静态方法和静态域 `import static java.lang.System.out;` 这样`out.println()`就能打印内容了

### 访问控制

java后缀的源代码通常称为编译单元，每个编译单元内只能有一个public类。

代码组织：当编译一个java后缀文件时，文件的每个类都会生成.class文件。

关于package语句，必须是文件中除注释外的第一句代码。包命名规则全部使用小写字母，包括中间的字也是如此。导包用的星号如果两个包包含同一个类，那么这肯定是有问题的，但是只要不写调用的代码编译器是不会报错的，这种情况应该用完整路径来引用对象。

理解编译单元，默认包，friendly权限是java的默认权限，也称作包（package）访问权限

public private

1. 默认包：同一个包下的两个类，可以做到A类访问B类的未修饰方法，如果C类和A类不属于同一个包，那么C类的方法要修饰为public，A类才能访问到。这种默认的机制保护了包下的类，但是最好做修饰，在IDEA中，你不对类的方法做修饰，会有告警级别的提示，因为默认的包访问权限可以访问包，对不属于同一个包的访问做保护，但是最好考虑清楚，然后合理运用private。

2. 另外即使使用了private修饰，仍然可以在当前类定义新的方法去访问private，所有不能因为在类中某个对象的引用是private，就认为其他的对象无法拥有该对象的public引用

protected: 受保护的

一个类继承了另一个包中的类，那么唯一可访问的成员就是源包的public成员。（如果继承在同一个包中，就可以访问所有的拥有包访问权限的成员）。基类的创建者希望某个成员，把他的访问权限给予派生类而不是所有类，这时候就可以使用protected，protected也提供包访问权限，同一包内的其它类可以访问protected元素。这是为了在拥有包访问权限下，为了让跨包继承类也能访问的一种做法，否则就要用public来修饰。

`访问权限的控制常被称为是具体实现的隐藏`

类的访问权限：一个包下的类，要遵守他的文件名定义规则，然后只能有一个public类，可以把public去掉，这样该类就只有包访问权限，其它包即使导入他，也不能访问，通过也很少这么做。另外当不用public修饰类时，类命可以不和文件名一致。

再次强调，类前面不加修饰，权限就是包访问权限，当前包内的其它类可以访问，跨包不行。

### 复用类

组合，继承，代理

如果子类的构造器没有显示地调用超类的构造器，则将自动地调用超类默认(没有参数)的构造器。如果超类没有不带参数的构造器，并且在子类的构造器中又没有显示地调用超类的其他构造器，则Java编译器报错。

每一个非基本类型对象都有一个toString方法，类似python的 `__str__`

初始化，继承的基类初始化的时候，默认会调用基类的构造方法，构建过程是从基类“向外”扩散的，当然这只能调用默认不带参数的构造器，带参数的构造器需要显示的调用super方法。

```java
class Cleanser {
    Cleanser(int i) {
        System.out.println(i);
    }
    Cleanser() {
        System.out.println("123");
    }
}

public class Detergent extends Cleanser {
    Detergent() {
        // super(321)
        System.out.println("abc");
    }
    public static void main(String[] args) {
        Detergent x = new Detergent();
        System.out.println("Testing base class");
    }
}
```

如果去掉Cleanser类的默认构造函数，IDEA会警告，因为找不到满足条件的构造函数，编译代码会去调用需要传递参数的构造函数导致报错。注释部分为显示调用。

名称屏蔽：如果Java的基类拥有某个已被多次重载的方法名称，那么在导出类中重新定义该方法名称并不会屏蔽在基类中的任何版本，这一点与C++不同`（如果C++要这么做需要屏蔽基类方法，另外本书比较旧了，不排除c++做了改动）`

```java
class Homer {
    char doh(char c) {
        System.out.println("doh(char)");
        return 'd';
    }

    float doh(float f) {
        System.out.println("doh(float)");
        return 1.0f;
    }
}

class Milhouse {
}

class Bart extends Homer {
    void doh(Milhouse m) {
        System.out.println("doh(Milhouse)");
    }
}

public class Hide {
    public static void main(String[] args) {
        Bart b = new Bart();
        b.doh(1);
        b.doh('x');
        b.doh(1.0f);
        b.doh(new Milhouse());
    }
}
// doh(float)
// doh(char)
// doh(float)
// doh(Milhouse)
```

向上转型：基类A，有一个方法，参数类型为A的引用，导出类B，调用A的方法，传递参数为B的实例，这似乎和强类型语言Java违背，但在继承中是可以的，你需要认识到B对象同样也是一种A对象，这种将B的引用转换为A的引用的动作，称为向上转型。父类引用变量可以引用子类对象。

final:

1. 允许空白final，但是在构造函数中必须进行初始化。
2. final 参数，可以用来修饰参数。被修饰的参数不能在方法中去修改它。
3. 修饰方法，防止方法被继承类修改。
4. 类修饰，那么该类无法被继承。类方法都会隐式的指向final。

private 和 final: 类中的private方法都隐式的指定为final。可以对private添加final，但这并不能给该方法增加任何额外的意义。 “覆盖”只有在某方法是基类的接口的一部分才会出现，即`必须能将一个对象向上转型`为它的基本类型并调用相同的方法。如果某方法为private，它就不是基类接口的一部分，用private修饰的方法在基类中同名方法不是方法覆盖，而是生成一个新的方法。
static、final、static final的区别(转自：http://blog.csdn.net/qq1623267754/article/details/36190715)


1. final 
 
final类不能被继承，没有子类，final类中的方法默认是final的
final方法不能被子类的方法复盖，但可以被继承
final成员变量表示常量，只能被赋值一次，赋值后不能再被改变
final不能用于修饰构造方法
private不能被子类方法覆盖，private类型的方法默认是final类型的

final修饰的变量有三种：静态变量、实例变量和局部变量，分别表示三种类型的常量。
注意：final变量定义的时候，可以先声明，而不给初值，这中变量也称为final空白，无论什么情况，编译器都确保空白final在使用之前必须被初始化。

final 关键字只是表示存储在变量中的对象的引用不会再指向其它对象，对象本身可以被修改(限于可更改对象)

2. static
 
static表示“全局”或者“静态”的意思，用来修饰成员变量和成员方法，也可以形成静态static代码块，但是Java语言中没有全局变量的概念。
 
被static修饰的成员变量和成员方法独立于该类的任何对象。也就是说，它不依赖类特定的实例，被类的所有实例共享。只要这个类被加载，Java虚拟机就能根据类名在运行时数据区的方法区内定找到他们。因此，static对象可以在它的任何对象创建之前访问，无需引用任何对象。
 
用public修饰的static成员变量和成员方法本质是全局变量和全局方法，当声明它类的对象市，不生成static变量的副本，而是类的所有实例共享同一个static变量。
 
- 类成员变量
  - 静态变量（类变量）: static修饰
  - 实例变量      : 无static修饰
- 局部变量
 
 
3. static和final一起使用
 
static final用来修饰成员变量和成员方法，可以理解为“全局变量，类常量，静态常量”

例如: System类的 `public static final PrintStream out = ...` 调用 System.out

对于变量，表示一旦给值就不可修改，并且通过类名可以访问。
对于方法，表示不可覆盖，并且可以通过类名直接访问。
 
注意：
对于被static和final修饰过的实例常量，实例本身不能再改变了，但对于一些容器类型（比如，ArrayList、HashMap）的实例变量，不可以改变容器变量本身，但可以修改容器中存放的对象。

`继承与初始化`

```java
class Insect {
    private int i = 9;
    protected int j;
    Insect() {
        System.out.println("i = " + i + ", j= " + j);
        j = 39;
    }
    private static int x1 = printInt("static Insecr.x1 initialized");
    static int printInt(String s) {
        System.out.println(s);
        return 47;
    }
}

public class Beetle extends Insect {
    private int k = printInt("Beetle.k initialized");
    public Beetle () {
        System.out.println("k = " + k);
        System.out.println("j = " + j);
    }
    private static int x2 = printInt("static Beetle.x2 initialized");
    public static void main(String[] args) {
        System.out.println("Beetle constructor");
        Beetle b = new Beetle();
    }
}
//static Insecr.x1 initialized
//static Beetle.x2 initialized
//Beetle constructor
//i = 9, j= 0
//Beetle.k initialized
//k = 47
//j = 39
```

理解以上输出结果。

1. 每个类的编译代码都存在于它自己的独立文件夹中。
2. 该文件只在需要使用程序代码时才会被加载。
3. 一般来说类的代码在初次使用时才会加载，这通常指加载发生于创建类的第一个对象之时（但是访问static域或static方法时，也会加载。构造器也是static方法，它没有显示的表示出来，更准确的说，类是在其任何static成员被访问时才加载的）
4. 按照继承先加载对象，继承最顶层的类先被加载，然后是下面的类。然后创建对象，基本类型设置为默认值，对象的引用设置为null（通常是将对象内存二进制设置为零），然后是构造器调用。

### 多态

Polymoph 多态

1. 到底什么是多态呢？

- 官方说：
接口的多种不同的实现方式即为多态。
多态性是允许你将父对象设置成为一个或更多的他的子对象相等的技术。
我们在程序中定义的引用变量所指向的具体类型和通过该引用变量的方法调用在编程的时候并不确定，当处于运行期间才确定。就是这个引用变量究竟指向哪一个实例对象，在编译期间是不确定的，只有运行期才能确定，这样不用修改源码就可以把变量绑定到不同的类实例上，让程序拥有了多个运行状态，这就是多态。

- 说人话：
允许将子类类型的指针赋值给父类类型的指针，把不同的子类对象都当作父类来看。比如你家有亲属结婚了，让你们家派个人来参加婚礼，邀请函写的是让你爸来，但是实际上你去了，或者你妹妹去了，这都是可以的，因为你们代表的是你爸，但是在你们去之前他们也不知道谁会去，只知道是你们家的人。可能是你爸爸，可能是你们家的其他人代表你爸参加。这就是多态。

多态又分为 编译时多态和运行时多态。
编译时多态：比如重载
运行时多态：比如重写

2. 多态的实现机制

- 简单版本：

原理也很简单，父类或者接口定义的引用变量可以指向子类或者具体实现类的实例对象，由于程序调用方法是在运行期才动态绑定的，那么引用变量所指向的具体实例对象在运行期才确定。所以这个对象的方法是运行期正在内存运行的这个对象的方法而不是引用变量的类型中定义的方法。

- 术语版本：

我们将引入Java静态分派和动态分派这个概念。

静态分派:所有依赖静态类型来定位方法执行版本的分派动作。静态分派发生在编译阶段，因此确定静态分派的动作实际上不是由虚拟机来执行的，而是由编译器来完成。（编译时多态）
动态分派：在运行期根据实际类型确定方法执行版本的分派动作。（运行时多态）


在面向对象程序设计语言中，多态是继数据抽象和继承之后的第三种基本特征。

“封装”通过合并特征和行为来创建新的数据类型。“实现隐藏”则通过将细节“私有化”把接口和实现分离开来。多态的作用是消除类型之间的耦合关系。

方法调用绑定：在程序执行前就把方法同相关联的方法主体关联起来称为前期绑定，与之相对的就是后期绑定，就是在运行时根据对象类型进行绑定。所有编译器需要有一种机制在运行时判断对象类型。Java除了static和final（private也是final）之外，其它都是后期绑定。使用final就可以告诉编译器关系动态绑定，一定程度优化代码，不过完全没有这个必要。
Java中的所有方法都是通过动态绑定来实现多态的。

需要注意私有方法，确定你是要覆盖还是重载。

多态存在的三个必要条件
1. 继承
2. 重写
3. 父类引用指向子类对象

当使用多态方式调用方法时，首先检查父类中是否有该方法，如果没有，则编译错误；如果有，再去调用子类的同名方法。

动态链接: 当父类中的一个方法只有在父类中定义而在子类中没有重写的情况下，才可以被父类类型的引用调用； 对于父类中定义的方法，如果子类中重写了该方法，那么父类类型的引用将会调用子类中的这个方法，这就是动态连接。

协变返回类型: 在Java1.4及以前，子类方法如果要覆盖超类的某个方法，必须具有完全相同的方法签名，包括返回值也必须完全一样。Java5.0放宽了这一限制，只要子类方法与超类方法具有相同的方法签名，或者子类方法的返回值是超类方法的子类型，就可以覆盖。

"协变返回(covariant return)"，仅在subclass（子类）的返回类型是superclass（父类）返回类型的extension（继承）时才被容许。

{% blockquote %}
方法是放在代码区(code seg)里面的，里面的方法就是一句句代码。
因此当使用pet引用去访问父类对象的方法时，首先是找到这个父类对象，然后看看它里面的方法到底在哪里存着，找到那个方法再去执行。
这里头就比较有意思了，code seg里面有很多个enjoy方法，有父类的enjoy()方法，也有子类重写了从父类继续下来的enjoy()方法，那么调用的时候到底调用的是哪一个呢？是根据谁来确定呢？

注意：这是根据你实际当中的对象来确定的，你实际当中new出来的是谁，就调用谁的enjoy方法，当你找这个方法的时候，通过pet引用能找得到这个方法，但调用代码区里面的哪一个enjoy方法不是通过引用类型来确定的，如果是通过引用类型pet来确定，那么调用的肯定是Animal的enjoy()方法，可是现在是根据实际的类型来确定，我们的程序运行以后才在堆内存里面创建出一只Cat，然后根据你实际当中new出来的类型来判断我到底应该调用哪一个enjoy()方法。如果是根据实际类型，那么调用的就应该是Cat的enjoy()方法。如果是根据引用类型，那么调用的就应该是Animal的enjoy()方法。

现在动态绑定这种机制指的是实际当中new的是什么类型，就调用谁的enjoy方法。所以说虽然你是根据我父类里面的enjoy方法来调用，可是实际当中却是你new的是谁调用的就是谁的enjoy()方法。

即实际当中调用的却是子类里面重写后的那个enjoy方法。

当然，讲一点更深的机制，你实际当中找这个enjoy方法的时候，在父类对象的内部有一个enjoy方法的指针，指针指向代码区里面父类的Animal的enjoy方法，只不过当你new这个对象的时候，这个指针随之改变，你new的是什么对象，这个指针就指向这个对象重写后的那个enjoy方法，所以这就叫做动态绑定。
只有在动起来的时候，也就是在程序运行期间，new出了这个对象了以后你才能确定到底要调用哪一个方法。我实际当中的地址才会绑定到相应的方法的地址上面，所以叫动态绑定。
调这个方法的时候，只要你这个方法重写了，实际当中调哪一个，要看你实际当中new的是哪个对象，这就叫多态，也叫动态绑定。

动态绑定带来莫大的好处是使程序的可扩展性达到了最好，我们原来做这个可扩展性的时候，首先都是要在方法里面判断一下这只动物是哪一类里面的动物，通过if (object instanceof class)这样的条件来判断这个new出来的对象到底是属于哪一个类里面的，如果是一只猫，就调用猫的enjoy方法，如果是一条狗，就调用狗的enjoy方法。

如果我现在增加了一个Bird类，那么扩展的时候，你又得在方法里面写判断这只鸟属于哪一个类然后才能调用这只鸟的enjoy方法。每增加一个对象，你都要在方法里面增加一段判断这个对象到底属于哪个类里面的代码然后才能执行这个对象相应的方法。

即每增加一个新的对象，都要改变方法里面的处理代码，而现在，你不需要再改变方法里面的处理代码了，因为有了动态绑定。

你要增加哪一个对象，你实际当中把这个对象new出来就完了，不再用去修改对象的处理方法里面的代码了。也就是当你实际当中要增加别的东西的时候，很简单，你直接加上去就成了，不用去改原来的结构，你要在你们家大楼的旁边盖一个厨房，很简单，直接在旁边一盖就行了，大楼的主要支柱什么的你都不用动，这就可以让可扩展性达到了极致，这就为将来的可扩展打下了基础，也只有动态绑定（多态）这种机制能帮助我们做到这一点——让程序的可扩展性达到极致。因此动态绑定是面向对象的核心，如果没有动态绑定，那么面向对象绝对不可能发展得像现在这么流行，所以动态绑定是面向对象核心中的核心。

总结动态绑定（多态）：动态绑定是指在“执行期间”（而非编译期间）判断所引用的实际对象类型，根据其实际的类型调用其相应的方法。所以实际当中找要调用的方法时是动态的去找的，new的是谁就找谁的方法，这就叫动态绑定。动态绑定帮助我们的程序的可扩展性达到了极致。

虽然及其的啰嗦，这也是先入为主的影响吧，如果你先学的python，你会觉得这是理所当然的。
{% endblockquote %}


多态和普通引用代码举例

```java
package polymoph;

public class SupperParents {
    public void supperFunc() {
        System.out.println("supperFunc");
    }
}



package polymoph;

public class Parents extends SupperParents {

    public String name = "123456";
    public String name2 = "qweasd";

    public void func1() {
        System.out.println("parents func1");
    }

    public void parentsFunc1() {
        System.out.println("parents parentsFunc1");
    }
}



package polymoph;

public class Children extends Parents {

    public String name = "123";
    public String name1 = "mnb";

    @Override
    public void func1() {
        System.out.println("children func1");
    }

    public void func2() {
        System.out.println("children func2");
    }

    public static void main(String[] args) {
        /**
         * 多态 方法 向上转型
         * 1. 对于方法，只能调用父类中定义的，如果这个方法子类重写了，实际执行使用子类的，如func1
         * 2. 如果方法子类没有，调用父类的，如parentsFunc1
         * 3. 不能调用只有子类才有的方法，如func2
         *
         * 多态 属性
         * 1. 属性则不会这样，只能调用父类的属性，name属性子父类都有，使用父类的
         * 2. 子类才有的属性，父类无法调用，如name1
         */
        Parents children = new Children();
        children.func1(); // children func1
        children.parentsFunc1(); // parents parentsFunc1
        // children.func2(); 无法调用
        System.out.println(children.name); // 123456
        // System.out.println(children.name1); 无法调用
        children.supperFunc();
        // Parents 继承类 SupperParents 的方法，情况参照 2，你就把supperFunc当成是 Parents 类的方法，
        // 因为supperFunc是Parents通过继承得到的
        
        /**
         * 使用对象自身的引用的情况就简单多了，自己有的属性和方法就调用自己的，否则看父类有没有
         */
        Children children1 = new Children();
        children1.func1(); // children func1
        children1.parentsFunc1(); // parents parentsFunc1
        children1.func2(); // children func2
        children1.supperFunc(); // Parents 继承类的方法
        System.out.println(children1.name); // 123
        System.out.println(children1.name1); // mnb
        System.out.println(children1.name2); // qweasd
    }
}
```


### Class类

Java程序在运行时，Java运行时系统一直对所有的对象进行所谓的运行时类型标识，即所谓的RTTI(Run-Time Type Identification)。

这项信息纪录了每个对象所属的类。虚拟机通常使用运行时类型信息选准正确方法去执行，用来保存这些类型信息的类是Class类。Class类封装一个对象和接口运行时的状态，当装载类时，Class类型的对象自动创建。

说白了就是：

Class类也是类的一种，只是名字和class关键字高度相似。Java是大小写敏感的语言。

Class类的对象内容是你创建的类的类型信息，比如你创建一个shapes类，那么，Java会生成一个内容是shapes的Class类的对象

Class类的对象不能像普通类一样，以 new shapes() 的方式创建，它的对象只能由JVM创建，因为这个类没有public构造函数

Class类的作用是运行时提供或获得某个对象的类型信息，和C++中的typeid()函数类似。这些信息也可用于反射。

```java
public static void main(String[] args) throws ClassNotFoundException, IllegalAccessException,            InstantiationException, NoSuchMethodException {
        // Class类的作用是运行时提供或获得某个对象的类型信息

        // 获取Class对象的方式
        Class class_user1 = Class.forName("ioclearn.User");

        User instance_user = new User();
        Class class_user2 = instance_user.getClass();

        Class class_user3 = User.class;

        // 使用Class类的对象来生成目标类的实例
        Object user = class_user1.getMethod("Display");
        Object newShape = class_user3.newInstance();

        // newInstance返回的对象，只能是Object类型

        // 利用泛型
        Class obj1 = int.class;
        Class<Integer> obj2 = int.class;
        obj1 = double.class;
        // obj2=double.class; 错误

        Class<? super Double> obj3 = Double.class;
        // Class<Number> obj3 = Double.class; // 这里很特殊，不能直接用超类的引用，要用<? super Double>，记住就行了
        obj3 = Number.class;

    }
```

## 接口

接口和内部类为我们提供了一种将接口与实现分离的更加结构化的方法

抽象类或抽象方法：abstract定义抽象方法，如果一个类包含一个抽象方法，必须修饰为抽象类。在C++中，这相当于虚函数，纯虚函数。

接口使得抽象的概念更向前迈进了一步，接口产生一个完全抽象的类，允许创建者确定方法名，列表参数，返回类型，不提供方法体。

接口中的方法默认是public的，接口类默认是包访问权限。

通过关键字implements(实现)来像继承一样声明当前类是哪个接口的实现。

```java
interface B {

}
interface A extends B {

}
// 接口继承
```

```java
class A implements B
// 普通的实现
```

```java
class A implements B, C, D
// 实现来自多个接口
```

```java
class A extends B implements C, D {}
// 实现了多继承，也避免了钻石继承问题
```

接口适配：允许一个接口有多个不同的实现

接口的域：由于接口的任何域都是final和static，历史代码会用接口来做常量，Java SE5 后，一般用枚举enum来代替了

嵌套接口：接口可以嵌套在类中，可能这种运用有点少见，要知道可以这么做，遇到了可以来看看书

接口特性：接口不能被实例化，可以声明接口类型的变量，接口可以包含常量

在JavaSE8中，可以在接口中定义默认方法，default关键字修饰。接口的在实现的时候，默认方法可以不用覆盖

特点:

1. 接口不能用于实例化对象。
1. 接口没有构造方法。
1. 接口中所有的方法必须是抽象方法。
1. 接口不能包含成员变量，除了 static 和 final 变量。
1. 接口不是被类继承了，而是要被类实现。
1. 接口支持多继承。

- 接口并不是类，编写接口的方式和类很相似，但是它们属于不同的概念。类描述对象的属性和方法。接口则包含类要实现的方法。
- 除非实现接口的类是抽象类，否则该类要定义接口中的所有方法。
- 接口无法被实例化，但是可以被实现。一个实现接口的类，必须实现接口内所描述的所有方法，否则就必须声明为抽象类。另外，在 Java 中，接口类型可用来声明一个变量，他们可以成为一个空指针，或是被绑定在一个以此接口实现的对象。

- 接口中每一个方法也是隐式抽象的,接口中的方法会被隐式的指定为 public abstract（只能是 public abstract，其他修饰符都会报错）。
- 接口中可以含有变量，但是接口中的变量会被隐式的指定为 public static final 变量（并且只能是 public，用 private 修饰会报编译错误）。
- 接口中的方法是不能在接口中实现的，只能由实现接口的类来实现接口中的方法。

## 抽象类

abstract 修饰抽象类或抽象方法

特点:

1. 方法和类都可以被修饰为抽象
2. 只要类中有一个方法被修饰为抽象，这个类就必须加上抽象修饰
3. 抽象方法不能有方法体
4. 抽象类不能被实例化
5. abstract interfact 可以修饰接口，不过其实从本质上来说接口就是抽象的，这么写不会报错，但是没意义(具体没考证过，不过没有这么写的，写了也不会报错)
6. 由于抽象类不能被实例化，它都是用来被继承的，但是它本身也有类的大多数功能，比如成员变量，自己的方法，构造函数等，抽象类的构造函数由子类去调用
7. 不能有抽象构造方法或抽象静态方法

第6点举例

```java
package polymophIntface;

public abstract class BaoMa implements Car {

    private String name;
    private Integer price;

    BaoMa(String name, Integer price) {
        this.name = name;
        this.price = price;
    }

    @Override
    abstract public String getName();

    @Override
    public Integer getPrice() {
        return price;
    }
}

package polymophIntface;

public class QQCar extends BaoMa {
    QQCar(String name, Integer price) {
        super(name, price); 
        // 抽象类有构造函数，其实结合前面聊到的构造函数的东西，就能理解抽象类需要调用构造函数对成员变量赋值
        // 如果是无参构造器，当然就不需要子类去帮抽象类初始化了
    }

    @Override
    public String getName() {
        return null;
    }
}
```


## 接口和抽象类

1. 抽象类中的方法可以有方法体，就是能实现方法的具体功能，但是接口中的方法不行。
2. 抽象类中的成员变量可以是各种类型的，而接口中的成员变量只能是 public static final 类型的。
3. 接口中不能含有静态代码块以及静态方法(用 static 修饰的方法)，而抽象类是可以有静态代码块和静态方法。
4. 一个类只能继承一个抽象类，而一个类却可以实现多个接口。

{% blockquote %}
注：JDK 1.8 以后，接口里可以有静态方法和方法体了，静态方法直接用接口类名调用，也不需要实现
{% endblockquote %}

```java
static String hello() {
    return "hello";
}
```

java 8 以后还可以定义默认方法，不过这个默认方法要接口的实现类才可以调用

```java
default void stu() {
        System.out.println("stu");
    }
```



## 容器和基本对象

容器相关知识点

### 容器类(持有对象)

容器类是整个语言最重要的一部分，或者称为集合框架，记住，定义容器类的时候，最好使用泛型

Java集合主要有两个接口派生而出：Collection和Map，这个两个接口是Java集合框架的根接口

Collection: 容器类型对象的父接口

容器类：基本类型是List，set，Queue和Map，这些对象类型也称为集合类，但由于Java的类库使用了Collection这个名字来指代该类库的一个特殊子集，所有更广泛的称为容器

Map：也被称为关联数组

List：基本的ArrayList，它长于随机访问元素，但是在List的中间插入和移除元素时较慢。LinkedList，它通过代价较低的在List中间进行的插入和删除操作，提供了优化的顺序访问，在随机访问较慢，占空间也大。

迭代器：对容器的访问

Collection 是描述所有序列容器的共性的根接口

Foreach 也可以用于任何Collection对象

容器不能持有基本类型，但是自动包装机制会仔细地执行基本类型到容器中所持有的包装器类型之间的双向转换

大量的随机访问，使用ArrayList，经常从中间插入或删除元素使用LinkedList，各种Queue以及栈的行为，由LinkedList提供支持

关于Map，HashMap用于快速访问，TreeMap始终让键保持在排序状态(类似二叉树插入)，LinkedHashMap保持插入顺序，也提供散列提供快速访问的能力

### HashMap

通过h & (table.length -1)来得到该对象的保存位，而HashMap底层数组的长度总是2的n次方，这是HashMap在速度上的优化。当length总是2的n次方时，h& (length-1)运算等价于对length取模，也就是h%length，但是&比%具有更高的效率。在JDK1.8的实现中，优化了高位运算的算法，通过hashCode()的高16位异或低16位实现的：(h = k.hashCode()) ^ (h >>> 16)，主要是从速度、功效、质量来考虑的，这么做可以在数组table的length比较小的时候，也能保证考虑到高低Bit都参与到Hash的计算中，同时不会有太大的开销。

我理解了，hashmap如果直接对hash值取模结果会有明显的局部性，并且引起堆积。 解决了哈希碰撞问题，思想就是把高位和低位混合进行计算，提高分散性

### 自动装箱

```java
ArrayList<Integer> list = new ArrayList<>();

list.add(3);
```

上诉的add操作将自动的变成 `list.add(Integer.ValueOf(3));`，这种变换称之为自动装箱(autoboxing)

将一个包装器对象赋值给一个基本对象，称为自动拆箱

```java
int n = list.get(i);

int n = list.get(i).intValue();
```

装箱和拆箱是编译器认可的，而不是虚拟机。编译器在生成类的字节码时，插入必要的方法调用。虚拟机只是执行这些字节码。

### Integer的自动装箱

```java
Integer f1 = 100, f2 = 100, f3 = 150, f4 = 150;
System.out.println(f1 == f2);
System.out.println(f3 == f4);
```

上面的代码，由于是引用比较，会认为都是false，但是f1 == f2是true，原因是:

如果整型字面量的值在-128 到 127 之间，那么不会 new 新的 Integer对象，而是直接引用常量池中的 Integer 对象

具体可以追溯到源码，自动装箱后调用valueof，IntegerCache做了判断，字面量的值在-128 到 127 之间

```java
public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```

### 数组

数组声明方法：

dataType[] arrayRefVar;   // 首选的方法
 
或
 
dataType arrayRefVar[];  // 效果相同，但不是首选方法

声明后通过new关键字创建数组，完整实例：double[] myList = new double[10];

元素是10个的固定数组，元素类型是double，数组大小是固定的，不固定请使用集合框架的相关数组类

记住数组的定义是`类型`加上`[]`，double[] 作为数组类型声明，然后再写一个数组变量名，最后就组成了 dataType[] arrayRefVar

常见写法:

`int[] abc = new int[]{1, 2, 3};`
`char[] words = {'1', '2'};`
`String[] words = {"1", "2"};` // 字符和字符串是不同的

声名二维数组: `int[][] array`

打印数组: Arrays.toString(a) 该方法会将数组拼成字符串

数组值拷贝(不是引用): `int[] array = Arrays.copyOf(array1, array1.length`，可以调整长度的值，做的扩充数组，多余的元素会被赋初值，int为0，booler为false，如果是小于原长度，则截取，只拷贝前面的数值

数组排序: Arrays.sort()

### 字符串

1. 基本概念
字符串由char值序列组成，char数据类型是一个采用UTF-16编码表示Unicode码点的代码单元，大多数的常用Unicode字符串使用一个代码单元就可以表示，而辅助字符需要一对代码单元表示

码点和代码单元可能是一个需要去了解的概念:
- 码点: 就是某个任意字符在Unicode编码表中对应的代码值
- 代码单元: 是在计算机中用来表示码点的，大部分码点只需要一个代码单元表示，但是有一些是需要两个代码单元表示的

```java
public String(int[] codePoints, int offset, int count) // 可以用一个由码点值组成的数组来创建字符串
String greeting = "Hello";
int[] array = greeting.codePoints().toArray();
String str = new String(array, 0, array.length);
```


length方法返回代码单元数量，实际长度即码点数调用`int cpCount = "Hello".codePointCount(0, "Hello".length());`，由于这是一个比较简单的纯英文字符串，码点数量和代码单元都是5

获取指定位置的代码单元: `str.charAt(0)` 返回0位置的代码单元

获取指定位置的码点: `str.offsetByCodePoints(0, index); int cp = greeting.codePointAt(index);`

2. 常用方法和操作

字面量也可以调用很多方法

例子：
```java
str.length();
str.equals();

str != null && str.length() != 0 // 检查既不是null也不是空串
```

截取操作: subString(0, 3) 实例方法
join方法合并字符串，可以指定分隔符，静态方法
```java
String[] list = {"1", "2"};
s = String.join("mn", list);

s = String.join("mn", "a", "b");
```

3. 构建字符串

拼接字符串的效率是低下的，每次连接字符串都需要频繁的创建对象，可以使用StringBuilder类来实现

```java
StringBuilder builder = new StringBuilder();
builder.append(character or string);
String completedString = builder.toString();
```

StringBuilder 和 StringBuffer 两者都有相同的API，StringBuffer运行在多线程中操作，单线程用StringBuilder

## 异常

一个异常的例子：throw new NullPointerException("abc")

异常根类 Throwable

try 处理块：

```java
try {

} catch (Exception e) {

} catch (Exception2 e) {

}
```

## 泛型

泛型使用过程中，操作的数据类型被指定为一个参数，这种参数类型可以用在类、接口和方法中，分别被称为泛型类、泛型接口、泛型方法。

Java中的泛型，只在编译阶段有效。在编译过程中，正确检验泛型结果后，会将泛型的相关信息擦出，并且在对象进入和离开方法的边界处添加类型检查和类型转换的方法。也就是说，泛型信息不会进入到运行时阶段。

在JavaSE7之后的版本，构造函数中可以省略泛型类型


1. 参数化类型:

把类型当作是参数一样传递
<数据类型> 只能是引用类型

2. 相关术语：

ArrayList<E>中的E称为类型参数变量
ArrayList<Integer>中的Integer称为实际类型参数
整个称为ArrayList<E>泛型类型
整个ArrayList<Integer>称为参数化的类型ParameterizedType

1. 泛型方法

- 定义：

所有泛型方法声明都有一个类型参数声明部分（由尖括号分隔），该类型参数声明部分在方法返回类型之前
每一个类型参数声明部分包含一个或多个类型参数，参数间用逗号隔开。一个泛型参数，也被称为一个类型变量，是用于指定一个泛型类型名称的标识符
类型参数能被用来声明返回值类型，并且能作为泛型方法得到的实际参数类型的占位符
泛型方法体的声明和其他方法一样。注意类型参数只能代表引用型类型，不能是原始类型（像int,double,char的等）

泛型方法可以定义在泛型类或普通类中

- 举例：

假设当前类名是MyClass

```public static <E> void printArray(E[] inputArray)```
```public static <T> T printArray(T[] inputArray)```

可以看到和一般方法的区别就是在返回类型前面加上参数类型的定义，然后这个参数类型就可以被用在`(1.返回类型，2.方法参数类型，3.方法体内)`
调用泛型方法：`MyClass.<String>printArray(参数为String类型的数组)`

2. 泛型类

- 定义：

泛型类的声明和非泛型类的声明类似，除了在类名后面添加了类型参数声明部分
和泛型方法一样，泛型类的类型参数声明部分也包含一个或多个类型参数，参数间用逗号隔开。一个泛型参数，也被称为一个类型变量，是用于指定一个泛型类型名称的标识符。因为他们接受一个或多个参数，这些类被称为参数化的类或参数化的类型

- 举例：

```public class Box<T>```

3. 通配符类型

类型参数使用规范：
E: 表示集合元素类型
K,V: 表示表的关键字和值
T(U 或 S): 表示任意类型 

通配符类型一般是使用?代替具体的类型参数。例如 List<?> 在逻辑上是List<String>,List<Integer> 等所有List<具体类型实参>的父类
这是一个很抽象的概念，能不能学会泛型，就看能不能理解通配符

4. 类型限制

extends 关键字来限制泛型参数的超类 <T extends Comparable> 这样实际类型参数就必须是Comparable的子类
<? super T> 表示类型下界（Java Core中叫超类型限定），表示参数化类型是此类型的超类型（父类型），直至Object

5. 类型擦除

这是一个很重要的概念，无论何时定义一个泛型类型，都自动提供了一个相应的原始类型


## 反射

理解jvm和Class对象就能理解了反射

最重要的一步，获得Class对象，方式如下:

类名.class 泛型为T
getClass() 实例调用 泛型为? 
Class.forName("ioclearn.Test") 泛型为?

所以，只有`类名.class`的形式能确定类型，其它情况获取的对象，在后续的使用中要么类型转换或者用Object对象

其它方法补充:
getName() Class的实例调用，返回 字符串 ioclearn.Test

### 判断是否为某个类的实例

son instanceof Son

Student.class.isInstance(student)

### 创建实例

1. 利用newInstance创建对象：调用的类必须有无参的构造器

```java
//Class<?>代表任何类的一个类对象。
//使用这个类对象可以为其他类进行实例化
//因为jvm加载类以后自动在堆区生成一个对应的*.Class对象
//该对象用于让JVM对进行所有*对象实例化。
Class<?> c = String.class;

//Class<?> 中的 ? 是通配符，其实就是表示任意符合泛类定义条件的类，和直接使用 Class
//效果基本一致，但是这样写更加规范，在某些类型转换时可以避免不必要的 unchecked 错误。

Object str = c.newInstance();
```

2. 先通过Class对象获取指定的Constructor对象，再调用Constructor对象的newInstance()方法来创建实例。这种方法可以用指定的构造器构造类的实例。

`public Constructor<T> getConstructor(Class<?>... parameterTypes)` 观察getConstructor的方法签名，它接收Class<?>对象，这里就是和有参数的构造器要想对应，比如构造器需要`String a, Integer b`，那么getConstructor传递`String.class, Integer.class`

```java
//获取String所对应的Class对象
Class<?> c = String.class;
//获取String类带一个String参数的构造器
Constructor constructor = c.getConstructor(String.class);
//根据构造器创建实例
Object obj = constructor.newInstance("23333");
System.out.println(obj);
```

```java
import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;

class Student {
    private String a;
    public Student(String a, Integer b) {

    }

    public Student(String a) {
        this.a = a;
    }

    public Student(Boolean b) {

    }

    @Override
    public String toString() {
        return "Student{" +
                "a='" + a + '\'' +
                '}';
    }
}

public class Test2 {
    public static void main(String[] args)
            throws IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException {
        Class<Student> c = Student.class;
        Constructor<Student> constructor = c.getConstructor(String.class);
        Student student = constructor.newInstance("abc");
        System.out.println(student);
    }
}
```

### 获取方法

1. getDeclaredMethods 
返回类或接口声明的所有方法，包括公共、保护、默认（包）访问和私有方法，但不包括继承的方法
2. getMethods 
返回某个类的所有公用（public）方法，包括其继承类的公用方法
3. getMethod 
方法返回一个特定的方法，其中第一个参数为方法名称，后面的参数为方法的参数对应Class的对象

包含Declared描述的方法，获取的是自己类的，继承的没有(包含私有，公有的)，不包含的返回公有的，包括继承的(只返回公有的)
这个在获取字段中也是类似的情况

再次强调，获取字段也是类似的：

getDeclaredMethod*()获取的是类自身声明的所有方法，包含public、protected和private方法。
getMethod*()获取的是类的所有共有方法，这就包括自身的所有public方法，和从基类继承的、从接口实现的所有public方法。

### 获取构造器

getConstructor

### 获取成员变量，字段

getFiled: 访问公有的成员变量 
getDeclaredField：所有已声明的成员变量。但不能得到其父类的成员变量 

### 调用方法

这个例子包含了调用的很多情况

```java
 public static void main(String[] args)
            throws IllegalAccessException,
            InstantiationException, NoSuchMethodException, InvocationTargetException {
        Class<UserBean> userBeanClass = UserBean.class;
        // 获取该类所有的方法，包括静态方法，实例方法。
        // 此处也包括了私有方法，只不过私有方法在用invoke访问之前要设置访问权限
        // 也就是使用setAccessible使方法可访问，否则会抛出异常
        // getDeclaredMethod*()获取的是类自身声明的所有方法，包含public、protected和private方法。
        // getMethod*()获取的是类的所有共有方法，这就包括自身的所有public方法，和从基类继承的、从接口实现的所有public方法。
        // IllegalAccessException的解释是
        // * An IllegalAccessException is thrown when an application tries
        // * to reflectively create an instance (other than an array),
        // * set or get a field, or invoke a method, but the currently
        // * executing method does not have access to the definition of
        // * the specified class, field, method or constructor.
        //IllegalAccessException的解释是 就是说，当这个类，域或者方法被设为私有访问，使用反射调用但是却没有权限时会抛出异常。

        Method[] methods = userBeanClass.getDeclaredMethods(); // 获取所有成员方法
        for (Method method : methods) {
            //反射可以获取方法上的注解，通过注解来进行判断
            if (method.isAnnotationPresent(Invoke.class)) { // 判断是否被 @Invoke 修饰
                // 判断方法的修饰符是是static
                // getModifiers获取方法的修饰
                if (Modifier.isStatic(method.getModifiers())) { // 如果是 static 方法
                    //反射调用该方法
                    //类方法可以直接调用，不必先实例化
                    method.invoke(null, "wingjay", 2); // 直接调用，并传入需要的参数 devName
                } else {
                    //如果不是类方法，需要先获得一个实例再调用方法
                    //传入构造方法需要的变量类型
                    Class[] params = {String.class, long.class};
                    //获取该类指定类型的构造方法
                    //如果没有这种类型的方法会报错
                    Constructor<UserBean> constructor = userBeanClass.getDeclaredConstructor(params); // 获取参数格式为 String,long 的构造函数
                    //通过构造方法的实例来进行实例化
                    Object userBean = constructor.newInstance("wingjay", 11); // 利用构造函数进行实例化，得到 Object
                    if (Modifier.isPrivate(method.getModifiers())) {
                        method.setAccessible(true); // 如果是 private 的方法，需要获取其调用权限
                        //     Set the {@code accessible} flag for this object to
                        //     * the indicated boolean value.  A value of {@code true} indicates that
                        //     * the reflected object should suppress Java language access
                        //     * checking when it is used.  A value of {@code false} indicates
                        //     * that the reflected object should enforce Java language access checks.
                        //通过该方法可以设置其可见或者不可见，不仅可以用于方法
                        //后面例子会介绍将其用于成员变量
                        //打印结果
                        // I'm a public method
                        // Hi wingjay, I'm a static methodI'm a private method
                    }
                    method.invoke(userBean); // 调用 method，无须参数
                }
            }
        }
    }
```

### 利用反射创建数组

`import java.lang.reflect.Array;` 利用反射中提供的Array类来创建

```java
public static void main(String[] args) {
        Class<?> cls = null;
        try {
            cls = Class.forName("java.lang.String");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        Object array = Array.newInstance(cls,25);
        //往数组里添加内容
        Array.set(array,0,"hello");
        Array.set(array,1,"Java");
        Array.set(array,2,"fuck");
        Array.set(array,3,"Scala");
        Array.set(array,4,"Clojure");
        //获取某一项的内容
        System.out.println(Array.get(array,3));
        //Scala
    }
```


## 输入与输出

Scanner类实现标准输入流(可以留意构造函数，它指定了流的来源)，在调用读取输入方法的时候，会阻塞进程等待输入，得到输入后程序继续

```java
String greeting = "Hello";
Scanner in = new Scanner(System.in);
System.out.println("what is your name?");
String name = in.nextLine();
System.out.println(name);
```


## 注解

包含三种标准注解和四种元注解

标准注解，就是JDK内置的注解，位于java.lang.annotation中:
1. Override
编译器可以给你验证@Override下面的方法名是否是你父类中所有的，如果没有则报错。例如，你如果没写@Override，而你下面的方法名又写错了，这时你的编译器是可以编译通过的，因为编译器以为这个方法是你的子类中自己增加的方法
2. Deprecated 
弃用的，一个方法被加上这个注解后，在子类中重写这个方法，idea工具会把这个方法名划线，表示这个方法被弃用了，最好不要用，但是，此方法有可能在以后的版本升级中会被慢慢的淘汰(因为我查到的资料说编译器会有提示，运行了没有，JDK1.8下测试的，补充，可能是idea工具的原因，因为编译程序的时候是可以指定参数的，个人猜测idea优化了这些东西了)
3. SuppressWarnings
根据传递的参数来抑制警告

Java8新增 @FunctionalInterface
 * 此注解是 Java8 提出的函数式接口，接口中只允许有一个抽象方法
 * 加上这个注解之后，类中多一个抽象方法或者少一个抽象方法都会报错

元注解，用来注解其它注解的，自定义注解一般会用到：
1. @Documented –注解是否将包含在JavaDoc中

2. @Retention –什么时候使用该注解
定义该注解的生命周期，参数如下：
- RetentionPolicy.SOURCE : 
在编译阶段丢弃。这些注解在编译结束之后就不再有任何意义，所以它们不会写入字节码。@Override, @SuppressWarnings都属于这类注解

- RetentionPolicy.CLASS : 
在类加载的时候丢弃。在字节码文件的处理中有用。注解默认使用这种方式
  
- RetentionPolicy.RUNTIME : 
始终不会丢弃，运行期也保留该注解，因此可以使用反射机制读取该注解的信息。我们自定义的注解通常使用这种方式。

3. @Target –注解用于什么地方
表示该注解用于什么地方。`默认值为任何元素`，表示该注解用于什么地方，不能把用于字段的用在方法上，可用的ElementType参数包括

● ElementType.CONSTRUCTOR:用于描述构造器
● ElementType.FIELD:成员变量、对象、属性（包括enum实例）
● ElementType.LOCAL_VARIABLE:用于描述局部变量
● ElementType.METHOD:用于描述方法
● ElementType.PACKAGE:用于描述包
● ElementType.PARAMETER:用于描述参数
● ElementType.TYPE:用于描述类、接口(包括注解类型) 或enum声明

4. @Inherited – 是否允许子类继承该注解
@inherited注解修饰的注解@A，@A修饰某个类，则该类的子类也被@A修饰

### 注解的属性

注解的属性是注解里面使用的，注解的属性也叫做成员变量，注解只有成员变量，没有方法。
注解的成员变量在注解的定义中以“无形参的方法”形式来声明，其方法名定义了该成员变量的名字，其返回值定义了该成员变量的类型
在注解中定义属性时它的类型必须是 8 种基本数据类型外加 类、接口、注解及它们的数组，String类型也可以用

示例:
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {

    int id();

    String msg();

}
```

默认值`public int id() default -1;`

如果一个注解内仅仅只有一个名字为 value 的属性时，应用这个注解时可以直接把属性值填写到括号内，比如上面的测试注解只有id属性时`@TestAnnotation(123)`

注解没有属性可以省略括号

当注解中含有数组属性时，使用{}赋值，各个元素使用逗号分隔
定义 `String[] parentsName();` 
赋值 `@ParentsAnnotation(parentsName = {"1", "2"})`

注解的属性可以是另外一个注解

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Documented
public @interface MyAnnotation {
    String name();

    int id();

    ParentsAnnotation parentsannotation();
    // ParentsAnnotation parentsannotation() default  @ParentsAnnotation(parentsName = {"!"}, parentsAge = 1); 设置默认值，这样对方法添加注解的时候就不用赋值了
}
```

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface ParentsAnnotation {
    String[] parentsName();

    int parentsAge();
}
```

使用
```java
    @MyAnnotation(name = "x", id = 1, parentsannotation = @ParentsAnnotation(parentsName = {"1", "2"}, parentsAge = 2))
    public void test() {

    }

// annotation.parentsannotation().parentsAge() 获取到继承注解的属性
```

属性数组的元素可以是另外一个注解
`Hello[] helloArrayValue() default {@Hello,@Hello};`

以上，就是注解属性定义的各种情况


### 注解处理器

注解是一种标记，基本注解用来决定注解到什么地方，什么时候发挥作用。我们需要读取注解的工具，也就是`创建与使用注解处理器`。注解处理器是一个单独的类，通过获取被注解的类，然后调用获取注解信息的方法得到注解信息，之后进行操作。有各种反射方法来获取注解标记的信息。

第一步，当然是获取注解数据了

Class.getAnnotation(Class< A > annotationClass) 获取指定的注解，该注解可以是自己声明的，也可以是继承的
Class.getDeclaredAnnotations() 获取自己声明的注解
Class.getAnnotations() 获取所有的注解，包括自己声明的以及继承的

上面是类的注解获取方式，方法和字段的注解用对应类型即可

后两种返回的是数组，这个继承是说使用了`@Inherited`的情况，也就是说是对于`类的注解`这两个方法有区别，其它是一样的(测试是这样的)

得到注解后，获取注解标注的属性

```java
Class<Test> c = Test.class;
Method method = c.getMethod("test");
Annotation[] annotationsArray = method.getDeclaredAnnotations();
MyAnnotation annotation = (MyAnnotation) annotationsArray[0];
System.out.println(annotation.name()); // 普通属性
System.out.println(annotation.parentsannotation().parentsAge()); // 属性是注解，再去获取这个注解的属性
```


## 内置包

jdk内置包使用方法总结

### java.io.Serializable

在序列化的时候需要使用到，只有实现这个接口的类才能序列化，这个Serializable不包含任何方法，所以实现类不需要做任何处理(这里的原理待后续展开)。
运用：比如把一个普通对象通过FileOutputStream文件输出流写到文件中，然后通过FileInputStream读取对象恢复它，序列化只能处理简单的对象值，方法等复杂的成员不能被处理。

## 扩展包

扩展包用法记录

### lombok

一个插件，减少对象操作的代码编写

## 语法部分

主要是Java语言语法特点部分，和一些关键字概念

### 控制执行流程

不允许将一个数字作为布尔值使用，应该if(a==0)，for循环语法，for(int i : range(100))

for 循环的3部分组成 for(int i;i<=10;i++)，在for循环中定义的变量i作用域只在for循环内，要在循环外使用，应该一开始就声名变量

不允许在嵌套块中对已存在的变量再从声名，c++可以

while 和 do while 接收一个布尔变量作为循环执行条件

switch语法，case 和 break case标签类型可以是char，byte，short，int常量，枚举常量，SE7可以使用字符串字面量

break和continue 控制循环执行，可以加标签跳到指定标签处，在嵌套循环中发挥作用，如果没有嵌套，标签写不写都可以
```java
int a = 1;
labels:
for (int j = 1; j<=10; j += 1) {
    System.out.println(123);
    a += 1234;
    for (int ii=0; ii<=2; ii++) {
        if (a == 12345) {
            break labels;
        }
    }
}
```

### transient

Java中transient关键字的作用，简单地说，就是让某些被修饰的成员属性变量不被序列化，这一看好像很好理解，就是不被序列化，那么什么情况下，一个对象的某些字段不需要被序列化呢？如果有如下情况，可以考虑使用关键字transient修饰：

1、类中的字段值可以根据其它字段推导出来，如一个长方形类有三个属性：长度、宽度、面积（示例而已，一般不会这样设计），那么在序列化的时候，面积这个属性就没必要被序列化了；

2、其它，看具体业务需求吧，哪些字段不想被序列化；

### ++操作符

分为前缀与后缀

后缀:
```java
// 先将1与变量的值相加，将新值(11)存回变量
// 表达式返回旧值(10)
int i = 10;
System.out.println(i ++); // 输出10
```

前缀:
```java
// 先将1与变量的值相加，将新值(11)存回变量
// 表达式返回新值(11)
int i = 10;
System.out.println(++ i);
```

这个后缀的形式不太友好，如果选用后缀++, 那么表达式将返回变量的旧值，那么这一瞬间，内存将同时记录（保存）旧值和新值两个变量，其中旧值是返回后就不被使用的临时变量。但这个临时变量是不必要的，亦即“拷贝旧值的动作”以及“占用的内存”都是不必要的。所以我说，在不关心表达式的返回值时，优先用前缀++, 以避免不必要的拷贝和内存浪费。

Python里面没有++这种做法，最好Java也用 i += 1

### ...符号

```java
class Test {
    static void MyTest(String... basePackages) {
        System.out.println(basePackages.length);
    }
//    static void MyTest(String[] basePackages) {
//        System.out.println(basePackages[2]);
//    }
    public static void main(String... args) {
        MyTest(new String[]{"1", "2", "3"});
        MyTest("1", "2", "3");
    }
}
```

大致上来说 `String...` 和 `String[]` 差不多，在上面的例子中，两种传参都可以(但是不推荐这样用)，但是参数类型换成数组的时候，`MyTest("1", "2", "3");`就不行了。另外此时两种MyTest方法不能同时存在，会被认为是方法重复。

### 取余和取模

Python3中，" / "就一定表示`浮点数除法`，返回浮点结果，" // "表示`整数除法`

Java中两个整数类型相除，结果是整数，其中一个是浮点数，结果就是浮点数

上面回顾了除法的问题，在Java中 %为取余（rem），Math.floorMod()为取模（mod）

取余运算在计算商值向0方向舍弃小数位
取模运算在计算商值向负无穷方向舍弃小数位

例如： 4 / (-3) 约等于-1.3
在取余运算时候商值向0方向舍弃小数位位 -1
在取模运算时商值向负无穷方向舍弃小数位为 -2

这个概念很重要，未完待续

## JVM

栈内存：是一种连续储存的数据结构，具有先进后出的性质。
通常的操作有入栈（压栈），出栈和栈顶元素。想要读取栈中的某个元素，就是将其之间的所有元素出栈才能完成

堆内存：是一种非连续的树形储存数据结构，每个节点有一个值，整棵树是经过排序的。特点是根结点的值最小（或最大），且根结点的两个子树也是一个堆。常用来实现优先队列，存取随意。

1. 栈：为编译器自动分配和释放，如函数参数、局部变量、临时变量等等
2. 堆：为成员分配和释放，由程序员自己申请、自己释放。否则发生内存泄露。典型为使用new申请的堆内容。
除了这两部分，还有一部分是：
3. 静态存储区：内存在程序编译的时候就已经分配好，这块内存在程序的整个运行期间都存在。它主要存放静态数据、全局数据和常量。

### 类型信息

`本章重点，class后缀文件，和类加载器`

RTTI：运行时类型识别

Class对象，它是一个特殊的对象，每当编译一个新类就会产生其同名的Class对象，后缀名是Class，也就是字节码。为了生成这个类对象，运行这个程序的JVM将使用被称为“类加载器的”子系统。该部分都是运行时涉及到的概念。
所以的类对象都是在对其第一次使用的时候，动态的加载到JVM中的。使用new关键字创建类的新对象被当作对类的静态成员的引用。

## javac 和 javap

编译java文件

1. 程序中编译
java提供了JavaCompiler，我们可以通过它来编译java源文件为class文件，这个相关的类，用来在代码中编译，大致流程就是读取一个java后缀文件，把它编译成class后缀文件

2. 命令编译
classpath是什么？

在dos下编译java程序，就要用到classpath这个概念，尤其是在没有设置环境变量的时候。classpath就是存放.class等编译后文件的路径

javac：如果当前你要编译的java文件中引用了其它的类(比如说：继承)，但该引用类的.class文件不在当前目录下，这种情况下就需要在javac命令后面加上-classpath参数，通过使用以下三种类型的方法 来指导编译器在编译的时候去指定的路径下查找引用类。

(1).绝对路径：javac -classpath c:/junit3.8.1/junit.jar Xxx.java

(2).相对路径：javac -classpath ../junit3.8.1/Junit.javr Xxx.java

(3).系统变量：javac -classpath %CLASSPATH% Xxx.java (注意：%CLASSPATH%表示使用系统变量CLASSPATH的值进行查找，这里假设Junit.jar的路径就包含在CLASSPATH系统变量中)

## Java中的代码块

Java中，静态代码块、构造代码块、构造函数、普通代码块，要记住和理解它们的执行顺序

1. 静态代码块

```java
static {
    System.out.println("静态代码块");
}
```

特点: 程序启动就执行，所以不能在方法，也不能访问普通变量，只能访问静态变量

2. 构造代码块

```java
{
    System.out.println("构造代码块");
}
```

执行时机，在对象被创建的时候就会执行一次，所以可以用来统计对象创建的次数，存在多个构造代码块，按顺序执行它们

3. 构造函数

这个就不展开说了

4. 普通代码块

```java
public void sayHello(){
    {
        System.out.println("普通代码块");
    }
}
```

普通代码块和构造代码块的区别是，构造代码块是在类中定义的，而普通代码块是在方法体中定义的。且普通代码块的执行顺序和书写顺序一致

- 执行顺序

`静态代码块>构造代码块>构造函数>普通代码块`

对象的初始化顺序：

首先执行父类静态的内容，父类静态的内容执行完毕后，接着去执行子类的静态的内容，当子类的静态内容执行完毕之后，再去看父类有没有构造代码块，如果有就执行父类的构造代码块，父类的构造代码块执行完毕，接着执行父类的构造方法；父类的构造方法执行完毕之后，它接着去看子类有没有构造代码块，如果有就执行子类的构造代码块。子类的构造代码块执行完毕再去执行子类的构造方法。

总之一句话，静态代码块内容先执行，接着执行父类构造代码块和构造方法，然后执行子类构造代码块和构造方法。


##  jdk工具

| Command            | Description 
| -----------------  | :---------------: 
| appletviewer.exe   | 用于运行并浏览applet小程序。 
| apt.exe            | 注解处理工具(AnnotationProcessingTool)，主要用于注解处理。 
| extcheck.exe       | 扩展检测工具，主要用于检测指定jar文件与当前已安装的JavaSDK扩展之间是否存在版本冲突。 
| idlj.exe           | IDL转Java编译器(IDL-to-JavaCompiler)，用于为指定的IDL文件生成Java绑定。IDL意即接口定义语言(InterfaceDefinitionLanguage)。 
| jabswitch.exe      | Java访问桥开关(JavaAccessBridgeswitch)，用于启用/禁用Java访问桥。Java访问桥内置于Java7Update6及以上版本，主要为Windows系统平台提供一套访问Java应用的API。 
| jar.exe            | jar文件管理工具，主要用于打包压缩、解压jar文件。 
| jarsigner.exe      | jar密匙签名工具。 
| java.exe           | Java运行工具，用于运行.class字节码文件或.jar文件。 
| javac.exe          | Java编译工具(JavaCompiler)，用于编译Java源代码文件。 
| javadoc.exe        | Java文档工具，主要用于根据Java源代码中的注释信息生成HTML格式的API帮助文档。 
| javafxpackager.exe | JavaFX包装器，用于执行与封装或签名JavaFX应用有关的任务。 
| javah.exe          | Java头文件工具，用于根据Java类生成C/C++头文件和源文件(主要用于JNI开发领域)。 
| javap.exe          | Java反编译工具，主要用于根据Java字节码文件反汇编为Java源代码文件。 
| java-rmi.exe       | Java远程方法调用(JavaRemoteMethodInvocation)工具，主要用于在客户机上调用远程服务器上的对象。 
| javaw.exe          | Java运行工具，用于运行.class字节码文件或.jar文件，但不会显示控制台输出信息，适用于运行图形化程序。 
| javaws.exe         | JavaWebStart，使您可以从Web下载和运行Java应用程序，下载、安装、运行、更新Java应用程序都非常简单方便。 
| jcmd.exe           | Java命令行(JavaCommand)，用于向正在运行的JVM发送诊断命令请求。 
| jconsole.exe       | 图形化用户界面的监测工具，主要用于监测并显示运行于Java平台上的应用程序的性能和资源占用等信息。 
| jdb.exe            | Java调试工具(JavaDebugger)，主要用于对Java应用进行断点调试。 
| jhat.exe           | Java堆分析工具(JavaHeapAnalysisTool)，用于分析Java堆内存中的对象信息。 
| jinfo.exe          | Java配置信息工具(JavaConfigurationInformation)，用于打印指定Java进程、核心文件或远程调试服务器的配置信息。 
| jmap.exe           | Java内存映射工具(JavaMemoryMap)，主要用于打印指定Java进程、核心文件或远程调试服务器的共享对象内存映射或堆内存细节。 
| jmc.exe            | Java任务控制工具(JavaMissionControl)，主要用于HotSpotJVM的生产时间监测、分析、诊断。 
| jps.exe            | JVM进程状态工具(JVMProcessStatusTool)，用于显示目标系统上的HotSpotJVM的Java进程信息。 
| jrunscript.exe     | Java命令行脚本外壳工具(commandlinescriptshell)，主要用于解释执行javascript、groovy、ruby等脚本语言。 
| jsadebugd.exe      | Java可用性代理调试守护进程(JavaServiceabilityAgentDebugDaemon)，主要用于附加到指定的Java进程、核心文件，或充当一个调试服务器。 
| jstack.exe         | Java堆栈跟踪工具，主要用于打印指定Java进程、核心文件或远程调试服务器的Java线程的堆栈跟踪信息。 
| jstat.exe          | JVM统计监测工具(JVMStatisticsMonitoringTool)，主要用于监测并显示JVM的性能统计信息。 
| jstatd.exe         | jstatd(VMjstatdDaemon)工具是一个RMI服务器应用，用于监测HotSpotJVM的创建和终止，并提供一个接口，允许远程监测工具附加到运行于本地主机的JVM上。 
| jvisualvm.exe      | JVM监测、故障排除、分析工具，主要以图形化界面的方式提供运行于指定虚拟机的Java应用程序的详细信息。 
| keytool.exe        | 密钥和证书管理工具，主要用于密钥和证书的创建、修改、删除等。 
| kinit.exe          | 主要用于获取或缓存Kerberos协议的票据授权票据。 
| klist.exe          | 允许用户查看本地凭据缓存和密钥表中的条目(用于Kerberos协议)。 
| ktab.exe           | Kerberos密钥表管理工具，允许用户管理存储于本地密钥表中的主要名称和服务密钥。 
| native2ascii.exe   | 本地编码到ASCII编码的转换器(Native-to-ASCIIConverter)，用于"任意受支持的字符编码"和与之对应的"ASCII编码和(或)Unicode转义"之间的相互转换。 
| orbd.exe           | 对象请求代理守护进程(ObjectRequestBrokerDaemon)，它使客户端能够透明地定位和调用位于CORBA环境的服务器上的持久对象。 
| pack200.exe        | JAR文件打包压缩工具，它可以利用Java类特有的结构，对普通JAR文件进行高效压缩，以便于能够更快地进行网络传输。 
| packager.exe       | 这是微软提供的对象包装程序，用于对象安装包。 
| policytool.exe     | 策略工具，用于管理用户策略文件(.java.policy)。 
| rmic.exe           | JavaRMI编译器，为使用JRMP或IIOP协议的远程对象生成stub、skeleton、和tie类，也用于生成OMGIDL。 
| rmid.exe           | JavaRMI激活系统守护进程，rmid启动激活系统守护进程，允许在虚拟机中注册或激活对象。 
| rmiregistry.exe    | Java远程对象注册表，用于在当前主机的指定端口上创建并启动一个远程对象注册表。 
| schemagen.exe      | XMLschema生成器，用于生成XMLschema文件。 
| serialver.exe      | 序列版本命令，用于生成并返回serialVersionUID。 
| servertool.exe     | JavaIDL服务器工具，用于注册、取消注册、启动和终止持久化的服务器。 
| tnameserv.exe      | JavaIDL瞬时命名服务。 
| unpack200.exe      | JAR文件解压工具，将一个由pack200打包的文件解压提取为JAR文件。 
| wsgen.exe          | XMLWebService2.0的JavaAPI，生成用于JAX-WSWebService的JAX-WS便携式产物。 
| wsimport.exe       | XMLWebService2.0的JavaAPI，主要用于根据服务端发布的wsdl文件生成客户端存根及框架 
| xjc.exe            | 主要用于根据XMLschema文件生成对应的Java类。 


## JDK 官方版本特性参考

https://www.oracle.com/java/technologies/javase/8u191-relnotes.html

## linux java

tar -zxvf jdk-8u281-linux-x64.tar.gz
mkdir -p /usr/java
cp jdk1.8.0_281/ /usr/java/
cd /usr/java/
vim ~/.bashrc

JAVA_HOME=/usr/java/jdk1.8.0_281
CLASSPATH=$JAVA_HOME/lib/
PATH=$PATH:$JAVA_HOME/bin
export PATH JAVA_HOME CLASSPATH

cd ~
source .bashrc
java -version、




---
weight: 1
title: "Maven 总结"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Maven 总结"
resources:
- name: "base-image"
  src: "base-image.jpg"

tags: [Java, Note]
categories: [Java]

lightgallery: true

toc:
  auto: false
---

Maven的使用与学习总结

<!-- more -->

## base

POM 项目对象模型

构建生命周期：包含三个标准生命周期，clean default site，每个生命周期都包含若干个阶段，执行某个阶段的时候，该阶段之前的其它阶段都会被执行

仓库：在 Maven 中，任何一个依赖、插件或者项目构建的输出，都可以称之为构件。
仓库安装本地，中央，远程的顺序来查找，中央是官方仓库，远程是用户自定义远程仓库

插件：生命周期的执行都是由对应插件来完成的，可以配置相关插件的具体操作

项目模板，项目文档

快照：通过快照可以固化某个pom配置

自动化构建：构建流程控制，比如依赖的项目构建完成，才开始构建本项目

依赖管理

自动构建，通过对应的插件Release实现自动构建

什么是 maven的uber-jar

在maven的一些文档中我们会发现 "uber-jar"这个术语，许多人看到后感到困惑。其实在很多编程语言中会把super叫做uber （因为suber可能是关键字）， 这是上世纪80年代开始流行的，比如管superman叫uberman。所以uber-jar从字面上理解就是super-jar，这样的jar不但包含自己代码中的class ，也会包含一些第三方依赖的jar，也就是把自身的代码和其依赖的jar全打包在一个jar里面了，所以就很形象的称其为super-jar ，呵呵，uber-jar来历就是这样的

## 打包

通过IDEA就可以打包了，前提是要配置好pom文件，项目是maven工程

打包会扫描测试部分的代码，测试不通过也不行(可以配置跳过)。打包后得到jar包，用`java -jar api-1.0.jar`命名就可以运行，`api-1.0.jar.original`后缀的文件是给其它项目依赖用的(暂时不管，这个不能用来部署)

## 导入包到当前仓库

system 除了可以用于引入系统classpath 中包，也可以用于引入系统非maven  收录的第三方Jar，做法是将第三方Jar放置在 项目的 lib 目录下，然后配置 相对路径，但因system 不会打包进去所以需要配合 maven-dependency-plugin 插件配合使用。当然推荐大家还是通过将第三方Jar手动install 到仓库(这一点非常重要，因为各个企业不可能没有一点定制，除非jar包在私服仓库，否则最好就是要走这个安装流程，下面是一个安装示意)

`mvn install:install-file -DgroupId=com.ctg.itrdc.cache -DartifactId=ctg-cache-nclient -Dversion=2.4.0 -Dpackaging=jar -Dfile=/Users/liuzhi/Downloads/ctg-cache-nclient-2.4.0.jar`

根据实际情况，修改对应的参数

## maven快速构建项目

mvn使用插件快速构建项目

mvn archetype:generate 
    -DgroupId=com.liuzhidream.app 
    -DartifactId=spring-learn 
    -DarchetypeArtifactId=maven-archetype-quickstart 
    -DinteractiveMode=false

执行mvn操作，要进行pom文件所在目录

## Maven仓库

默认远程仓库： http://repo1.maven.org/maven2

本地仓库： 默认在 ~/.m2/respository 

```xml
<localRepository>D:/maven/repository</localRepository>
```

## 使用阿里仓库

```xml
<repositories>
    <repository>
        <id>central</id>
        <name>aliyun maven</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
        <layout>default</layout>
        <!-- 是否开启发布版构件下载 -->
        <releases>
            <enabled>true</enabled>
        </releases>
        <!-- 是否开启快照版构件下载 -->
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </repository>
</repositories>
```

这里还可以延伸很多知识点，首先maven是先从本地仓库找，找不到，再找远程仓库

远程仓库分为: 中央仓库，私服，其它远程仓库

- 默认是去中央仓库找
- 私服是一种特殊的远程仓库，架设于局域网内。当maven需要下载构件时，先从私服寻找，私服中没有再从外部仓库下载
- 任何一个仓库的id是唯一的，maven自带的中央仓库使用的id为central，如果其他仓库的id也为central，那么它将覆盖自带的中央仓库

实践总结: 有时候我们去maven官网找到的jar包并不是默认的中央仓库的(注意看，一般都会有个note告诉你)，这个时候就需要配置repositories，并且id不能用central。还有一种情况就是用阿里云的仓库代替maven的默认中央仓库

## 配置

`settings.xml`文件一般存在于两个位置

全局配置:  ${MAVEN_HOME}/conf/settings.xml
用户配置:  user.home/.m2/settings.xml

需要注意的是：**局部配置优先于全局配置**。
配置优先级从高到低：`pom.xml> user settings > global settings`
如果这些文件同时存在，在应用配置时，会合并它们的内容，如果有重复的配置，优先级高的配置会覆盖优先级的

/Applications/IntelliJ\ IDEA.app/Contents/plugins/maven/lib/maven3/conf  idea 路径，记得转义空格cp到目录下

我本地情况: 默认源码，idea配置文件都是空的配置，m2下没有配置文件（Mac环境）

## 依赖关系

项目依赖是指maven 通过依赖传播、依赖优先原则、可选依赖、排除依赖、依赖范围等特性来管理项目ClassPath

## dependency scope

依赖中有一个属性是描述依赖作用域的

1. compile
默认的scope，表示 dependency 都可以在生命周期中使用。而且，这些dependencies 会传递到依赖的项目中

2. provided
跟compile相似，但是表明了dependency 由JDK或者容器提供，如：servlet-api，因为servlet-api，tomcat等web服务器已经存在了，如果再打包会冲突。这个scope 只能作用在编译和测试时，同时没有传递性

使用这个时，不会将包打入本项目中，只是依赖过来，使用默认或其他时，会将依赖的项目打成jar包，放入本项目的Lib里

3. runtime
表示dependency不作用在编译时，但会作用在运行和测试时

4. test
表示dependency作用在测试时，不作用在运行时

5. system
跟provided 相似，但是在系统中要以外部JAR包的形式提供，maven不会在repository查找它

```
<scope>system</scope>
<systemPath>${java.home}/lib/rt.jar</systemPath>
```

6. import
只能用在dependencyManagement里面

我用实际例子说明这个作用，在`A项目`下`dependencyManagement`定义的`依赖B`使用`import`配置，可以把这个`依赖B`自己的`dependencies`下的依赖加到`A项目`下的`dependencyManagement`中

A项目

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

就相当于Spring-cloud的依赖被加入进来了，记住dependencyManagement的特性，加入进来要使用才在项目中被依赖

然后其他项目

```xml
 <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

这个项目的父pom是Spring-boot，却可以使用Spring-cloud的，不用声名版本号，就是因为import的特性

## 项目聚合与继承

聚合：是指将多个模块整合在一起，统一构建，避免一个一个的构建。聚合需要个父工程，然后使用 `<modules>` 进行配置其中对应的是子工程的相对路径

继承：是指子工程直接继承父工程 当中的属性、依赖、插件等配置，避免重复配置

1. 属性继承
2. 依赖继承
3. 插件继承

上面的三个配置子工程都可以进行重写，重写之后以子工程的为准

依赖管理：通过继承的特性，子工程是可以间接依赖父工程的依赖，但多个子工程依赖有时并不一至，这时就可以在父工程中加入 `<dependencyManagement>` 声明工程需要的JAR包，然后在子工程中引入

```xml
<！-- 父工程中声明 junit 4.12 -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
    </dependencies>
</dependencyManagement>
<!-- 子工程中引入 -->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
</dependency>
```

1. dependencyManagement 只是声明依赖的版本号，该依赖不会引入，因此子项目需要显示声明所需要引入的依赖，若不声明则不引入
2. 子项目声明了依赖且未声明版本号和scope，则会继承父项目的版本号和scope，否则覆盖

上述的2点描述了dependencyManagement的作用，本来父pom文件定义的依赖，子pom继承，但是两个子pom可能版本号不一样，这就导致了同一个jar包有两个版本，而dependencyManagement就是解决这个问题的

## maven optional

俗称依赖传递，当pom A 依赖了 依赖 B之后，如果pom C 继承 pom A，那么依赖 B 也会传到 pom C中，如果我们不想要这个依赖，一个办法就是在
pom C 中排除依赖 `exclusions`，还有一种办法就是用 optional，在依赖 B中设置 optional，这个依赖就只在当前pom生效，不会传递到 pom C

如果使用了 optional 后，想在 pom C 中去使用这个依赖，按照常规方式声名就行了

## 项目属性

```xml
<!-- 配置proName属性 -->
<properties>
  <proName>fox</proName>
</properties>

<!-- 引用方式 -->
${proName}
```

默认属性

```
${basedir} 项目根目录;    
${version} 表示项目版本;  
${project.basedir}同${basedir};  
${project.version}表示项目版本,与${version}相同;  
${project.build.directory} 构建目录，缺省为target  
${project.build.sourceEncoding}表示主源码的编码格式;  
${project.build.sourceDirectory}表示主源码路径;  
${project.build.finalName}表示输出文件名称;  
${project.build.outputDirectory} 构建过程输出目录，缺省为target/classes 
```

## 项目构建配置

就是如何构建，包括资源文件的各种包含，过滤等配置，不过如果是spring-boot项目，直接依赖一个插件就完事了，如果是其它项目需要自定义配置就是在这里配置

```xml
<build>
</build>
```

pluginManagement 的作用，就是声名，和依赖类似，一般是父pom来定义，子pom就可以直接用这个定义了

```xml
<pluginManagement>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-source-plugin</artifactId>
            <version>2.1</version>
            <configuration>
                <attach>true</attach>
            </configuration>
            <executions>
                <execution>
                    <phase>compile</phase>
                    <goals>
                        <goal>jar</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</pluginManagement>

<!-- 子pom中 -->
<plugins>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-source-plugin</artifactId>
    </plugin>
</plugins>
```

## 关于测试

maven测试为 default 生命周期中的test阶段
test阶段与 maven-surefire-plugin 的test目标相绑定了， 这是一个内置的绑定
Maven通过插件来执行 JUnit 和 TestNG 的测试用例

maven-surefire-plugin 的test目标会自动执行测试源码路径下符合命名模式的测试类
默认测试源代码路径：src/test/java/
测试类命名模式：

```s
**/Test*.java
**/*Test.java
**/*TestCase.java
```

按上述模式命名的类， 使用 mvn test 命令就能自动运行他们

## 跳过测试

mvn install -DskipTests，不过会一直跳过测试

配置式：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.19.1</version>
    <configuration>
        <skipTests>true</skipTests>
        <!-- 
            skip 对应命令行参数为 maven.test.skip  
            -->
        <skip>false</skip>
    </configuration>
</plugin>
```

如果想不仅跳过测试运行，还跳过测试代码的编译，使用下面命令：

`mvn package -Dmaven.test.skip=true`
maven.test.skip 控制了 maven-compiler-plugin 和 maven-surefire-plugin 两个插件的行为

或借助idea，比较方便

## 指定测试

maven-surefire-plugin 使用 test 参数指定测试用例, 为测试用例的类名
`mvn test -Dtest=RandomTest`
只执行 RandomTest 这个测试类.
`mvn test -Dtest=RandomTest#myTest`
上面命令，只运行 RandomTest 类的 myTest 方法

可以指定多个类，逗号分隔
`mvn test -Dtest=RandomTest,Random2Test`
也可以用 * 匹配多个
`mvn test -Dtest=Random*Test`
*和 逗号可以结合使用。

如果不指定或者找不到测试类则构建失败
`mvn test -Dtest`
failIfNoTests 参数控制没有测试用例不报错
`mvn test -Dtest -DfailIfNoTests=false`

## 包含测试用例

将不符合命名模式测试类自动运行测试

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.19.1</version>
    <configuration>
        <includes>
            <include>**/*Tests.java</include>
        </includes>
    </configuration>
</plugin>
```

两个星号 ** 表示匹配任意路径。
上面表示匹配已 Tests.java 结尾的Java类

## 排除测试用例

排除测试用例不实用test自动运行
使用 excludes 节点

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.19.1</version>
    <configuration>
        <excludes>
            <exclude>**/*ServiceTest.java</exclude>
        </excludes>
    </configuration>
</plugin>
```

## 测试代码重用

mvn package 会打包项目主代码和资源文件代码，没有包含测试代码。
如果想一起打包测试用例，供依赖方使用， 使用 maven-jar-plugin 插件

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>2.4</version>
    <executions>
        <execution>
            <goals>
                <goal>test-jar</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

maven-jar-plugin 有两个目标 jar ，test-jar,
jar 内置绑定在 default 生命周期的 package 阶段。
test-jar没有内置绑定。

依赖方引入时 dependency

```xml
<dependency>
    <groupId>org.A</groupId>
    <artifactId>A</artifactId>
    <version>5.0.0</version>
    <type>test-jar</type>
    <scope>test</scope>
</dependency>
```

需要设置 type 和 scope

参考原文：https://blog.csdn.net/yonggang7/article/details/79780487
---
weight: 1
title: "jar包结构讲解"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "jar包结构讲解"
resources:
- name: "base-image"
  src: "base-image.jpg"

tags: [Java, Note]
categories: [Java]

lightgallery: true

toc:
  auto: false
---

jar包结构讲解，搞懂哪些META-INF/MANIFEST.MF是什么意思

## 官方参考

https://docs.oracle.com/javase/7/docs/technotes/guides/jar/jar.html {% github_emoji tada %} {% github_emoji tada %}

本文参考: https://www.cnblogs.com/applerosa/p/9736729.html

## 什么是jar

JAR(Java Archive File)，Java 档案文件。通常jar 为压缩文件, 与 ZIP/RAR 压缩文件 一样的概念
区别在于 jar 文件中存在一个名为`META-INF/MANIFEST.MF`的清单文件，关于JAR包的描述信息、启动时的配置信息和安全性信息等均保存在其中，可以理解为 jar 的一个`配置文件`

## 可执行的 JAR

- 一个可执行的 jar 文件是一个自包含的 Java 应用程序，它存储在特别配置的JAR 文件中，可以由 JVM 直接执行它而无需事先提取文件或者设置类路径
- 要运行存储在非可执行的 JAR 中的应用程序，必须将它加入到你的类路径中，并用名字调用应用程序的主类。就是我们常说的”第三方类库的概念”
- 但是使用可执行的 JAR 文件，我们可以不用提取它或者知道主要入口点就可以运行一个应用程序。可执行 JAR 有助于方便发布和执行 Java 应用程序

1. 创建可执行 JAR

假设应用程序中的主类是: cn.china.demo.Application.java, 里面有个可执行的main() 函数入口,要创建一个包含应用程序代码的 JAR 文件并标识出主类。为此，在某个位置(不是在应用程序目录中)创建一个名为 manifest 的文件，并在其中加入以下一行:

`Main-Class: cn.china.demo.Application`

然后，像这样创建 JAR 文件：

`jar cmf manifest example.jar application-dir`

所要做的就是这些了 -- 现在可以用 java -jar 执行这个 JAR 文件 example.jar

一个可执行的 JAR 必须通过 menifest 文件的头引用它所需要的所有其他从属 JAR

如果使用了 -jar 选项，那么环境变量 CLASSPATH 和在命令行中指定的所有类路径都被 JVM 所忽略

2. 启动可执行 JAR

既然我们已经将自己的应用程序打包到了一个名为 example.jar 的可执行 JAR 中了，那么我们就可以用下面的命令直接从文件启动这个应用程序:
`java -jar example.jar`

3. 扩展打包

扩展为 Java 平台增加了功能，在 JAR 文件格式中已经加入了扩展机制。扩展机制使得 JAR 文件可以通过manifest 文件中的 Class-Path 头指定所需要的其他 JAR 文件

假设 extension1.jar 和 extension2.jar 是同一个目录中的两个 JAR 文件，extension1.jar 的 manifest 文件包含以下头:

`Class-Path: extension2.jar`

这个头表明 extension2.jar 中的类是 extension1.jar 中的类的 扩展类。extension1.jar 中的类可以调用extension2.jar 中的类，并且不要求 extension2.jar 处在类路径中

在装载使用扩展机制的 JAR 时，JVM 会高效而自动地将在 Class-Path 头中引用的 JAR 添加到类路径中。不过，扩展 JAR 路径被解释为相对路径，所以一般来说，扩展 JAR 必须存储在引用它的 JAR 所在的同一目录中

例如，假设类 ExtensionClient 引用了类 ExtensionDemo ,它捆绑在一个名为 ExtensionClient.jar 的 JAR 文件中，而类 ExtensionDemo 则捆绑在 ExtensionDemo.jar 中

为了使 ExtensionDemo.jar 可以成为扩展，必须将ExtensionDemo.jar 列在 　　ExtensionClient.jar 的 manifest 的 Class-Path 头中，如下所示:

```sh
Manifest-Version: 1.0
Class-Path: ExtensionDemo.jar
```

在这个 manifest 中 Class-Path 头的值是没有指定路径的 ExtensionDemo.jar，表明 ExtensionDemo.jar 与ExtensionClient JAR 文件处在同一目录中

## 总结

jar包含一个META-INF/MANIFEST.MF，描述jar的配置信息，一个jar可用依赖另一个jar，配置要正确，处在同一目录

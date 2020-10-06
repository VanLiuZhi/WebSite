---
weight: 1000
title: "Java IDEA"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Java IDEA"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Java, Note]
categories: [Java-MicroServices]

lightgallery: true

toc:
  auto: false
---

IDEA 使用技巧补充总结

<!-- more -->

## IDEA 工具使用

ntellig idea 使用@Resource或者@Autowire报错，出现红色波浪线；

虽然不影响使用，但是看着很不爽，所以还是解决了下：

报错提示：Could not autowire. No beans of '' type found. less... (Ctrl+F1)  Checks autowiring problems in a bean class.

解决方法：Settings - Editor - Inspections - Spring - Spring Core - Code - Autowiring for Bean Class 修改成告警级别

## idea 错误提示取消和恢复

idea提供了很强大的错误提示功能，但是有些不一定是错误的，强迫症患者可能会取消，下图展示 spring yaml配置错误提示 取消和开启的设置的图片

![image](/images/Java/java-idea-inspections.png)

## idea 注释调整

我们在注释代码的时候，发现注释顶格了，这样的风格特别不好看，可以去editor code style中调整，不用语言用不同配置，找到code generatic，修改comment code配置

## 热部署

静态语言不像动态语言，要实现热部署要程序和IDEA配合才行

引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
</dependency>
```

依赖的作用：

默认禁用缓存选项。比如模板引擎将缓存编译完的模板，以避免重复解析模板文件。
自动重启。只要classpath下的文件有变动，应用就会自动重启。
在运行一个完整的、打包过的应用时，开发者工具（devtools）会被自动禁用。
如果应用使用 java -jar 或特殊的类加载器启动，都会被认为是一个产品级的应用（production application），从而禁用开发者工具。
只要classpath下文件有变动，应用就会重启。一些比如静态assets、视图模板文件等资源 文件的变动，应用不会重启。
`唯一触发重启的方式就是更新classpath`
IDEA本身提供了热部署功能，但是限制性比较大，只能对静态资源的修改、方法内的修改才能进行热更新，
对于方法参数或者方法类的修改不能进行热部署，但是像devtools,jrebel 都能够对类的修改进行重新加载。

总结：

添加spring-boot-devtools依赖
修改classpath下的Java文件，然后更新classpath，这时应用就会自动重启。
修改classpath下的页面文件，然后更新classpath，但是访问页面可以看到效果（即重新加载）。
IDEA更新classpath的方法：【Build】->【Build Project】，如果你设置了自动编译，那就可以省略这一步了（可能因为IDEA版本的原因，有需要手动操作）。

1. spring-boot-devtools 热部署是对修改的类和配置文件进行重新加载，所以在重新加载的过程中会看到项目启动的过程，其本质上这个时候只是对修改类和配置文件的重新加载，所以速度极快；
2. spring-boot-devtools 对于前端使用模板引擎的项目，能够自动禁用缓存，在页面修改后，只需要刷新浏览器器页面即可；
3. 为什么在 idea 中 spring-boot-devtools 没有热部署？ 因为在Idea 中自动编译默认是停用的，启用路径 build -> compile -> buildProjectAutomatically
4. 为什么在 idea 中启用自动编译依然没有热部署？ idea监测到项目runninng 或者 debuging 会停用自动编译，所以还需要手动biild [Ctrl + F9] 或者 [ctrl +  b]

全自动设置方式，打开运行时编译：

1. build -> compile -> build Project Automatically 开启自动编制，但是running和debug不能用
2. 快捷键Ctrl + Shift + Alt + /，选择Registry  让running能自动编译
3. 勾选 Compiler autoMake allow when app running

上述情况的热部署，有些时候需要手动点击编译(我在Windows上会自动重启)，推测和idea版本有关系，不过手动控制其实挺好的
另外通过上述做的热部署，是会重启(可能依赖比较复杂的情况下会这样，有些时候又不会)，推荐使用插件jrebel，它也是走上面的流程，然后可以只加载需要重新加载的类，效果很好

spring-boot-devtools依赖 注入的时候，通过idea可以看到服务后面有devtool的标识

jrebel xrebel 有2个东西，xrebel是分析程序用的，jrebel是热部署

百度一下安装和激活：两个要分别激活，激活后大都改用离线模式，可以用好几个月

只为了热部署：百度一下 jrebel 激活，需要一个服务器地址，和一个uuid，这两个组合，加上自己的邮箱就行了


## gradle 编译，只运行所选测试

Delegate IDE build开启，选择 reun tests using下拉选项

## IDEA 插件

Lombok plugin
GsonFormat
FindBugs-IDEA
CodeGlance	右侧文档结构图
.ignore	git 文件提交过滤
Maven Helper maven插件，打开该pom文件的Dependency Analyzer视图

## IDEA 快捷键

    Ctrl+Alt+O 优化导入的类和包 
    Alt+Insert 生成代码(如get,set方法,构造函数等) 或者右键（Generate） 
    fori/sout/psvm + Tab 
    Ctrl+Alt+T 生成try catch 或者 Alt+enter 
    CTRL+ALT+T 把选中的代码放在 TRY{} IF{} ELSE{} 里 
    Ctrl + O 重写方法 
    Ctrl + I 实现方法 
    Ctr+shift+U 大小写转化 
    ALT+回车 导入包,自动修正 
    ALT+/ 代码提示 
    CTRL+J 自动代码 
    Ctrl+Shift+J，整合两行为一行 
    CTRL+空格 代码提示 
    CTRL+SHIFT+SPACE 自动补全代码 
    CTRL+ALT+L 格式化代码 
    CTRL+ALT+I 自动缩进 
    CTRL+ALT+O 优化导入的类和包 
    ALT+INSERT 生成代码(如GET,SET方法,构造函数等) 
    CTRL+E 最近更改的代码 
    CTRL+ALT+SPACE 类名或接口名提示 
    CTRL+P 方法参数提示 
    CTRL+Q，可以看到当前方法的声明
    Shift+F6 重构-重命名 (包、类、方法、变量、甚至注释等) 
    Ctrl+Alt+V 提取变量



    Ctrl＋Shift＋Backspace可以跳转到上次编辑的地 
    CTRL+ALT+ left/right 前后导航编辑过的地方 
    ALT+7 靠左窗口显示当前文件的结构 
    Ctrl+F12 浮动显示当前文件的结构 
    ALT+F7 找到你的函数或者变量或者类的所有引用到的地方 
    CTRL+ALT+F7 找到你的函数或者变量或者类的所有引用到的地方
    Ctrl+Shift+Alt+N 查找类中的方法或变量 
    双击SHIFT 在项目的所有目录查找文件 
    Ctrl+N 查找类 
    Ctrl+Shift+N 查找文件 
    CTRL+G 定位行 
    CTRL+F 在当前窗口查找文本 
    CTRL+SHIFT+F 在指定窗口查找文本 
    CTRL+R 在 当前窗口替换文本 
    CTRL+SHIFT+R 在指定窗口替换文本 
    ALT+SHIFT+C 查找修改的文件 
    CTRL+E 最近打开的文件 
    F3 向下查找关键字出现位置 
    SHIFT+F3 向上一个关键字出现位置 
    选中文本，按Alt+F3 ，高亮相同文本，F3逐个往下查找相同文本 
    F4 查找变量来源
    CTRL+SHIFT+O 弹出显示查找内容
    Ctrl+W 选中代码，连续按会有其他效果 
    F2 或Shift+F2 高亮错误或警告快速定位 
    Ctrl+Up/Down 光标跳转到第一行或最后一行下
    Ctrl+B 快速打开光标处的类或方法 
    CTRL+ALT+B 找所有的子类 
    CTRL+SHIFT+B 找变量的类
    Ctrl+Shift+上下键 上下移动代码 
    Ctrl+Alt+ left/right 返回至上次浏览的位置 
    Ctrl+X 删除行 
    Ctrl+D 复制行 
    Ctrl+/ 或 Ctrl+Shift+/ 注释（// 或者/…/ ）
    Ctrl+H 显示类结构图 
    Ctrl+Q 显示注释文档
    Alt+F1 查找代码所在位置 
    Alt+1 快速打开或隐藏工程面板
    Alt+ left/right 切换代码视图 
    ALT+ ↑/↓ 在方法间快速移动定位 
    CTRL+ALT+ left/right 前后导航编辑过的地方 
    Ctrl＋Shift＋Backspace可以跳转到上次编辑的地 
    Alt+6 查找TODO



    SHIFT+ENTER 另起一行 
    CTRL+Z 倒退(撤销) 
    CTRL+SHIFT+Z 向前(取消撤销) 
    CTRL+ALT+F12 资源管理器打开文件夹 
    ALT+F1 查找文件所在目录位置 
    SHIFT+ALT+INSERT 竖编辑模式 
    CTRL+F4 关闭当前窗口 
    Ctrl+Alt+V，可以引入变量。例如：new String(); 自动导入变量定义 
    Ctrl+~，快速切换方案（界面外观、代码风格、快捷键映射等菜单



    alt+F8 debug时选中查看值 
    Alt+Shift+F9，选择 Debug 
    Alt+Shift+F10，选择 Run 
    Ctrl+Shift+F9，编译 
    Ctrl+Shift+F8，查看断点
    F7，步入 
    Shift+F7，智能步入 
    Alt+Shift+F7，强制步入 
    F8，步过 
    Shift+F8，步出 
    Alt+Shift+F8，强制步过
    Alt+F9，运行至光标处 
    Ctrl+Alt+F9，强制运行至光标处 
    F9，恢复程序 
    Alt+F10，定位到断点



    Ctrl+Alt+Shift+T，弹出重构菜单 
    Shift+F6，重命名 
    F6，移动 
    F5，复制 
    Alt+Delete，安全删除 
    Ctrl+Alt+N，内联

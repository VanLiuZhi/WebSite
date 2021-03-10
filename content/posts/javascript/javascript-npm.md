---
weight: 8800
title: "npm JavaScript 包管理工具"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "npm JavaScript 包管理工具"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Node.js, Note]
categories: [FrontEnd(前端)]

lightgallery: true

toc:
  auto: false
---

npm是node的包管理工具，不建议使用任何第三方的工具，也不建议换源，这些操作解决一时问题也带来其它问题。关于网络问题，是在不行用手机热点，如果你们公司的网络都下不了，那公司不用待了。第三方工具也是，开始npm确实比不上第三方工具，不过现在渐渐好多了，官方也意识到这些问题了。

<!-- more -->

# npm

npm是node的包管理工具，不建议使用任何第三方的工具，也不建议换源，这些操作解决一时问题也带来其它问题。关于网络问题，实在不行用手机热点，如果你们公司的网络都下不了，那公司不用待了。第三方工具也是，开始npm确实比不上第三方工具，不过现在渐渐好多了，官方也意识到这些问题了。

## 全局和局部

一般在全局安装的是工具，比如webpack，这样这些工具在构建项目或者执行项目的命令的时候由于是全局任何地方都能使用，而局部就是装模块的，这些模块可能因为依赖关系，你最好不要在全局装模块，如果你的项目引用全局模块，多个项目的时候，可能依赖不一样，这样你去更新全局模块的时候就可能由于依赖的问题影响其它项目了。

## 命令

| Command         	            |  Description                     
| ----------------------------- |:-------------------------------: 
| npm list -g --depth 0		      | 查看全局安装包 
| npm install packagename -g    | 全局安装
| npm uninstall package -g      | 全局卸载
| npm install pg --save         | 项目依赖安装
| npm install pg --save-dev     | 项目非依赖安装
| npm view jquery versions      | 查看模块版本号，这里举例的是jQuery

要更新某个包到最新版本，直接安装即可 npm install element-ui 直接把当前包安装到最新版本

## node-sass 问题

这东西就非常的离谱，对Windows非常不友好，可以替换成 sass

执行命令

npm uninstall node-sass
npm install sass -S -D

不过会有不兼容的语法，全局搜索 /deep/ 如果有需要改一下这种写法


```css
.a {
  /deep/ {
    .b {
      color: red;
    }
  }
}

/* 修改为 */
.a {
  ::v-deep {
    .b {
      color: red;
    }
  }
}
```

参考

https://github.com/PanJiaChen/vue-element-admin/releases/tag/4.3.0

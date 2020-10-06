---
weight: 5502
title: "python web 开发(二) 前端工程"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "python web 开发(二) 前端工程"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Python, Web, Technology]
categories: [Python]

lightgallery: true

toc:
  auto: false
---

前端最近几年发展的势头应该是最大的了，开始我在项目中是利用框架的模板来编写前端代码的，后来随着业务的复杂度不断的上升，用传统的jQuery已经很难实现需求了，这时候转向了Vue这种数据驱动的框架，这使得很多需求几行代码就搞定了，开发效率大幅度提升。

<!-- more -->

## 概述

这应该是前端最好的时代了，以前用jQuery去实现，少的都是一千行起步，而且由于js的压力是在客户端，很多人基本都不怎么考虑性能。

我认为JavaScript处在一种需要重构的阶段，它的初衷不是为了编写大型应用的，所以现在各种加特性，各种扩展，包括css等，后缀不知道多少种。个人建议最好选择其中一种生态来学习，如果不是写底层，有些特性不需要了解太多。

推荐 Vue 或 React，本项目使用的是Vue，是全生态的Vue，就是前端都是基于Vue的，后端只做接口。

Vue在前端中有两个用法，一种就是作为扩展引入它，解决那些复杂的需求。另外就是整个前端都是用Vue来写的，就是常说的单页面Single page，它只会加载一个页面，其它的操作都是通过更新DOM来完成的，这节约了很多请求的开销，同时体验也很好，像企业系统等项目就很适合用Vue来做。

另外移动开发也适合Single page这样的应用，可以做混合式APP，省去了开发Android和iOS的麻烦。

当然弊端是不可少的，就是兼容性，作为新的技术，它使用了es4无法模拟的特性，使得像IE浏览器这种老内核支持很差，不过有对应的方案，可以去官方文档找。

关于Vue的学习参考官方文档即可，总的来说上手还是很快的。

## 构建

通过Vue的脚手架vue-cli来构建工程，需要配合到很多工具，需要用到Node.js作为环境。由于官方一直在更新，使用最新的脚手架构建的工程应该跟我有一些出入，影响不大就行。脚手架就是快速创建工程的工具，前端全生态Vue，需要用到很到工具，如果你每个都自己搭，学习成本还是很高，其实我的前端主要精力放在开发上，我也没像Python一样很系统的学习过它的语法，主要真的是用的少，像为了实现面向对象的原型链，几乎在开发中不会去用，这也因为不做底层，套用框架的原因。

vue的生态主要就是用到了vue, vue-cli，vue-loader，webpack，理解这几个模块的用法，请求使用axios模块，不要再用ajax了。

个人建议使用nvm创建虚拟环境，因为更新很频繁，有些版本会和其它有冲突，使用虚拟环境，这样安装在全局的模块不想要了直接删除虚拟环境就行了，不会对系统造成很大的文件残留。

## 与后端配合

虽然Node.js提供了全套的开发方案，不过本项目是以Python为主要后端，VUE应该怎么样和后端配合呢？就是把编译后的静态文件由Python来处理就行了，一般会有一个index.html文件作为入口，Python需要在首页的url返回这个index.html，然后组织好编译后的静态文件，确保能被搜索到，这属于部署阶段。

开发阶段就是以Node.js为前端环境，前端需要跑一个独立的服务，需要和后端的服务进行交互。

## Vue 基本

使用Vue，主要是数据驱动式，所以要把思维转换过来，传统的jQuery的方式是操作DOM，而Vue底层也是操作DOM，但是用了虚拟DOM的概念，会更节省资源，而且除非必须，一般不会显示的去操作DOM，因为没这个必要，它的数据驱动式通过双向绑定很容易就修改DOM。

讲解几个比较常用的点：

1. 在全生态Vue的架构中，vue实例只被创建一次，之后都是围绕组件来展开的，这点很重要。

2. 理解生命周期，这是常见的话题，尤其是在早前版本的时候，更新DOM的时间点，引发很多问题，导致不能及时修改，其实有解决方案，只是可能你用的版本还不支持。

3. 基础部分都是围绕着数据双向绑定来的，这部分运用的差不多了，就到了组件了，组件就是把常用的功能封装出来，比如一个页面的尾部，可以独立成一个组件，在各个页面中去使用。

### 在组件中触发父组件的方法

有时候需要在组件中调用父组件的方法，应该这样写 `@click="$emit('click_event', item.guid)"`

在父组件中需要由该事件的处理函数 `<post-tag v-on:click_event="tagHandler"></post-tag>`

如此，由子组件发起的事件被父组件接收，由tagHandler方法来处理。

### 插槽

还有一个特性就是插槽，对于这个特性，可以用在编写组件的地方，但是最常用的还是各种框架会使用它，所以如果你不了解它，就不能很好的使用框架提供的组件。

在 element-UI 的 el-table 组件中，把remark绑定到v-model中，就是使用了作用域插槽

```html
<template slot-scope="scope">
    <textarea
            style="height: 50px;width: 80px; border: 1px solid #dedede;"
            type="textarea" placeholder="备注"
            v-model="scope.row.remark">
    </textarea>
</template>
```

### 模块化

这部分主要关注的是路由和状态管理，至于服务端渲染，还没有用过。

路由是vue生态中一定会用的，它把前端的页面请求拦截了，交由特定的组件来处理，路由可以带参数，在组件页面可以通过 `this.$route.params.data` 获取路由对象相关的数据。

```js
Vue.use(Router)

export default new Router({
  routes: [
    {
      path: '/',
      name: 'index',
      component: Index
    },
    {
      path: '/article_details/:guid',
      name: 'ArticleDetails',
      component: ArticleDetails
    },
    {
      path: '/auther',
      name: 'auther',
      component: Auther
    },
    {
      path: '/test',
      name: 'test',
      component: Test
    }
  ]
})
```

状态管理是比较大型的应会用到的东西，需要使用vuex来使用，我在一些别人的大型的项目中看到过，本项目没有用到不做过多赘述。

## 项目分析

由于这不是一个具体的实际需求的项目，前端部分主要讲个人认为需要注意的坑，和一些需要修改的地方。

### 依赖

各种依赖在package.json中，其中script模块用来描述脚本命令：

```js
"scripts": {
    "dev": "webpack-dev-server --inline --progress --config build/webpack.dev.conf.js",
    "start": "npm run dev",
    "unit": "jest --config test/unit/jest.conf.js --coverage",
    "e2e": "node test/e2e/runner.js",
    "test": "npm run unit && npm run e2e",
    "lint": "eslint --ext .js,.vue src test/unit test/e2e/specs",
    "build": "node build/build.js"
```

可以修改默认命令，或者添加命令。

### eslint

如果开启了eslint，会在外层有对应的配置文件，和Python flak8类似的工具

### build 目录

主要放置和webpack工具相关的东西，这部分就是对新手来说不友好的地方了，因为我看过很多人的代码结构都不一样，你为了完全理解需要仔细学习webpack。

分开了开发和生产的配置，以及编译相关的东西。

### config 目录

前面的 build 目录会读取 config 目录的内容，所以把配置修改放在这里。

### 跨域处理

在开发阶段，是没有编译代码的，所以后端和前端的服务都需要启动，这就涉及到一个老生常谈的问题，跨域，解决方案是使用代理，在index.js中去配置它。

```js
proxyTable: {
      '/api': { // 使用/api 来代替 target的内容，在ajax url 里面 /api/url
        target: 'http://0.0.0.0:5000/',
        changeOrigin: true,
        pathRewrite: {
          '^/api': 'http://0.0.0.0:8083'
        }
      }
    },
```

它把特定的请求代理了，由前端服务来向后端请求，这样就不会被浏览器认为使用了跨域。

### 配置编译文件路径

这也是非常重要的一个环节，默认的编译路径也许和后端配合不上，还要去改后端的代码，其实可以配置编译路径，比如使用 `/static/ele_ui_web/` 这样webpack组织文件的时候就是以这个路径为基准，然后把编译的代码放到flask的静态文件目录就可以了，flask静态文件搜索目录默认就是 `static`。

```js
build: {
    // Template for index.html
    index: path.resolve(__dirname, '../static/index.html'),

    // Paths
    // assetsRoot: path.resolve(__dirname, '../dist'),
    assetsRoot: path.resolve(__dirname, '../static/ele_ui_web'),
    assetsSubDirectory: 'static',
    assetsPublicPath: '/static/ele_ui_web/', // If you are deployed on the root path, please use '/'
    // assetsPublicPath: '/',
}
```

## src 源文件

src就是放源码的文件夹，其中有一个assets文件需要注意，放在该目录下的文件会直接在编译后，扔到对应的路径中，也就是说这个目录你应该用来存放图片或不参与编译的资源。

### api

存储api相关的内容，就是和后端交互的接口部分，接口请求推荐使用axios

主模块

```js
import axios from 'axios'

// 创建代理请求axios的实例, 更进一步的需求请配置参数
const service = axios.create({
  timeout: 3000
})

export default service
```

接口部分，对开发和生产做统一处理，每次请求就是调用一次service实例。

```js
import base from './base'

// 当前接口统一请求路径处理
let baseApiUrl = ''
if (process.env.NODE_ENV === 'development') {
  baseApiUrl = '/api/api/'
} else {
  baseApiUrl = '/api/'
}

const getUrl = function(url) {
  return baseApiUrl + url
}

export function getPostList(params) {
  return base({
    url: getUrl('post/getPostList'),
    method: 'post',
    data: params
  })
}
```

{% blockquote %}
由于走axios的请求和传统的ajax不同，这个更加原生，后端取数据需要注意
{% endblockquote %}

### components 和 view

components，存放组件的文件，view可以理解为实际组织的页面

### utils

utils 是工具文件，比如日期转换函数。

### 最外层

分别有App.vue文件，一些全局的设置可以在这里，main.js 全局初始化，index.html html文件基本结构，可以用来修改head头等，vue挂载的DOM也在这里。

### element-UI

本次项目使用的前端UI组件，按照官方的要求来安装即可，各个组件都有例子，可以从例子来修改，快速上手。




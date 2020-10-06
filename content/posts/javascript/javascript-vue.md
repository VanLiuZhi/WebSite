---
weight: 8800
title: "Vue 渐进式前端框架"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Vue 渐进式前端框架"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [JavaScript, Framework, Note, Vue]
categories: [FrontEnd(前端)]

lightgallery: true

toc:
  auto: false
---

Vue是MVVM框架，一个构建数据驱动的 web 界面的渐进式框架。Vue.js 的目标是通过尽可能简单的 API 实现响应的数据绑定和组合的视图组件。它不仅易于上手，还便于与第三方库或既有项目整合。

<!-- more -->

## Vue

[一个不错的“简书”入门](https://www.jianshu.com/p/dc5057e7ad0d)

![image](/images/JavaScript/vue-1.png)

## 基本

- 在通过ajax获取到数据需要赋值到data里面的时候，如果是不可变变量，可以直接赋值，但是如果是arrey，需要迭代每个值，加到data中。`Ajax.data.forEach(function(val, index){  vue.data.push(val) })`。

- vue:在html中传递 this  ，在vue中this都是指向vue的组件，如果我们想使用原本的this指向这个dom,需要这样使用`dofunc($event)`。在函数里面 `dofunc(v){ v.target }`。如果转换为 jQuery 对象 `$(v.target)`  

- vue:由于dom是由js去渲染的，所以你在渲染的时候去操作dom，是选不到的。这里涉及到了vue的生命周期的问题，实例创建完毕(挂载)，再去渲染dom。

- vue:template不会渲染成元素，用div的话会被渲染成元素。把if,show,for等语句抽取出来放在template上面，把绑定的事件放在temlpate里面的元，可以使html结构更加清晰，还可以改善一个标签过长的情况。

- 注册指令：全局注册，在new vue同块写Vue.directive局部注册，当前组件使用，作为vue实例的一个属性  directives  多了个 S
注册组件也是如此，和指令类似

- 在vue实例中的选择基本都是可以在组件里面使用的（vue实例怎么写组件就怎么写），但是data必须是函数，如果是一般的对象，你在组件里面使用这个对象会报错

- 单页面的VUE实例只有一个，组件化开了，要想从实例拿到data，只能是组件props向下传递，记得绑定想要的数据在你的模板上。向上使用events

- props  data  是驼峰命名，绑定数据的写法 `<child :msg-a="msgA"></child>  v-bind=" a "` 使用绑定，外部的引号不是想表达这个是个字符串，它应该当成一个变量，这也是在绑定url的时候，我们可以使用变量加上字符串，其中的字符串就用单引号。

- props: 单项流数据，从父组件流向子组件，子组件试图修改它会报错，如果你要用它，应该把这个值给data，定义局部变量的方法。如果data是可变类型的，在子组件中修改了是会影响到父组件的。

- props:验证，可以验证流进来的数据。验证在这个组件实例创建之前，所以你不能把这个组件里面的 option 诸如 data methods用在验证里面。

- 插槽：组件嵌套的时候使用，定义了如何进行内容分发

- 组件实例的作用域是孤立的

- vue:自定义组件命名不要命名常见的（怕和框架冲突）

- 给vue组件绑定事件时候，必须加上native ，不然不会生效（监听根元素的原生事件，使用 .native 修饰符）。等同于在自组件中：子组件内部处理click事件然后向外发送click事件：`$emit("click".fn)`

## 字符串模板和非字符串模板

```html

<script id="component1" type="x-template">
</script>

```

在实例中option使用 template 会把挂载元素的内容替换掉，在组件中 option 使用 template是HTML元素扩展被替换的内容，很像，都是替换。  

实例的模板字符串，执行元素的时候，此时元素应该是template标签或者 `script type=x-template` 都是把这两个的内容替换到实例挂载的元素上。

## 指令

vue指令类似 v-model 可以自定义指令，在创建实例的时候声明即可

目前的vue架构，对于一个vue文件来说，在里面使用其它组件（就是引用的各种组件），那么这些组件对于当前vue文件来说就是子组件，当前vue文件是父组件。这在理解一些概念的时候会有用，比如子组件 `双向绑定值` 使用 `sync` 来修饰，把子组件的某个属性绑定到父组件上，做到双向绑定。

| Command   | Description 
| --------- | :-------: 
| v-bind    | 动态绑定数据。简写为“:” => 以后的:class="{red:boolean}" 
| v-on      | 绑定时间监听器。简写为“@”，例：@click="xxx"
| v-text    | 更新数据，会覆盖已有结构。类似 `{ {msg} }`
| v-show    | 根据值的真假，切换元素的display属性
| v-if      | 根据值的真假，切换元素会被销毁、重建；=> 在dom中已消失 
| v-else-if | 多条件判断，为真则渲染
| v-else    | 条件都不符合时渲染
| v-for     | 基于源数据多次渲染元素或模块
| v-model   | 在表单控件元素（input等）上创建双向数据绑定（数据源）
| v-pre     | 跳过元素和子元素的编译过程
| v-once    | 只渲染一次，随后数据更新也不重新渲染
| v-cloak   | 隐藏未编译的Mustache语法，在css中设置[v-cloak]{display:none;} 

## 交互命令

| Command                     | Description 
| --------------------------- | :-------------------------: 
| vue cli                     | 主要功能就是创建vue工程 
| vue init webpack myproject  | 构建vue项目 

## ref 和 $refs

ref 这个通常在元素上使用（组件自定义的元素也可以），比如现在有个组件 
- `<my-component></my-component>`
- 使用ref `<my-component ref='new-name' attr-a='hello'></my-component>`
- 在 js  中 `this.$refs['new-name'].attr  // res hello` 就可以通过别名获取到元素，并且拿到元素对应的属性。

补充：

利用ref属性可以获取到dom元素或者是子组件，从而可以调用子组件的方法（注意2.0版本用ref取代了el）

1. 当ref直接定义在dom元素上时，则通过this.$refs.name可以获取到dom对dom进行原生的操作
- `<div class="foods-wrapper" ref="foods-wrapper">`
- 通过 `this.$refs` 获取到dom进行操作（注意ref属性的命名不能用驼峰，同时获取的时候也是）
- `let menuList=this.$refs['menu-wrapper'].getElementsByClassName('menu-list-hook');` 此处如果用 `this.$refs["menuWrapper"]` 将获取不到元素

2. 通过在引用的子组件上使用ref属性实现父组件调用子组件的方法以及属性
- 在父组件中引用子组件并定义ref
- `<v-food  ref="selectfood"></v-food>`
- 调用定义在子组件中的方法show
- `this.$refs.selectfood.show();` 同时也可以调用子组件中的属性

声明下上面说的是vue 2.0的

## template

template是html5的一个新元素，主要用于保存客户端中的内容，表现为浏览器解析该内容但不渲染出来，可以将一个模板视为正在被存储以供随后在文档中使用的一个内容片段。

## slot 插槽模板和非插槽模板

非插槽模板指的是html模板，比如 `div、span、ul、table` 这些，非插槽模板的显示与隐藏以及怎样显示由组件自身控制。

插槽模板是slot，它是一个空壳子，因为它的显示与隐藏以及最后用什么样的html模板显示由父组件控制。但是插槽显示的位置确由子组件自身决定，slot写在组件template的什么位置，父组件传过来的模板将来就显示在什么位置。

一般的用法就是在子组件里面：

```js

<!-- 子组件名称：<children> -->

<template>
    <div>
        <solt></solt>
    </div>
</template>

<!-- 父组件是这样的 -->

<template>
    <children>
        <span>被插入的内容，这整个span便签都会替换子组件中的solt</span>
    <children/>
</template>

```

这就是匿名插槽或叫做具名插槽，就是 `<span solt='name'> </span>` 在父组件上为要插入的内容取个名字，子组件`<solt name='name'></solt>` 这样来和父组件对应起来。

作用域插槽：这个概念比较难理解，先看怎么用:

```js

<!-- 父组件： -->

<template>
<div class="father">
    <h3>这里是父组件</h3>
    <!--第一次使用：用flex展示数据-->
    <child>
        <template slot-scope="user">
            <div class="tmpl">
            <span v-for="item in user.data">{ {item} }</span>
            </div>
        </template>
    </child>

    <!--第二次使用：用列表展示数据-->
    <child>
        <template slot-scope="user">
            <ul>
            <li v-for="item in user.data">{ {item} }</li>
            </ul>
        </template>
    </child>

    <!--第三次使用：直接显示数据-->
    <child>
        <template slot-scope="user">
        { {user.data} }
        </template>
    </child>

    <!--第四次使用：不使用其提供的数据, 作用域插槽退变成匿名插槽-->
    <child>
    我就是模板
    </child>
</div>
</template>

<!-- 子组件： -->
<template>
    <div class="child">
        <h3>这里是子组件</h3>
        // 作用域插槽
        <slot :data="data"></slot>
    </div>
</template>

export default {
    data: function(){
    return {
        data: ['zhangsan','lisi','wanwu','zhaoliu','tianqi','xiaoba']
    }
    }
}

```

可以看到，子组件写法是 `<slot :data="data"></slot>` 把数据绑定给data属性，而且数据的来源是子组件，这点就很重要了。 在父组件会这么写：

```html

<template slot-scope="scope">
    <span>{ {scope.row.id} }</span>
</template>

```

此时通过scope就可以拿到子组件绑定的data了，这个scope可以随便写。

在什么时候会用到呢？由于做研发比较少，但是用框架的时候你就要知道这种写法，通常会对子组件绑定父组件的数据，子组件拿到父组件的时候后，做了处理，得到自己的 data 就是上面插槽绑定的 data 这个时候你就可以去这个里面拿一些你像要的数据了。

像element UI 的table组件，通过给组件list数据，在 el-table-column 组件里面用作用域插槽就可以拿到赋值给list也就是表格的数据。

## this.$nextTick

在vue中，当页面加载完成以后，dom还没有加载，是无法获取进行操作的，但是在vue2.0中提供了一个方法 `this.$nextTick`，在这个回调函数里面写dom操作即可，如下代码：

```js

created() {
    this.$nextTick(() => {
    //do somthing
});　

```

其实这里还有一个小技巧，就是用settimeout(fn,20),来取代this.$nextTick,（20 ms 是一个经验值，每一个 Tick 约为 17 ms），对用户体验而言都是无感知的。

现在vue都快要到3.o了，不要使用settimeout了，在使用 `this.$nextTick` 如果失败了，很可能是生命周期相关问题没处理好。

## 路由跳转

当我们需要跳转一个页面的时候，既然是单页面应用，可以使用路由会很方便，比如带很多的参数过去。如果是普通的url跳转只能在url里面带参数，限制较大

比如我们的跳转由方法来处理 `@click="getDescribe(article.id)"`

方法内容(三种情况)：

```js

<!-- 情况1.基本使用 -->

this.$router.push({
    path: `/describe/${id}`,
})
<!-- 路由配置 -->
{
    path: '/describe/:id',
    name: 'Describe',
    component: Describe
}

<!-- 情况2.通过路由配置的name来匹配 -->

this.$router.push({
    name: 'Describe',
    params: {
    id: id
    }
})

<!-- 情况3.通过path来匹配 -->

this.$router.push({
    path: '/describe',
    query: {
    id: id
    }
})

```

方案2要优雅的多，可以在params中传递参数，这里的id用来做路由传参了。 

在子组件中通过 `$route.params` 获取到参数。方案3为 `$route.query` 就是获取 `$route` 对象的属性了。

运用：通过方法查询接口，返回数据由路由来响应，把参数都传给子组件，子组件通过在created生命周期中 `this.$route` 获取传递给子组件的参数。

## v-html 与 深度作用选择器

vue 使用v-html指令渲染的页面样式处理问题

由于是动态加载的页面，在style中写的class不会作用于v-html渲染的内容，作者给出的解决方案是给外层容器加个类名, 然后用后代选择器，css的选择器可以是类选到类 `.classA .classB`， 选择元素的 `.classA a` (选择a标签)

`.classA > a` 只对一代a标签作用。直接这样写还不行，需要深度作用选择器 `.classA >>> a`。 有些像 `Sass` 之类的预处理器无法正确解析 `>>>`。这种情况下你可以使用 `/deep/` 操作符取而代之——这是一个 `>>>` 的别名，同样可以正常工作。

总结：在使用指令的便签上加个类，用这个类选择后代（注意要用深度作用选择器）这样就可以解决问题了。

```html

<div class='myclass' v-html='content'></div>
<style>
    .myclass /deep/ a{
    font-size: 10px
    }
</style>

```

或者在被渲染的Html里面加style（没有测试过，感觉是可行的）

## 子组件向父组件传递事件

子组件向父组件传递事件，通常用来实现子组件向父组件传递值，然后调用父组件的方法

- 在子组件中对某个标签绑定点击事件 `v-on:click="$emit('click_event', data.guid)"`
- 这样在父组件中我们可以监听这个事件，`<article-classify v-on:click_event="classifyHandler"></article-classify>` 方法 `classifyHandler` 会接受传递的参数，也就是 `data.guid`，这样我们就拿到子组件传递来的参数了，然后后面的逻辑也就可以去跟着执行方法

## vuex

全局状态管理，修改store不要直接修改对象，应该通过调用mutations的方法来修改。 `store.commit("CHANGE_NAME")` 调用了CHANGE_NAME方法修改name属性

### mutations 和 actions

通过 module 分割模块，每个模块都可以包含 state、mutation、action、getter 属性，以减轻store对象的臃肿程度，记住，包含这些属性的就是一个store，通过module可以把多个store组织在一起成为全局的store

1. state: 就是属性存储的地方，类似组件的data

2. getter: 类似计算属性，getter 的返回值会根据它的依赖被缓存起来，且只有当它的依赖值发生了改变才会被重新计算。
this.$store.getters.doneTodosCount 访问getter设置的属性
通过函数的形式 store.getters.getTodoById(2) 此时不再缓存属性，每次都计算

3. mutation: 用于修改属性，里面定义修改属性的方法

```js
store.commit({
  type: 'increment',
  amount: 10
})
```

对象风格的调用方式：

- 添加新的属性：`state.obj = { ...state.obj, newProp: 123 }`，利用对象展开运算符加上新对象，组织成一个全新的对象重新赋值
使用this.$store.commit()提交即可

- actions: store.dispatch('increment') 的方式触发

```
默认情况下,模块内的getter, mutation, action是注册在全局空间的, state只注册在局部命名空间的
要想使模块内的getter,mutation,action注册在模块命名空间,必须在模块内加上 namespaced: true, 比如访问 store.getters['publish/useComponent']
```

### 辅助函数

mapState 计算属性中使用，就是需要获取store的属性并映射到计算属性的时候可以用它简化代码

```js
userNameTwo() {
      return this.$store.state.user.name
    },
```

通常你的计算属性需要通过this才能访问store，如果使用辅助函数
`userName: state => state.user.name, `
如果是同名的直接写字符串
`'login'`
总之就是少些一点代码

mapGetters 和上面的 mapState类似了 通过字符的形式就行了，或者通过对象的形式重命名
mapMutations 是 mutation 相关的辅助函数，可以在组件的methods中使用，把mutation映射成方法

### 和表单相关

由于修改属性需要提交，你不能直接使用v-model来绑定store的属性了，一个比较精简的用法

```js
<input v-model="message">
computed: {
  message: {
    get () {
      return this.$store.state.obj.message
    },
    set (value) {
      this.$store.commit('updateMessage', value)
    }
  }
}
```

## 报错处理

运行vue项目的时候，出现了[Vue warn]: Duplicate keys detected: '0'. This may cause an update error（错误，检测到重复的key值：”0“，这可能会导致更新错误）

错误原因：
我们在使用v-for的时候，都要必须加上一个唯一的key值，但是这里写了两个for循环，尽管都加上了key值,然而又将key的值写成一样的了。所以就导致了警告。

解决办法：
可以将其中一个的key修改一下即可。不行就两个都修改。

出错的地方：
写了两个一样的for循环，绑定的key相同。

```html
<div class="info" v-for="(item, index) in itemList" :key="index"></div>
<div class="info" v-for="(item, index) in itemList" :key="index"></div>
```

## 数组对象做绑定，关于响应式原理

`https://cn.vuejs.org/v2/guide/reactivity.html` 官方参考

场景问题，通过后端来的数据需要迭代，迭代出来的数据无法进行双向绑定

原因: `受现代 JavaScript 的限制 (而且 Object.observe 也已经被废弃)，Vue 无法检测到对象属性的添加或删除。由于 Vue 会在初始化实例时对属性执行 getter/setter 转化，所以属性必须在 data 对象上存在才能让 Vue 将它转换为响应式的`

这句话的意思是，假如数组一开始就有了，那么初始化的时候就会对各个属性进行响应式转换，使得我们在对迭代出来的数据进行v-model的时候可以生效，但是当数据是在初始化后加载来的，就不行了。

`解决方法`: 对于已经创建的实例，Vue 不允许动态添加根级别的响应式属性。但是，可以使用 Vue.set(object, propertyName, value) 方法向嵌套对象添加响应式属性，也就是新属性都通过Vue.set的方式添加，或`this.$set(el, 'isFold', false)`

假如通过el.isFold = false的方法加入的属性，是没有响应式支持的

- 多属性添加(没有测试过，详情看官网): 

有时你可能需要为已有对象赋值多个新属性，比如使用 Object.assign() 或 _.extend()。但是，这样添加到对象上的新属性不会触发更新。在这种情况下，你应该用原对象与要混合进去的对象的属性一起创建一个新的对象。

- 疑问

另外，还留有一点疑惑的地方，如果后端直接返回的数组，好像是可以响应式的，但是对这个数组添加的属性却不行

问题解决了，就是赋值给vue对象后，再添加属性不在具有响应式，需要用$set方法来添加，或者先添加完，再赋值

```js
// DataList为vue实例数据，data为借口返回数据

// 情况1，可以有响应式，通过this.$set赋值
that.DataList = data;
that.DataList.forEach(el => {
    this.$set(el, 'isFold', false);
    this.$set(el, 'checked', false);
})

// 情况2，可以有响应式，因为data被修改后才赋值给vue属性DataList
data.forEach(el => {
    el.isFold = false;
    el.checked = false;
})
that.DataList = data;

// 情况3，新增的属性不具有响应式，因为data先赋值给vue属性DataList，然后再去添加属性
that.DataList = data;
that.DataList.forEach(el => {
    el.isFold = false;
    el.checked = false;
})
```

## virtual Scroller

众所周知，渲染性能主要取决于渲染量， 过多的 DOM 会使你的网站速度变得很慢，即使在本地，也会出现相同的问题，但若通过虚拟滚动的方式，也就是在用户滚动时用足够的内容填充屏幕并不断更新，就能让网站保持高速运行，这也是 virtual-scroller 的基本原理。

## 缓存组件

```html
<keep-alive include="list">
    <router-view/>
</keep-alive>
```

另外路由有生命周期，可以在进入路由，跳去下一个路由的时候做处理，比如改变路由meta数据，然后关联keep-alive实现动态缓存组件



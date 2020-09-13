---
weight: 1000
title: "JavaScript 学习笔记"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "JavaScript 学习笔记"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [JavaScript, Note]
categories: [JavaScript]

lightgallery: true

toc:
  auto: false
---

javascript学习笔记

<!-- more -->

## DOM对象

DOM对象，即是我们用传统的方法(javascript)获得的对象，jQuery对象即是用jQuery类库的选择器获得的对象;
复制代码 代码如下:

```js
var domObj = document.getElementById("id"); //DOM对象
var $obj = $("#id"); //jQuery对象;
```

jQuery对象就是通过jQuery包装DOM对象后产生的对象，它是jQuery独有的。如果一个对象是jQuery对象，那么就可以使用jQuery里的方法，例:

$("#foo").html(); //获取id为foo的元素内的html代码，html()是jQuery特有的方法;

上面的那段代码等同于:

document.getElementById("foo").innerHTML;
$("#foo").innerHTML  是错误的

可以将jquery 和 dom  对象互相转换，这样dom对象就可以使用jquery的方法了，jquery对象亦如此。

## Json 方法

JSON.stringify(a) stringify()用于从一个对象解析出字符串

JSON.parse(str)  parse用于从一个字符串中解析出json对象

## 取得url中get请求的参数

```js
function getUrlParam(name){
	var reg = new RegExp("(^|&)" + name + "=([^&]*)(&|$)");
	var r = window.location.search.substr(1).match(reg);  
	if (r != null) return decodeURI(r[2]); return null;
}
```

## ready 和 onload事件

页面加载完成有两种事件，一是ready，表示文档结构已经加载完成（不包含图片等非文字媒体文件），二是onload，指示页 面包含图片等文件在内的所有元素都加载完成。(可以说：ready 在onload 前加载！！！) 一般样式控制的，比如图片大小控制放在onload 里面加载。 

## 关键字

JavaScript 关键字必须以字母、下划线（_）或美元符（$）开始。

后续的字符可以是字母、数字、下划线或美元符（数字是不允许作为首字符出现的，以便 JavaScript 可以轻易区分开关键字和数字）。

![image](/images/JavaScript/keyword.png)

## BOM & DOM

- BOM是浏览器对象模型，用来获取或设置浏览器的属性、行为，例如：新建窗口、获取屏幕分辨率、浏览器版本号等。
- DOM是文档对象模型，用来获取或设置文档中标签的属性，例如获取或者设置input表单的value值。
- BOM的内容不多，主要还是DOM。

## 字面量，变量

有时候会遇到字面量的概念，它和变量对应，字面量就是固定值的表示法。

## 异常

js也有异常，不过很少见人使用。

```html
<html>
<title>菜鸟教程(runoob.com)</title>
<script>
	function myFunction() {
	try {
		//错误判断
		var x = document.getElementById("demo").value;
		if (x == "") throw "值为空";
		if (isNaN(x)) throw "不是数字";
		if (x > 10) throw "太大";
		if (x < 5) throw "太小";
	} catch (err) {
		//发生错误时在此执行，err为自定义错误 throw 对应的值，
		var y = document.getElementById("mess");
		y.innerHTML = "错误：" + err + "。";
	}
	}
</script>

<body>

	<h1>我的第一个 JavaScript</h1>
	<p>请输出一个 5 到 10 之间的数字:</p>
	<input id="demo" type="text">
	<button type="button" onclick="myFunction()">测试输入</button>
	<p id="mess"></p>

</body>

</html>
```

## 函数

函数的定义方式大体有以下两种，浏览器对于不同的方式有不同的解析顺序。

```js
//“定义式”函数定义
function Fn1(){
alert("Hello World!");
}
//“赋值式”函数定义
var Fn2 = function(){
alert("Hello wild!");
}
```

## 快速测试一段代码的执行时间

```js
console.time('test')
/* 这里运行待测代码 */
console.timeEnd('test')
```

## 对象总结

对象

1. javascript 对象

```
JS Array     JS Boolean   JS Date  
JS Number    JS String    JS RegExp 
JS Functions JS Events    JS Math
```

其它对象

```
Browser  Window  Navigator 
Screen   History Location
```

Window 对象表示一个浏览器窗口或一个框架。在客户端 JavaScript 中，Window 对象是全局对象，所有的表达式都在当前的环境中计算。也就是说，要引用当前窗口根本不需要特殊的语法，可以把那个窗口的属性作为全局变量来使用。

2. HTML DOM 对象

- 每个载入浏览器的 HTML 文档都会成为 Document 对象。
- Document 对象使我们可以从脚本中对 HTML 页面中的所有元素进行访问。

Element 节点，文本节点，元素节点等。

Attribute 属性； Event 事件； HTML 对象；

标签即是HTML对象，标签和元素的区别，属性的定义：

    比如<p>这就是一个标签； 
    <p>这里是内容</p>这就是一个元素，
    也就是说元素由一个开始的标签和结束的标签组成，用来包含某些内容。

属性：

为HTML元素提供各种附加信息的就是HTML属性，它总是以"属性名=属性值"这种名值对的形式出现，而且属性总是在HTML元素的开始标签中进行定义。

节点的作用：

在有了标签，元素，属性后，引申出节点的概念，标签的元素中可能会有更多的元素，将多个或一个元素看作节点，节点就是为了去操作元素的。

## virtual DOM 

一些理解：

虚拟DOM，是一个模拟DOM数的js对象。 就是当我们需要更改DOM的时候，如果用原始方法比较慢，这在多节点的页面中体现就更明显了，原因是dom设计的复杂，所以我们用一个虚拟的DOM，虚拟的DOM记录了要更改的DOM，它通常不是立刻执行的，等到需要的时候，计算最小的执行，把执行更新到DOM上。这里为什么会有最小的DOM执行，是应为不是所有的地方都需要变更。

总结：virtual DOM 通过计算最小的DOM执行，能更快的渲染DOM。

别人的讲解：

- Virtual DOM 是一个模拟 DOM 树的 JavaScript 对象。 React 使用 Virtual DOM 来渲染 UI，当组件状态 state 有更改的时候，React 会自动调用组件的 render 方法重新渲染整个组件的 UI。
- React 主要的目标是提供一套不同的, 高效的方案来更新 DOM.不是通过直接把 DOM 变成可变的数据, 而是通过构建 “Virtual DOM”, 虚拟的 DOM, 随后 React 处- 理真实的 DOM 上的更新来进行模拟相应的更新。

引入额外的一个层怎么就更快了呢?

- 那不是意味着浏览器的 DOM 操作不是最优的, 如果在上边加上一层能让整体变快的话?是有这个意思, 只不过 virtual DOM 在语义上和真实的 DOM 有所差别.最主要的是, virtual DOM 的操作, 不保证马上就会产生真实的效果.这样就使得 React 能够等到事件循环的结尾, 而在之前完全不用操作真实的 DOM。在这基础上, React 计算出几乎最小的 diff, 以最小的步骤将 diff 作用到真实的 DOM 上。批量处理 DOM 操作和作用最少的 diff 是应用自身都能做到的.任何应用做了这个, 都能变得跟 React 一样地高效。但人工处理出来非常繁琐, 而且容易出错. React 可以替你做到。
- 前面提到 virtual DOM 和真实的 DOM 有着不用的语义, 但同时也有明显不同的 API。
- DOM 树上的节点被称为元素, 而 virtual DOM 是完全不同的抽象, 叫做 components。
- component 的使用在 React 里极为重要, 因为 components 的存在让计算 DOM diff 更高效。

简单的说就是：

当然如果真的这样大面积的操作 DOM，性能会是一个很大的问题，所以 React 实现了一个虚拟 DOM，组件 DOM 结构就是映射到这个虚拟 DOM 上，React 在这个虚拟 DOM 上实现了一个 diff 算法，当要更新组件的时候，会通过 diff 寻找到要变更的 DOM 节点，再把这个修改更新到浏览器实际的 DOM 节点上，所以实际上不是真的渲染整个 DOM 树。这个虚拟 DOM 是一个纯粹的 JS 数据结构，所以性能会比原生 DOM 快很多。

React的核心机制之一就是可以在内存中创建虚拟的DOM元素。React利用虚拟DOM来减少对实际DOM的操作从而提升性能。 

## indexOf

数组过滤。indexOf作用是返回字符串第一次出现在给定字符串的index，可以用来处理某个字符串有没有在给定字符串中。  给定 `str.indexOf(某个字符串) = 0` 说明第一个就匹配到，这个给定字符串。如果是空格分隔的，如几个单词，那么结果就不一定是0了，因为会在后面的位置。记住是给定来调用这个方法就行了

补充：

js array indexOf 参数是对象的时候，不一定能返回对应位置的index(有的时候可以，我查了资料，有人是这么说的：让数组去判断`一个新创建的对象`，所以会得到 -1。我在vue中，把循环出来的元素做为参数去在原数组中判断，是可以的，不是循环出来的对象，虽然对象和数组元素字面看起来一摸一样，但是不行，猜测这和底层有关) 所以这个东西的使用，要很小心
推荐使用 `Array.findIndex()`

findIndex()方法返回数组中满足提供的测试函数的第一个元素的索引。否则返回-1。

```js

var array1 = [5, 12, 8, 130, 44];

function findFirstLargeNumber(element) {
	return element > 13;
}

console.log(array1.findIndex(findFirstLargeNumber));
// expected output: 3

```

find() 方法返回数组中满足提供的测试函数的第一个元素的值。否则返回 undefined。

```js

var array1 = [5, 12, 8, 130, 44];

var found = array1.find(function(element) {
return element > 10;
});

console.log(found);
// expected output: 12

```

通过find，findIndex可以完成很多的事情，少用通过各种方法获取索引，然后再去 `array[index]`。find就可以了

更详细的使用查看文档。[文档地址](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/find)

我发现这个从列表中给出来的数据，你不段的引用，其中一个引用改了，也会影响到原数组。

## 格式化字符串  

```js
`a${var}` 
```
如果var是1，result 为 a1。注意两边的符号为tab键上面的

## bject.keys(obj)

返回值: 一个表示给定对象的所有可枚举属性的字符串数组

1. 传入字符串，返回索引
```js
var arr = ['a', 'b', 'c'];
console.log(Object.keys(arr)); // console: ['0', '1', '2']
```

2. 传入对象，返回属性名
```js
var obj = { a: 'alive', b: 'bike', c: 'color' };
console.log(Object.keys(obj)); // console: ['a', 'b', 'c']
```

## length  

只对字符串和数组有用，整形数字和对象返回未定义undefined

## includes  

数组调用，监测数组是否包含给定的元素 `array.include(0)` 返回boolean

## this

箭头函数与普通函数中的this指向不一样，前者基于定义时的上下文环境，后者则只是基于调用者。

## typeof cb == "function" && cb()

强大的js总有一些没见过的用法

`function delay(time, cb) { typeof cb == "function" && cb(time) } `

`cb&&cb(value)` 的意思是：
- 如果cb为真（有值），那么执行cb(value)；
- 如果cb为假，&&短路，那么不执行cb(value)。

## Date 日期对象

字符串转化为日期对象，将字符串作为参数实例化即可，new Date(2019 10 10)，传递时间戳也可以

时间对象格式化，建议使用扩展包，官方原生好像没有支持

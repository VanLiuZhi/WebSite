---
weight: 1000
title: "jQuery 学习笔记"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "jQuery 学习笔记"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [JavaScript, Note]
categories: [JavaScript]

lightgallery: true

toc:
  auto: false
---

javascript封装，快速编写前端代码，不过在MVVM框架的势头下，jQuery慢慢的不再需要了

<!-- more -->

## jQuery 方法

javascript封装，快速编写前端代码，不过在MVVM框架的势头下，jQuery慢慢的不再需要了

## each( )

如果是去迭代数组 类似 Inpute标签组成的 a:1,b:2：

使用 `$.each(对象，function（index,value）{   });` 这样可以拿到数组的键和值

如果是迭代元素 类似p标签的集合：

使用 `$("p").eache(function(index){  });` 在函数中，使用this得到当前迭代的元素  

## is()

is() 根据选择器、元素或 jQuery 对象来检测匹配元素集合，如果这些元素中至少有一个元素匹配给定的参数，则返回 true。

## parseFloat() 

解析一个字符串，并返回一个浮点数，参数必须且是一个字符

该函数先去判断第一个字符串是否是数字，不是，函数返回 NaN，是，继续执行。如果在解析过程中遇到了正负号（+ 或 -）、数字 (0-9)、小数点，或者科学记数法中的指数（e 或 E）以外的字符，则它会忽略该字符以及之后的所有字符，返回当前已经解析到的浮点数。同时参数字符串首位的空白符会被忽略。

NaN 属性是JS的Number对象，代表非数字值的特殊值。该属性用于指示某个值不是数字。可以把 Number 对象设置为该值，来指示其不是数字值。比如月份用数字5代表5月份，var number=5; number.NaN,number现在就不是数字了。

- isNaN()  该一个要检测的参数（必须）看是不是NaN，是就返回True 其它值返回False。
- parseInt() 函数可解析一个字符串，并返回一个整数。
- indexOf() 方法可返回某个指定的字符串值在字符串中首次出现的位置。即索引值。
- match() 方法可在字符串内检索指定的值，或找到一个或多个正则表达式的匹配。

## other

text（）用来设置或返回值，val()返回value的值

attr() 方法设置或返回被选元素的属性值。该方法不同的参数会有不同的效果

attr("id",123)   选择器得到的JQuery对象的attr方法，将对象的id 改变为 “123”（没有id直接添加一个id）

appendto() append()

append() 方法在被选元素的结尾（仍然在内部）插入指定内容。
提示：append() 和 appendTo() 方法执行的任务相同。不同之处在于：内容的位置和选择器。
例子：  

```
p标签原来的内容 <p>This is a paragraph.</p>
$("p").append(" <b>Hello world!</b>");   
执行方法后  This is a paragraph. Hello world!

$("<b> Hello World!</b>").appendTo("p");
p标签原内容和执行结果同上。
```

find（）方法 由给定表达式去匹配满足条件的后代元素，返回jquery对象。

```js
var b=$("#id").find('[name=id]');
```

window.location.href="url"  当前页面打开URL

当点击元素时，会发生 click 事件。

当鼠标指针停留在元素上方，然后按下并松开鼠标左键时，就会发生一次 click。

click() 方法触发 click 事件，或规定当发生 click 事件时运行的函数。

on 为元素绑定事件，比如click事件，然后加个函数，这个元素点击后就会去执行这个函数。

## html()

jquery渲染页面方法，`$.html()` 对dom执行html方法，会将dom的内容给替换了，比如 `<div class=123></div>`  对这个dom执行方法`html(<p>123</p>)` 结果是`<div class=123><p>123</p></div> `

## 元素切换

sildeup sildedown，show hide 元素切换隐藏

## jQuery遍历

- siblings：`dom.siblings(.class).addClass()` 对选择对象执行遍历，找到所有class类，并给他们添加样式。

## scroll()  

dom调用，可以在滚动条滚动的时候触发，只要有滚动就触发。这里注意如果逻辑涉及到滚动的数值判断，使用比较不要使用相等，因为滚动很快，相关的判断不一定每次执行到。

## 样式

直接写在dom上，相当于 `style : dom.css('color', 'red')`

hasClass()   addClass()   removClass()   对dom类的控制


## js拼接字符串，传递对象

在JS中拼接html字符串，遇到的坑往往是由书写格式引起的。
注意以下几点：

单双引号需嵌套使用；
传入 JSON 对象需要使用 JSON.stringify() 序列化为一个 JSON 字符串，并且更需要注意单双引号嵌套问题。
示例：

```html
let html = "";
for(var i=0;i<data.length;i++){
    html += '<tr>'+
            '<td>'+ data[i].appName +'</td>'+
            '<td>'+ '<button type="button" onclick="edit('+JSON.stringify(data[i]).replace(/\"/g,"'")+')">按钮1</button>'+'</td>'+
            '<td>'+ "<button type='button' onclick='del("+JSON.stringify(data[i])+")'>按钮2</button>"+'</td>'+
            +'</tr>'
}
```
上述代码中，使用了两种不同的引号嵌套格式。按钮1外层使用单引号，因此需要将JSON字符串中所有双引号转化成单引号，比较繁琐。从语法角度考虑，JavaScript 字符串与 JSON 字符串的最大区别在于，JSON 字符串必须使用双引号（单引号会导致语法错误），因此不推荐此写法。

推荐第二种写法，直接在外层嵌套双引号，无需进行额外操作

```js
htmlstr += "<a class='opt ib upd' href=" + 'javascript:void(0);' + " onclick='updateGetData(" + JSON.stringify(useData) + ")'>修改</a>"
```



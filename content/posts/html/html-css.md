---
weight: 1000
title: "css 在 web 开发中常见操作与学习总结"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "css 在 web 开发中常见操作与学习总结"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Css, Html]
categories: [Css]

lightgallery: true

toc:
  auto: false
---

css总结笔记，不属于ECMAScript的知识点一般整理到此，可能包括浏览器对象模型，DOM等

<!-- more -->

## 自动换行

style='word-wrap:break-word; word-break:break-all;display:block;width:100%;'

## 不换行  

white-space:nowrap

## input

style="-webkit-appearance:checkbox"   复选相关样式

## 空格

- 使用 \&nbsp; 
- 使用样式，输出1 &nbsp; 2 &nbsp; 3 &nbsp;

```html
<span style="white-space:pre">1  2  3</span>
```

## table

```s
<tr> - 定义表行
<th> - 定义表头
<td> - 定义表元(表格的具体数据)
```

## width

width  100%   auto

1. 某div不显示设置宽度，那么width为auto

2. 某div的width在默认情况设置的是盒子模型中content的值

3. 某div的width为100%表示的是此div盒子内容部分的宽度为其父元素的宽度

4. 某个div的width不设置，或者设置为auto，那么表示的这个div的所有部分（内容、边框、内边距等的距离加起来）为父元素宽度

## overflow

关于滚动条：overflow 决定了溢出的操作，设置溢出用滚动条，那么这个容器必须有固定高度，才有溢出的概念。

## scoped

scoped 属性是一个布尔属性。
如果使用该属性，则样式仅仅应用到 style 元素的父元素及其子元素。（在style type="text/css" scoped  这里使用）

## css3 选择器

- :nth-child(n)    ---->选中某个元素，该元素必须是某个父元素下的第n个子元素
- p:nth-child(n)   ---->选中p元素，且该p元素必须是某个父元素下的第n个子元素

如果n是数字，比如2，那么第2个作用样式，如果是n+2 那么从第2个开始，后面的都作用样式

## 默认行为

body 默认会有 margin 8px 这是浏览器决定的，不同浏览器不一样，所以初始创建一个项目，我们可以修改body 的css就可以0px 了

## 内联和块级元素

块级元素特点：

1、每个块级元素都从新的一行开始，并且其后的元素也另起一行。（真霸道，一个块级元素独占一行）

2、元素的高度、宽度、行高以及顶和底边距都可设置。

3、元素宽度在不设置的情况下，是它本身父容器的100%（和父元素的宽度一致），除非设定一个宽度。

内联元素(行内元素  inline)特点：

1、和其他元素都在一行上；

2、元素的高度、宽度及顶部和底部边距不可设置；

3、元素的宽度就是它包含的文字或图片的宽度，不可改变。

内联块级inline-block 元素特点：

1、和其他元素都在一行上；

2、元素的高度、宽度、行高以及顶和底边距都可设置。

## 盒子模型

CSS 盒子模型(Box Model)

所有HTML元素可以看作盒子，在CSS中，"box model"这一术语是用来设计和布局时使用。

CSS盒模型本质上是一个盒子，封装周围的HTML元素，它包括：边距，边框，填充，和实际内容。

盒模型允许我们在其它元素和周围元素边框之间的空间放置元素。

盒子模型居中显示，设置内联元素时 text-align:center;  这句话就可以让内联元素居中设置块状元素时定宽：

```html
<style>
div{
    border:1px solid red;/*为了显示居中效果明显为 div 设置了边框*/
    
    width:200px;/*定宽*/
    margin:20px auto;/* margin-left 与 margin-right 设置为 auto */
}
</style>
```

定宽就是width值固定，此时设置margin左或右为自动即可，注意两个设置缺一不可。
  
不定宽：

加table，或者将其改为内联

在实际工作中我们会遇到需要为“不定宽度的块状元素”设置居中，比如网页上的分页导航，因为分页的数量是不确定的，所以我们不能通过设置宽度来限制它的弹性。(不定宽块状元素：块状元素的宽度width不固定。)不定宽度的块状元素有三种方法居中（这三种方法目前使用的都很多）：
- 加入 table 标签
- 设置 display: inline 方法：与第一种类似，显示类型设为 行内元素，进行不定宽元素的属性设置
- 设置 position:relative 和 left:50%：利用 相对定位 的方式，将元素向左偏移 50% ，即达到居中的目的
第一种方法：
为什么选择方法一加入table标签? 是利用table标签的长度自适应性---即不定义其长度也不默认父元素body的长度（table其长度根据其内文本长度决定），因此可以看做一个定宽度块元素，然后再利用定宽度块状居中的margin的方法，使其水平居中。

第一步：为需要设置的居中的元素外面加入一个 table 标签 ( 包括 \<tbody>、\<tr>、\<td> )。

第二步：为这个 table 设置“左右 margin 居中”（这个和定宽块状元素的方法一样）。

举例如下：

html代码：
```html
<div>
 <table>
  <tbody>
    <tr><td>
    <ul>
        <li>我是第一行文本</li>
        <li>我是第二行文本</li>
        <li>我是第三行文本</li>
    </ul>
    </td></tr>
  </tbody>
 </table>
</div>
css代码：
<style>
table{
    border:1px solid;
    margin:0 auto;
}
</style>
```

隐形改变display类型
1. position : absolute 
2. float : left 或 float:right 加入这两句话中任意一句，就可以将元素变化inline-block

```css
.container a{
    position:absolute;
	width:200px;
	background:#ccc;
}
```

a原来是内联的，不能改变width，加入绝对定位后变为内联块就可以了。

width   height是改变盒子的背景background,    font-size才是改变文字大小

## 权值

```css
p{color:red;} /*权值为1*/
p span{color:green;} /*权值为1+1=2*/
.warning{color:white;} /*权值为10*/
p span.warning{color:purple;} /*权值为1+1+10=12*/
#footer .note p{color:yellow;} /*权值为100+10+1=111*/
```

浏览器默认的样式 < 网页制作者样式 < 用户自己设置的样式

## 定位

层模型有三种形式：

1、绝对定位(position: absolute)

2、相对定位(position: relative)

3、固定定位(position: fixed)

Relative与Absolute组合使用

在三种形式中，1是相对于浏览器，2是移动后原来位置有保留，3是固定盒子 `利用2，1可以实现相对与某一个的移动（不用相对于浏览器）`

```css
#box1{
    width:200px;
    height:200px;
    position:relative;        
}

#box2{
    position:absolute;
    top:20px;
    left:30px;         
}
```

这样一来，box2就是相对于box1来移动的

注意这样做的前提

参照定位的元素box1必须是相对定位元素box2的前辈元素

即：

```html
<div id="box1">
	<div id="box2">相对参照元素进行定位</div>
</div>
```

关于代码编写时移动的问题

绝对定位中，比如实现div元素相对于浏览器右移100px

此时代码写为  left:100px；

注意理解绝对定位，先把我的盒子定位好，来移动浏览器，这样为了让我在浏览器的右边，浏览器应该左移，绝对定位。

另一个理解方法

有四个参数来设置   left  right  top  bottom

比如设置left=50px   可以理解为此时盒子与浏览器左边相距50px，也就是盒子右移动50px

补充：
1. static（静态定位）：默认值。没有定位，元素出现在正常的流中（忽略 top, bottom, left, right 或者 z-index 声明）。
2. relative（相对定位）：生成相对定位的元素，通过top,bottom,left,right的设置相对于其正常（原先本身）位置进行定位。可通过z-index进行层次分级。　　
3. absolute（绝对定位）：生成绝对定位的元素，相对于 static 定位以外的第一个父元素进行定位。元素的位置通过 “left”, “top”, “right” 以及 “bottom” 属性进行规定。可通过z-index进行层次分级。
4. fixed（固定定位）：生成绝对定位的元素，相对于浏览器窗口进行定位。元素的位置通过 “left”, “top”, “right” 以及 “bottom” 属性进行规定。可通过z-index进行层次分级。

css 相对固定位置 处理方案：

需要固定位置的时候，常使用 `position：fixed` 这种方式往往达不到理想的效果，我们经常需要的是容器在某个位置固定，这样你需要花时间去调整左右上下参数。可以让fixed定位的容器处在一个父容器内，父容器是relative定位的，这样我们固定定位的容器就不用调整位置了，以父容器为基准

举例：

```html
    
<div style="position: relative;">
        <div class="rich_text_class" style="position:fixed;" v-html="menu">
        </div>
</div>

```

## index-z  

z序决定了dom的层级关系，数值越大的在最上层。关于宽度，父容器的宽度受到子dom的影响，可以调整让子dom的宽度不超过父容器，在设置width属性的时候，是块的宽加上内边距，可以改属性，让这个width只有块的宽决定。理解外边距和内边距和元素自己。块级元素才是占一行。

## :hover 

`:hover { }` css选择器中对被选中对象做操作，效果为鼠标指向时代码作用。例子鼠标指向时修改文字颜色（注意不能直接在便签中加style=color  先设置颜色，class设置的颜色就会失效，应该是遵循了就近原则）

## 超出显示文字省略

css3 超出显示文字省略 `text-overflow: ellipsis; white-space: nowrap; overflow: hidden;`

设置行数，超出行数省略

    overflow: hidden;
    text-overflow: ellipsis;
    display: -webkit-box;
    -webkit-line-clamp: 3;
    -webkit-box-orient: vertical;

把该样式作用于p标签上

## 控制锚点

1. js控制锚点跳转
```
<a name="anchor"></a>

location.hash="anchor";

不只有a其他元素也可以，比如在报表中：
<tr id="tr1">...</tr>
location.hash="tr1"

或者用jQuery的动画滚动效果：
var id="tr1";
$('html,body').animate({scrollTop: $("tr#"+id).offset().top}, 500);
```

2. html控制锚点跳转
```
<a href="#btn">跳转到点击位置</a>
<a name="btn" id="btn">点击</a>
```

3. 跨页面锚点跳转
```
代码如下:
<a href="123.html#btn">跳到btn</a>
<a name="btn" id="btn"></a>
```

4. js控制锚点跳转在HTML中实现方式

```
<!-- 假设一个需要跳转到的节点 -->
<div id="divNode"><!-- contents --></div>

<a href="#" onclick="
    document.getElemetnById('divNode').scrollIntoView(true);
    return false;">
    通过scrollIntoView实现锚点效果
</a>  
```

## box-sizing

box-sizing属性用于更改用于计算元素宽度和高度的默认的 CSS 盒子模型。可以使用此属性来模拟不正确支持CSS盒子模型规范的浏览器的行为。

简单来说就是改变盒子模型的计算方式，默认盒子模型，容器的width等于容器设置的width。如果box-sizing 设置为border-box 容器的宽度等于除外边距外的其它属性和。

## cursor: pointer  

`cursor: pointer` 运用了该属性，鼠标指上元素时变成“小手”

## 其它

给元素添加背景，做一个图标按钮，需要设置padding 扩充元素大小，不然图片没法显示

## 滚动条控制与计算相关属性

理解可视区域；div整体高度，包括因为滚动条看不见的部分

scrollTop：可视区域滚动的时候，顶部和div整体高度的顶部之间的距离

clientHeight：可视区域高度

scrollHeight：div整体高度

element.scrollHeight - element.scrollTop === element.clientHeight // 判断是否到底

element.scrollTop = 0 // 返回顶部

注意，滚动条的scrollTop等属性是设置div超过当前高度滚动时才会有的，如果设置div高度2000px，此时出现的滚动条是浏览器的机制，该属性scrollTop为0

## calc 计算函数

`:style="{ display: 'inline-block', width: 'calc(50% - 12px)' }"` 
可以实现动态的高度，比如一个dom1要使用剩余的20%的高度，但是已经存在了一个固定高度为20px的dom2，其它元素已经使用了剩下的80%，通过计算`calc(20% - 12px)`保证dom1加上dom2有20%

## event 事件

事件的回调函数是有参数的，默认有这个参数event，这个参数对象包含了事件的很多属性

event.preventDefault(); 取消事件的默认行为，因为有些事件它就是有默认行为的，可以用这个取消

## pre 
标签原样展示文本内容，前提是字体设置为等宽字体`font-family: monospace;`
`<pre style='word-wrap: break-word;white-space: pre-wrap;'>` 自动换行

## a标签补充

<a href='javascript:void(0);'> 很多时候我们这样写，让a标签不跳转，增加onclick可以去执行方法，并且保持一个超链接的样式

```html
<a href='javascript:void(0);' onclick='deposit_volume(this)' class='deposit_volume'>
```



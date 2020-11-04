---
weight: 8800
title: "ECMAScript 高级"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "ECMAScript 高级"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [JavaScript, Note]
categories: [FrontEnd(前端)]

lightgallery: true

toc:
  auto: false
---

ECMAScript 高级话题

<!-- more -->

## 补充

原始值和引用值

null 和 undefined

isNaN toString parseInt parseFloat

## 面向对象

constructor   Prototype

constructor
对创建对象的函数的引用（指针）。对于 Object 对象，该指针指向原始的 Object() 函数。
Prototype
对该对象的对象原型的引用。对于所有的对象，它默认返回 Object 对象的一个实例。

hasOwnProperty(property)
判断对象是否有某个特定的属性。必须用字符串指定该属性。（例如，o.hasOwnProperty("name")）
IsPrototypeOf(object)
判断该对象是否为另一个对象的原型。
PropertyIsEnumerable
判断给定的属性是否可以用 for...in 语句进行枚举。
ToString()
返回对象的原始字符串表示。对于 Object 对象，ECMA-262 没有定义这个值，所以不同的 ECMAScript 实现具有不同的值。
ValueOf()
返回最适合该对象的原始值。对于许多对象，该方法返回的值都与 ToString() 的返回值相同。

极晚绑定: 就是在创建了实例后，再把属性绑定到对象的原型上，一般都是在创建实例之前操作。不建议使用这种方式

继承：call apply 作用将函数绑定到另一个对象上执行，第一个参数都是要绑定的对象，后面的参数call可以是多个，二apply是一个对象

在面向对象中，对象的构建没有官方的用法，继承也有多种，不过有原型链来规范继承。

arguments，函数参数个数是不会进行验证的，Function 对象也有与所有对象共享的 valueOf() 方法和 toString() 方法。这两个方法返回的都是函数的源代码

ECMAScript 

## let const 命令

let 不存在变量提升，体现的是块级作用域，for循环中最后使用let，避免泄漏变成全局变量
ES6 允许了在块级作用域中声明函数，ES6之前是非法的，不过各种浏览器没有管这个事情
```js
// 函数声明语句
{
  let a = 'secret';
  function f() {
    return a;
  }
}

// 函数表达式
{
  let a = 'secret';
  let f = function () {
    return a;
  };
}
```

为了统一规范，使用函数表达式来进行块级作用域的函数声明

在代码块内，使用let命令声明变量之前，该变量都是不可用的。这在语法上，称为“暂时性死区”（temporal dead zone，简称 TDZ

const 声明不赋值会报错，除了不可变，其它的性质大体和let相似，是块级作用域，变量不提升，有暂时性死区

声明常量，只是变量指向的数据指针不能改，对了可变对象，要修改对象内容不是const来控制的

const foo = Object.freeze({}); 可以创建冻结对象，对象的属性一开始就要声明好，因为冻结对象是不可变的

顶层对象，在浏览器环境指的是window对象，在 Node 指的是global对象。

## promise

基本用法，可以嵌套

```js
const promise = new Promise(function(resolve, reject) {
  // ... some code
  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});

promise
// 调用成功后要做的事情
.then(value=>{

})
// 异常处理
.catch(error=>{

})

```


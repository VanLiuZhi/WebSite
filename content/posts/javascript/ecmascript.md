---
weight: 1
title: "ECMAScript6 学习笔记"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "ECMAScript6 学习笔记"
resources:
- name: "base-image"
  src: "base-image.jpg"

tags: [JavaScript, Note]
categories: [JavaScript]

lightgallery: true

toc:
  auto: false
---

ECMAScript6 是 JavaScript 的第六版本，是一个标准，主要增加了面向对象的支持和一些新特性。

<!-- more -->

## 扩展运算符（ spread ）

是三个点（...）。它好比 rest 参数的逆运算，将一个数组转为用逗号分隔的参数序列。

```js
console.log(...[1, 2, 3])
// 1 2 3
console.log(1, ...[2, 3, 4], 5)
// 1 2 3 4 5
[...document.querySelectorAll('div')]
```

## 箭头函数

```js
  const Person = {
    'name': 'little bear',
    'age': 18,
    'sayHello': function () {
      setInterval(function () {
        console.log('我叫' + this.Person + '我今年' + this.age + '岁!')
      }, 1000)
    }
  }
  Person.sayHello()
```

基础语法

- (参数1, 参数2, …, 参数N) => {函数声明}
- (参数1, 参数2, …, 参数N) => 表达式（单一）

相当于：(参数1, 参数2, …, 参数N) =>{ return表达式}

1. 当只有一个参数时，圆括号是可选的：
- (单一参数) => {函数声明}
- 单一参数 => {函数声明}

2. 没有参数的函数应该写成一对圆括号。
- () => {函数声明}

在上面的代码里面，谁调用，this指向谁，所以this指向的是setInterval, 就是window（因为setInterval）是window就注入的函数。 所以在setInterval 上一行，我们可以var self = this此时的this是sayHello 由Person来调用，这样才能得到我们想要的结果。

箭头函数最大特点： 不绑定this  不绑定arguments

es6箭头函数，这个是原来定义函数的缩写。let 和原来的 var 类似，var是声明变量，它所处的位置决定了变量的作用域，比如在函数里面就是函数的作用于，在外部就是全局作用域。let也是这样，但是它的位置决定的是最接近的块的作用域，作用域比var更细，除了函数全局外，如果你用在for，if里面，那么在整个函数里面是不可见的。所以可以用let声明作用域更细的变量。

## 继承

理解继承的机制

```js
function DOG(name){
this.name = name
}
```

这个函数我们称为构造函数，js通过对构造函数使用new 关键字创建实例（构造函数相当于Class），这样我们就从原型对象生产了一个实例对象。

1. 共有属性：

- 这样创建的实例没有共有属性，于是通过为构造函数设置prototype属性，来让从这个构造函数创建的实例都有共有属性。
- 这个属性包含一个对象（以下简称"prototype对象"），所有实例对象需要共享的属性和方法，都放在这个对象里面；那些不需要共享的属性和方法，就放在构造函数里面。

实例对象一旦创建，将自动引用prototype对象的属性和方法。也就是说，实例对象的属性和方法，分成两种，一种是本地的，另一种是引用的。

这个prototype是大家共同引用的，修改它会影响实例。


2. constructor：

通过构造函数创建的实例，访问这个属性就可以知道实例的构造函数是谁。

`cat1 instanceof Cat` 判断实例cat1是否是通过构造函数Cat来的，类似python的isinstance。

```
6.1 isPrototypeOf()

这个方法用来判断，某个proptotype对象和某个实例之间的关系。

　　alert(Cat.prototype.isPrototypeOf(cat1)); //true

　　alert(Cat.prototype.isPrototypeOf(cat2)); //true

6.2 hasOwnProperty()

每个实例对象都有一个hasOwnProperty()方法，用来判断某一个属性到底是本地属性，还是继承自prototype对象的属性。

　　alert(cat1.hasOwnProperty("name")); // true

　　alert(cat1.hasOwnProperty("type")); // false
```

```
prototype constructor
__proto__
```

3. 普通对象

- 最普通的对象：有__proto__属性（指向其原型链），没有prototype属性。
- 原型对象(person.prototype 原型对象还有constructor属性（指向构造函数对象）)。

4. 函数对象：
- 凡是通过new Function()创建的都是函数对象。
- 拥有__proto__、prototype属性（指向原型对象）。
- Function、Object、Array、Date、String、自定义函数。

特例： Function.prototype(是原型对象，却是函数对象，下面会有解释)

4. 如何判断是什么对象 typeof 对象

其实原型对象就是构造函数的一个实例对象。person.prototype就是person的一个实例对象。相当于在person创建的时候，自动创建了一个它的实例，并且把这个实例赋值给了prototype。

## 早绑定和晚绑定

所谓绑定（binding），即把对象的接口与对象实例结合在一起的方法。

早绑定（early binding）是指在实例化对象之前定义它的属性和方法，这样编译器或解释程序就能够提前转换机器代码。在 Java 和 Visual Basic 这样的语言中，有了早绑定，就可以在开发环境中使用 IntelliSense（即给开发者提供对象中属性和方法列表的功能）。ECMAScript 不是强类型语言，所以不支持早绑定。

另一方面，晚绑定（late binding）指的是编译器或解释程序在运行前，不知道对象的类型。使用晚绑定，无需检查对象的类型，只需检查对象是否支持属性和方法即可。ECMAScript 中的所有变量都采用晚绑定方法。这样就允许执行大量的对象操作，而无任何惩罚。

## 文件导入

export default 和 export 区别：

1. export与export default均可用于导出常量、函数、文件、模块等
2. 你可以在其它文件或模块中通过import+(常量 | 函数 | 文件 | 模块)名的方式，将其导入，以便能够对其进行使用
3. 在一个文件或模块中，export、import可以有多个，export default仅有一个
4. 通过export方式导出，在导入时要加{ }，export default则不需要

1. export

```js
//a.js
export const str = "blablabla~";
export function log(sth) { 
  return sth;
}
```

对应的导入方式：

```js
//b.js
import { str, log } from 'a'; //也可以分开写两次，导入的时候带花括号
```

2. export default

```js
//a.js
const str = "blablabla~";
export default str;
```

对应的导入方式：

```js
//b.js
import str from 'a'; //导入的时候没有花括号
```

## Object.assign 

`Object.assign({}, row)` 拷贝对象

## import和require的区别

node编程中最重要的思想就是模块化，import和require都是被模块化所使用。

1. 遵循规范
- require 是 AMD规范引入方式
- import是es6的一个语法标准，如果要兼容浏览器的话必须转化成es5的语法

2. 调用时间
- require是运行时调用，所以require理论上可以运用在代码的任何地方
- import是编译时调用，所以必须放在文件开头

3. 本质
- require是赋值过程，其实require的结果就是对象、数字、字符串、函数等，再把require的结果赋值给某个变量
- import是解构过程，但是目前所有的引擎都还没有实现import，我们在node中使用babel支持ES6，也仅仅是将ES6转码为ES5再执行，import语法会被转码为require

引用同级文件 `a.js`，`b.js` 都在一起，应该 `import  ./b` 不要直接 `import b`

1，给vue组件绑定事件时候，必须加上native ，不然不会生效（监听根元素的原生事件，使用 .native 修饰符）
2，等同于在自组件中：
   子组件内部处理click事件然后向外发送click事件：$emit("click".fn)

## Object.keys(object)

Object.keys(object) 对一个对象使用，返回对象键值组成的数组，对象为{}，返回空数组[]，可以Object.keys(object).length的方式判断对象是否为空

## forEach 跳出迭代和终止迭代

总的来说continue和break是for循环才有的，forEach是迭代数组，不应该有这个功能，但有时又需要
continue可以通过return true来实现
break通过try语句来实现，在需要终止的地方，通过throw new Error("EndIterative")手动触发异常来结束后续的循环

## hasOwnProperty

判断对象是否拥有这个属性 data.hasOwnProperty('name') 在for循环一个对象中，可能需要这个检测，规范问题

## slice 字符串数组切片

"name".slice(0, -1) 从开始切片到倒数第一个，name变成nam，该方法数组也拥有

## in 语法

和Python类似，判断属性是否在对象或原型中

## async 和 await

使用示例，async封装一个Promise对象，await等待执行结果返回，下列的searchModule方法被改造成一个Promise对象，
await等待一个ajax请求返回，通过then的形式拿到结果(注意这里只是一个简单情况，Promise还可以返回错误情况)

使用场景，把接口请求改成阻塞式的，虽然是异步的，但是阻塞了，等待请求结果。
针对需要先拿到请求结果再去做操作的情况，这部分逻辑不用写在接口里面了，可以在调用接口的时候写在then里面

```js
async searchModule(params) {
  await searchModule({keyName: params}).then(res => {
    let code = res.data.code;
    if (code === 200) {
      this.restaurants = res.data.data || [];
    } else {
      that.$message({
        message: res.data.msg,
        type: "warning"
      });
    }
  })
}

this.searchModule(queryString).then(() => {
  let restaurants = this.restaurants;
  cb(restaurants);
});
```

---
weight: 1000
title: "BootStrap Table 通过回调解决频繁拼接字符串带来的问题"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "BootStrap Table 通过回调解决频繁拼接字符串带来的问题"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Technology, Web, JavaScript]
categories: [Technology技术]

lightgallery: true

toc:
  auto: false
---

## 问题

很多人喜欢在编写前端代码的时候，通过字符串拼接的方式去组合DOM对象，虽然浏览器也能渲染，但是有很多弊端

1. 比如单双引号问题
2. 函数本身参数传递受到限制，只能传递字符串，你说一个函数不能传递指针或者函数引用也就算了，连基本的数组，对象都不能传递，这会导致编码复杂程度大幅度上升，造成代码冗余
3. 拼接的方式是提前渲染的，导致html标签很复杂

**这也是Jquery等传统框架的弊端，不能实现工程化**


比如下面的情况，仅仅是格式化一个字符串还行

```js
{
    field: "updateDate",
    title: "时间",
    visible: false,
    formatter: function (value, row, index) {
        return '更新:' + row.updateDate.substring(2, 16);
    }
}
```

如果是操作相关的，就会出现上面说到的两点限制，对于复杂的场景非常不适应

```js
{
    field: "id",
    title: "操作",
    width: 70,
    formatter: function (value, row, index) {
        return '<div class="dropdown-toggle authority" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">' +
            ' 操作' +
            '</div>' +
            '<div class="dropdown-menu" >' +
            '<a class="dropdown-item authority" href="javascript:processStatus(\'' + row.id + '\')">已处理</a>' +
            '<a class="dropdown-item authority" href="javascript:ignoreStatus(\'' + row.id + '\')">忽略</a>' +
            '</div>';
    }
}
```

## 有两种解决方案

方案1：绑定事件

```s
onCheck            check.bs.table：当用户选择某一行时触发，参数包含： row：与点击行对应的记录， $element：选择的DOM元素。

onClickRow       click-row.bs.table：当用户点击某一行的时候触发，参数包括： row：点击行的数据， $element：tr 元素， field：点击列的 field 名称。

onClickCell        click-cell.bs.table：当用户点击某一列的时候触发，参数包括： field：点击列的 field 名称， value：点击列的 value 值， row：点击列的整行数据， $element：td 元素。
```

比如弹窗的场景，我们只渲染DOM，通过事件绑定来执行逻辑

```js
{
    field: "id",
    title: "编号",
    formatter: function (value, row, index) {
        return "<a href=javascript:void(0)>" + row.id + "</a>";
    }
},
```

加一个事件，由于对每一个Cell都是生效的，注意自己判断一下，这样回调的方法中就能传递对象了

```js
onClickCell: function (field, value, row, $element) {
    if (field === "id") {
        openLog(row.alarmMessage)
    }
}
```

方案2:

也是通过事件绑定，比起方案1要灵活很多，事件只绑定到当前cell，在事件的回调中也能传递对象了

```js
{
    field: "op",
    title: "操作",
    width: 70,
    events: {
        'click #TableProcess': function (e, value, row, index) {
            processStatus(row.id)
        },
        'click #TableIgnore': function (e, value, row, index) {
            ignoreStatus(row.id)
        }
    },
    formatter: function (value, row, index) {
        return '<div class="dropdown-toggle authority" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">' +
            ' 操作' +
            '</div>' +
            '<div class="dropdown-menu" >' +
            '<button class="dropdown-item authority" id="TableProcess" type="button">已处理</button>' +
            '<button class="dropdown-item authority" id="TableIgnore" type="button">忽略</button>' +
            '</div>';
    }
},
```

需要特别注意的是，框架本身通过这种方式绑定事件，需要field字段唯一，很多情况对于操作这行是没有多余的字段去绑定的，如果field不是唯一的，绑定事件就会失效

解决方案就是在数据赋值给table的时候，冗余一个字段上去，如果本身有可以用的多余字段就不必执行这个过程了

```js
responseHandler: function (res) {
    var raw = res.data.rows
    var _raw = []
    raw.forEach(e => {
        e.op = e.id;  // 新加一个op属性给操作行用
        _raw.push(e)
    })
    return {
        "total": res.data.total, //总页数
        "rows": _raw //数据
    };
},
```
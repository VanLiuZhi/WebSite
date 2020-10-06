---
weight: 8800
title: "G2"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "G2"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [JavaScript, Note]
categories: [FrontEnd(前端)]

lightgallery: true

toc:
  auto: false
---

G2，一种全新的画图工具，有自己的一套设计语言

<!-- more -->

## 基础

### Chart 对象和 View 对象

G2 的图表可以由多个视图 View 构成，同时各个视图可以拥有各自的数据源，即支持异构数据。在结构上，视图和 chart 相同，拥有自己独立的数据源、坐标系和图层。
Chart 是一种特殊的View，两者之间也有一定的差异：
View 的功能 Chart 都具有
Tooltip（提示信息）和 Legend（图例）仅在 Chart 上支持
Chart 上可以创建 View ，但是 View 不再支持创建子 View

视图，由 Chart 生成和管理，拥有自己独立的数据源、坐标系和图层，用于异构数据的可视化以及图表组合，一个 Chart 由一个或者多个视图 View 组成。因此 view 上的 api 同 chart 基本相同。

### 关注数据Data和Scale度量两个重要概念

列定义：操作度量Scale称为列定义

Geom几何标记：决定最后画出来的是什么图形，与之相关的是图形属性Attr

Attr图形属性

1. position：位置，二维坐标系内映射至 x 轴、y 轴；
2. color：颜色，包含了色调、饱和度和亮度；
3. size：大小，不同的几何标记对大小的定义有差异；
4. shape：形状，几何标记的形状决定了某个具体图表类型的表现形式，例如点图，可以使用圆点、三角形、图片表示；线图可以有折线、曲线、点线等表现形式；
5. opacity：透明度，图形的透明度，这个属性从某种意义上来说可以使用颜色代替，需要使用 'rgba' 的形式，所以在 G2 中我们独立出来。

其中shape是重要的关注点

Legend图列：图表辅助元素，使用颜色、大小、形状区分不同的数据类型，用于图表中数据的筛选

Guide辅助元素：在图表上添加额外的注解，比如加辅助线，文本

## DataSet

Transform 数据变换函数，数据视图做数据处理时使用，包括图布局、数据补全、数据过滤等等

通过type决定如何处理数据，fold用于数据字段展开，filter用于数据过滤等

## 技巧

图表自适应

```js
const chart = new G2.Chart({
  container: 'c1',
  forceFit: true,
  height : 400
});
```

坐标轴格式化

```js
chart.axis('field', {
  label: {
    formatter: val => {
      return val + 'k';
    }
  }
});
```
---
weight: 1000
title: "javascript-definitive-guide 读书笔记"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "javascript-definitive-guide 读书笔记"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [ReadBook]
categories: [ReadBook]

lightgallery: true

toc:
  auto: false
---

JavaScript 权威指南 读书笔记，节选部分内容

<!-- more -->

## 关于分号

关于分号，分号可以不要，如果你不要分号，语句会被解释器连起来。

比如：

```js
  var x = y + z
  (a+b).result()
```

实际是 `var x = y + z(a+b).result()` 分号是语句的结束，你不加分号解释器会自己来处理，它处理不了的语句会自己加分号，等等一些规则，所以养成加分号是好习惯，避免前面的语句改了，没有分号语句被解释器组合了。
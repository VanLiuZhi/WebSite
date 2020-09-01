---
weight: 1
title: "yaml 标记语言"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "yaml 标记语言"
resources:
- name: "base-image"
  src: "base-image.jpg"

tags: [Yaml, Other]
categories: [Other]

lightgallery: true

toc:
  auto: false
---

yaml 语言

<!-- more -->

## 基本语法

- 大小写敏感
- 使用缩进表示层级关系
- 缩进时不允许使用Tab键，只允许使用空格。
- 缩进的空格数目不重要，只要相同层级的元素左侧对齐即可

## 注释

YAML 支持的数据结构有三种

对象：键值对的集合，又称为映射（mapping）/ 哈希（hashes） / 字典（dictionary）

数组：一组按次序排列的值，又称为序列（sequence） / 列表（list）

纯量（scalars）：单个的、不可再分的值

## python 操作

load  和 dump 方法
load 加载 yaml 文件   dump将数据写入yaml文件

先创建一个文件对象 ，并加载 `stream = file('example.yaml','r') dicts = yaml.load(stream)` 这个时候，我们就可以去取值了

举例：
yaml文件为

```yaml
url:
- www.baidu.com
- www.taobao.com
- vlcoa.inruan.com 
```

这个属于数组，取值时用索引就可以了 `print dicts['url'][1]` 结果是 `www.taobao.com`

yaml文件为

```yaml
language:
        top1:
            python
        top2:
               java
```

属于对象，language是一个字典 `print dicts['language']['top1']` 结果 `python`

比较简单的配置文件：

    settime: 5     

load后，`dicts[settime]`，就可以取到5了，冒号后面一点要有空格，使用数组就可以使用索引去取值，而且dicts到的是列表，可以去迭代参数，必须要用  '-' 加空格的形式，不能使用Tab键来缩进，一定要用空格。
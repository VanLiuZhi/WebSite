---
weight: 7700
title: "vim 编辑器操作"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "vim 编辑器操作"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Linux, Note]
categories: [Linux]

lightgallery: true

toc:
  auto: false
---

vim的使用

<!-- more -->

## 模式

- 导航(navigation)模式： 这时候，字母就是上下左右键。
- 输入模式：你按字母键，才会输入字母。
- 命令模式：需要先输入":" 冒号，才会进入。例如，你输入 :ls , 就相当于运行了 ls 命令。

## 粘贴模式(常用)

相信你一定遇到过把本地的代码粘贴到Vim出现排版错乱问题，Vim 正常模式下的粘贴，会导致粘贴的代码一行接一行的缩进。如果要取消这种缩进的话，就要进入到 "粘贴模式". （记得在这个模式下，无法使用 ctrl + t 命令来快速打开文件。）

    :set paste 进入到粘贴模式
    :set nopaste 取消粘贴模式

## 光标操作

    h 左 l 右
    j 下 k 上

    w: 下一个词。 (word)
    b: 上一个词。 (backword)

选择文本

    按住 v 加方向键


## 复制粘贴

    y#复制反白的地方

    d#删除反白的地方

    yy#复制光标所在的那一行（常用）

    dd#删除光标所在的那一行（常用）

## 文本替换

当前文件中替换

    :%s/原来的字符串/新字符串/

替换匹配的所有文本

    :%s/原来的字符串/新字符串/g

局部替换

    先v或 shift + v 选中若干行，然后:s/原来的/新的字符串

## 搜索

    / #搜索 some_thing: 
    n #继续搜索下一个：
    shift + n #搜索前一个

## 取消选中

:noh



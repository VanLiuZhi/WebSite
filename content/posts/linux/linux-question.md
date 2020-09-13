---
weight: 1000
title: "linux-question"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "linux-question"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Linux, Note]
categories: [操作系统]

lightgallery: true

toc:
  auto: false
---

Linux遇到的问题

<!-- more -->

# 遇到的问题

export LC_CTYPE=en_US.UTF-8 在 user目录下面，.bashrc 文件加入这一行，执行 `source .bashrc`

解决svn 编码错误问题。或者直接执行 LC_CTYPE=en_US.UTF-8（对本次登陆有效）

lsb_release -a  查看版本信息

本地仓库关联远程仓库：通过GitHub创建的仓库，通常会有一个README.md，在本地初始化一个目录为git  当我们想把这个目录和远程GitHub仓库关联起来的时候，实际上是合并两个分支，所以如果两个仓库有同名文件就会发生冲突，最好不要有同名文件，以确保合并分支成功。
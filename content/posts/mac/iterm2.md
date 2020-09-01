---
weight: 1
title: "Iterm2 终端工具"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Iterm2 终端工具"
resources:
- name: "base-image"
  src: "base-image.jpg"

tags: [Mac, Util]
categories: [Mac]

lightgallery: true

toc:
  auto: false
---

不错的终端软件

<!-- more -->

## 移动一个单词

在Profile -- keys下，找到如同的原配置信息。
![image](/images/Mac/iterm2_key_1.png)
其实这是已有的功能，不过Mac默认的快捷键被占用了，修改成我们习惯的。
分别修改option+←和option+→的映射，选择Action为“Send Escape Sequence”，然后输入“b”和“f”即可。

## 配置远程ssh登陆

使用脚本传参数的方式登陆，先准备脚本，内容如下：

```sh
#!/usr/bin/expect

set timeout 30
spawn ssh -p [lindex $argv 0] [lindex $argv 1]@[lindex $argv 2]
expect {
        "(yes/no)?"
        {send "yes\n";exp_continue}
        "password:"
        {send "[lindex $argv 3]\n"}
}
interact
```

在本地创建这么一个文件 `*.sh`
然后去配置iterm2，如下图，用绝对路径指向这个文件，后面加上参数。 端口，用户，IP，password。
举例：
`/Users/liuzhi/Documents/LinuxServer/liuzhiTX.sh 22 root 123.207.***.202 12345`

![image](/images/Mac/iterm2_ssh.png)

## 命令

| Command   | Description 
| :--------- | :-------
| 垂直分屏           | command + d 
| 水平分屏           | command + shift + d 
| 切换屏幕           | command + option + 方向键 command + [ 或 command + ] 
| 查看历史命令       | command + ; 
| 查看剪贴板历史     | command + shift + h 
| 新建标签           | command + t 
| 关闭标签           | command + w 
| 切换标签           | command + 数字 command + 左右方向键 
| 切换全屏           | command + enter 
| 查找               | command + f 
| 清除当前行         | ctrl + u 
| 到行首             | ctrl + a 
| 到行尾             | ctrl + e 
| 前进后退           | ctrl + f/b (相当于左右方向键) 
| 上一条命令         | ctrl + p 
| 搜索命令历史       | ctrl + r 
| 删除当前光标的字符 | ctrl + d 
| 删除光标之前的字符 | ctrl + h 
| 删除光标之前的单词 | ctrl + w 
| 删除到文本末尾     | ctrl + k 
| 交换光标处文本     | ctrl + t 
| 清屏1             | command + r 
| 清屏2             | ctrl + l 
| ⌘ + f            | 所查找的内容会被自动复制 
| ⌘ + r            | clear，而且只是换到新一屏，不会想 clear 一样创建一个空屏 
| ctrl + u         | 清空当前行，无论光标在什么位置 
| 输入开头命令后 按 ⌘ + | 会自动列出输入过的命令 
| ⌘ + shift + h     | 会列出剪切板历史 

复制：

选中即复制：iterm2有2种好用的选中即复制模式。
 
一种是用鼠标，在iterm2中，选中某个路径或者某个词汇，那么，iterm2就自动复制了。
 
另一种是无鼠标模式，command+f，弹出iterm2的查找模式，输入要查找并复制的内容的前几个字母，确认找到的是自己的内容之后，输入tab，查找窗口将自动变化内容，并将其复制。如果输入的是shift+tab（实测好像没用），则自动将查找内容的左边选中并复制。

输入command+shift+h，iterm2将自动列出剪切板的历史记录，这个历史要使用过无鼠标模式才行
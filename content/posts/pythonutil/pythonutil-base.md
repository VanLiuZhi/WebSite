---
weight: 1000
title: "python-util 工具"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "python-util 工具"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Python, Note]
categories: [Python编程]

lightgallery: true

toc:
  auto: false
---

python 偷懒小工具，通过编写脚本实现日常功能

<!-- more -->

## markdown table 语法生成

```python

"""
markdown 的table语法格式 自动生成代码，错误的地方需要手动处理一下，中文因为字符占位问题，无法做到统一格式化
"""

op_str = '''
v-bind ：动态绑定数据。简写为“:” 。=> 以后的:class="{red:boolean}"
v-on ：绑定时间监听器。简写为“@”，例：@click="xxx"；
v-text ：更新数据，会覆盖已有结构。类似{{ msg }} ；
v-show ：根据值的真假，切换元素的display属性；
v-if ：根据值的真假，切换元素会被销毁、重建； => 在dom中已消失
v-else-if ：多条件判断，为真则渲染；
v-else ：条件都不符合时渲染；
v-for ：基于源数据多次渲染元素或模块；
v-model ：在表单控件元素（input等）上创建双向数据绑定（数据源）；
v-pre ：跳过元素和子元素的编译过程；
v-once ：只渲染一次，随后数据更新也不重新渲染；
v-cloak ：隐藏未编译的Mustache语法，在css中设置[v-cloak]{display:none;}
'''

_op_str = op_str.split('\n')
_op_str.pop()
out_str = ''
max_str_len = len('v-else-if')  # pop(key[, default])
space = ' '

for item in _op_str:
    tmp = item.split(' ')
    if tmp[0] == '':
        tmp_str = "| Command{space} | Description \n".format(space=space * (max_str_len - len('Command')))
        tmp_str += "| {a} | :{b}: \n".format(a='-' * max_str_len, b='-' * (max_str_len - 2))
    else:
        tmp_str = "| {a}{space} | {b} \n".format(a=tmp[0], b=''.join(tmp[1:]), space=space * (max_str_len - len(tmp[0])))
    out_str += tmp_str

print(out_str)

# print(len('Inheritor'))

```

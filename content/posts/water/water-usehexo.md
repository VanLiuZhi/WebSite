---
weight: 1000
title: "博客迁移"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "博客迁移"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Water]
categories: [Water]

lightgallery: true

toc:
  auto: false
---

关于博客迁移，最开始是在有道上做笔记，学习记录，后来萌生了写类似博客网站的想法，于是用element-UI框架写了一个，整体的效果还不错，还有一个后台管理，使用Django作为web服务，每篇文章存储在数据库，但是想要的功能太多了，设计的功能也显得格格不入（完全按照自己的想法来），后来放弃了，转用了VuePress，后来又看到Hexo，还是迁移到Hexo吧。开始使用VuePress，也是因为作者说会在后续添加博客的功能，结果没有后续了，Hexo作为博客提供了很全面的功能

<!-- more -->

# 博客迁移

关于博客迁移，最开始是在有道上做笔记，学习记录，后来萌生了写类似博客网站的想法，于是用element-UI框架写了一个，整体的效果还不错，还有一个后台管理，使用Django作为web服务，每篇文章存储在数据库，但是想要的功能太多了，设计的功能也显得格格不入（完全按照自己的想法来），后来放弃了，转用了VuePress，后来又看到Hexo，还是迁移到Hexo吧。开始使用VuePress，也是因为作者说会在后续添加博客的功能，结果没有后续了，Hexo作为博客提供了很全面的功能

## 迁移脚本

大概扫描了一下Hexo的官方，马上起了一个项目，发现这个文件结构和VuePress差的有点多啊，不怕，写个迁移脚本就行了，这样就把原来文件结构都取出来放到通用目录下，顺便生成一下Hexo的yaml-Front-matter，对README的文件做一下重命名。

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

"""
@author:   LiuZhi
@file:     hexo_change.py
@time:     2019-10-22 23:48
@contact:  vanliuzhi@qq.com
@software: PyCharm
"""

import os

path = '/Users/liuzhi/PycharmProjects/test_code/zh/'

op_list = os.listdir(path)

write_data2 = '''
---
title: %s
date: 2018-10-22 00:00:00
updated: 2018-10-22 00:00:00
tags:
categories:
---

'''


def write_data(title):
    a = write_data2 % title
    return a


def get_absolute_path(dir_name):
    return path + dir_name


def get_dir_file(_dir, result):
    if _dir != '/Users/liuzhi/PycharmProjects/test_code/zh/.DS_Store':
        for item in os.listdir(_dir):
            res = _dir + '/' + item
            result.append(res)


def loop_file_path():
    result = []
    dir_list = map(get_absolute_path, op_list)

    for i in list(dir_list):
        get_dir_file(i, result)
    print(result)
    return result


def save_name(_dir, data):
    _list = _dir.split('/')
    name = _list[-1]
    if name == 'README.md':
        name = _list[-2] + '.md'
    with open('/Users/liuzhi/PycharmProjects/test_code/result/' + name, 'w') as f:
        f.write(data)


def get_name(_dir):
    _list = _dir.split('/')
    name = _list[-1]
    if name == 'README.md':
        name = _list[-2] + '.md'
    return name[:-3]


def hexo_change():
    """
    实现在文件头写入，不能使用a+模式，需要截断成0字节
    总体来说，要实现任意位置写入，只能是通过读取整个文件，然后将
    需要写入的插入，然后把整个结果写入
    :param file_name:
    :return:
    """
    for file_name in loop_file_path():
        with open(file_name, 'r+') as _file:
            old = _file.read()
            _file.seek(0)
            _file.write(write_data(get_name(file_name)))
            _file.write(old)

    for file_name in loop_file_path():
        with open(file_name, 'r+') as _file:
            old = _file.read()
            save_name(file_name, old)


def insert_content_to_file():
    file = open("a.txt", "r")
    file_add = open("a.txt", "r")
    content = file.read()
    content_add = file_add.read()
    pos = content.find("buildTypes")
    if pos != -1:
        content = content[:pos] + content_add + content[pos:]
        file = open("a.txt", "w")
        file.write(content)
        file.close()
        file_add.close()


def init():
    cp_cmd = 'cp -rf ~/JavaScriptProjects/my-notebook/docs/zh/ ~/PycharmProjects/test_code/zh'
    rm_zh = 'rm -rf ~/PycharmProjects/test_code/zh/'
    mark_dir = 'mkdir ~/PycharmProjects/test_code/zh'
    cmd_list = [rm_zh, mark_dir, cp_cmd]
    for i in cmd_list:
        os.system(i)
    # map(lambda x: os.system(x), cmd_list)


def cp_new_file():
    cp_cmd = 'cp -rf ~/PycharmProjects/test_code/result/ /Users/liuzhi/JavaScriptProjects/hexo_blog/source/_posts'
    os.system(cp_cmd)


if __name__ == '__main__':
    # init()
    # hexo_change()
    cp_new_file()
```

## 踩坑

浏览过一遍文档，如果不着急搞分享，评论什么的，上手会很快，开始我的图片资源也是没有问题了，直到我看了官方的`资源文件夹`功能，它提出了一个顾虑（我真的没感觉到会有这个问题，分类和主页的资源，怎么会有路径问题呢？难道是历史原因），推荐使用此功能，需要进行对应的配置，使用`{% raw %}{% asset_img example.jpg This is an example image %}{% endraw %}`，坑就坑在这么用完全没效果，然后我Google，发现别人都是装插件的，查了我的package.json，没插件啊，官方这文档不负责啊。

然而坑还没结束，装了插件仍然没用，于是我去看了GitHub对插件的用法，需要使用Markdown语法，我真是哔了狗了，Hexo的文档英文也是明确说明不要使用Markdown语法，用插件语法，这完全没用啊，看了别人的用法也是用Markdown语法。

最后，经过我的测试，使用Markdown语法，配合插件可以使用相对路径，如果这样用，你的每篇博客不能创建独立的文件夹，只能放在_posts下，不知道是BUG还是什么问题，总之一点官方文档给的用法没有效果，插件的作者都没这么用，个人猜测框架扩展了插件后，使用插件语法应该是没问题的，Hexo支持这么多插件，然而在这个插件上出了问题估计是BUG了。

解决方案：使用Markdown语法，图片放在`_posts/images`下，由于这是绝对路径，部署和本地都是正常的，另外不一定要把文章放在_posts下，可以再创建一个父级目录，虽然编译后文章被分类了，但是原始项目这么搞就太混乱了，这让我觉得使用命令创建文章有点鸡肋了。

## 常用命令补充

1. `hexo generate` 生成静态文件，不会去执行部署

2. 监视文件变动
Hexo 能够监视文件变动并立即重新生成静态文件，在生成时会比对文件的 SHA1 checksum，只有变动的文件才会写入。

`hexo generate --watch`

3. `hexo server`
参数：
-p, --port	重设端口
-s, --static	只使用静态文件
-l, --log	启动日记记录，使用覆盖记录格式

4. hexo --debug 调试模式，会记录日志

## Nginx

开始还在研究用什么应用服务器，最好使用hexo自带的，然后反向代理过去，结果502了，也不说是什么问题，后来发现直接用静态文件处理就行了，
不用这么麻烦。

```
location / {
    # 将blog目录下的全路径public复制进来
       root   /root/blog/public; 
        }
     }
```

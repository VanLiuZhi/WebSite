---
title: "Usehugo"
date: 2020-09-02T11:41:48+08:00
draft: true
---

## 文件结构

archetypes: 储存.md的模板文件，类似于hexo的scaffolds，该文件夹的优先级高于主题下的/archetypes文件夹
config.toml: 配置文件
content: 储存网站的所有内容，类似于hexo的source
data: 储存数据文件供模板调用，类似于hexo的source/_data
layouts: 储存.html模板，该文件夹的优先级高于主题下的/layouts文件夹
static: 储存图片,css,js等静态文件，该目录下的文件会直接拷贝到/public，该文件夹的优先级高于主题下的/static文件夹
themes: 储存主题
public: 执行hugo命令后，储存生成的静态文件

resources 目录
资源缓冲目录，非默认创建，用于加速 Hugo 的生成过程，也可以用给模板作者分发构建好的 SASS 文件，因此不必安装预处理器。
assets 目录
不是默认创建的资源目录，保存所有需要通过 Hugo Pipes 处理的资源，只有那些 .Permalink 和 .RelPermalink 引用的文件会发布到 public 目录中，参考 Hugo 管道处理。

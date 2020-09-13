---
weight: 1000
title: "利用Docker快速构建开发环境"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "利用Docker快速构建开发环境"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Docker, Linux, Technology]
categories: [Technology技术]

lightgallery: true

toc:
  auto: false
---

Docker是很不错的容器技术，利用Docker可用快速构建一个开发环境，这样的好处在于一台新的电脑，只要安装了Docker软件，搭建环境就是几个命令的事，这样整个开发团队都会在同样的环境下进行，而且部署的时候，运维的同学只需要针对安全性做一些调整即可上线。

<!-- more -->

## 序言

以Python语言为例，一个小团队的技术栈差不多会用到以下的东西：

1. Python环境，包括各种需要的包
2. 数据库，包括MySQL、MongoDB等
3. 缓存服务，使用Redis等
4. 任务队列，使用celery，RabbitMQ
5. Http服务器，Nginx
6. WSGI服务器，gunicorn，uwsgi

## 基于Dockerfile构建Python环境

对于不需要定制的服务，直接使用官方的镜像即可，而需要定制的列如Python，利用Dockerfile来构建，主要是安装运行环境，已经常用的软件，因为该运行环境可能后期还需要做调整。

```Dockerfile
FROM python:3.7
LABEL author="liuzhi<1441765847.com>"

# 换源，Python镜像基于Debian，使用阿里的Debian源
RUN rm /etc/apt/sources.list
COPY sources.list /etc/apt/sources.list

# 运行命令，安装常用软件
RUN apt-get update \
    # 修改时区
    && ln -sf /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime \
    # && apt-get install -y apt-utils \
    # && apt-get install -y wget \
    && apt-get install -y zsh \
    && chsh -s /bin/zsh root \
    && apt-get install -y curl \
    && apt-get install -y git \
    && apt-get install -y vim 

# 安装zsh的扩展
RUN sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"  || true

ENV FLASK_ENV dev
ENV APP_DIR /code
WORKDIR /code/
EXPOSE 5000
```

注意事项：

- 记得在同级目录下装备源文件 `sources.list`

- 有时候会提示需要安装 `apt-utils`，不过我这里是基于Debian的，这东西还装不上，没有apt-utils，安装不了第三方包，wget安装失败了，可用进入容器自行安装wget

- RUN命令中通过 `&&` 连接命令，因为在Docker中，每一个指令都会构建一层，因此尽量将命令都放在一个RUN指令中，用 && 来串联。还有命令后面的 `\` 符最后就不需要写了，不然和下面的命令连起来了，如果你使用Dockerfile静态语法检查工具，有错误提示的（xcode安装插件即可编写Dockerfile，错误的地方会有提示）

- 这里还安装了zsh的扩展，使用了 `||` ，不用直接安装镜像创建会失败，我猜测可能是这个命令后面没接上，不用 `||` 可以看到安装信息是成功了的，但是容器创建会失败，太具体的情况不知道了，安装了zsh，启动容器的时候记得通过 `/bin/zsh` 进入

这里使用了官方的Python镜像，体积有点大，好处是装软件一般不会出问题了，作为开发用就不在精简体积上花时间了。

## 使用Docker-composes

安装Docker-composes，不推荐使用Python来安装，可能因为Python版本或包依赖问题导致各种问题。

{% blockquote 使用GitHub源 %}
sudo curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-uname -s -u name -m -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
{% endblockquote %}



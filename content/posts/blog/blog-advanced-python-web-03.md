---
weight: 5503
title: "python web 开发(三) 使用Docker部署项目"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "python web 开发(三) 使用Docker部署项目"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Python, Web, Technology]
categories: [Python]

lightgallery: true

toc:
  auto: false
---

docker 是 2013 年发布的开源的应用容器引擎，虽然时间不长，但是很快占据市场受到开发者的青睐，docker不仅是运维同学的专项，作为一名开发者，学习docker是必备的。下面讲解如何使用docker开发和部署项目。

<!-- more -->

## 概述

docker使用的优势：

1. 利用docker你可以快速搭建环境，比如在本项目中用到的Elasticsearch是基于Java的，如果常规安装，还需要准备Java环境，安装Java虚拟机等，这对于一名Python开发者来说显然耗费大量时间，但是有了docker就不一样了，只需要安装docker软件，使用Elasticsearch的镜像就可以快速搭建服务，把精力放在如何使用Elasticsearch中去。我比较推崇的一种开发模式就是本地跑Python环境，然后把各种相关服务跑在docker中，这样很容易扩展，又不影响本地的开发，也不会因为个人电脑的关系导致相关服务难以安装。

2. 统一了环境，以前部署项目都是比较头疼的，一台新的服务器，由于安装Linux发行版不一样，在安装软件的时候经常因为缺少各种依赖浪费很多时间，以前我在阿里云上部署项目，新的服务器每次都安装有差异，我觉得把时间花在安装软件上真的是不值的。使用docker的话，本地和线上可以统一环境，只要本地能正常运行，在线上只需要调整一些安全相关的配置即可上线。

本项目除了Python环境外，其余在开发测试阶段都是依赖于docker的，像Redis，RabbitMQ等服务都是跑在docker中，部署的话只需要构建一个当前Python环境配合已有的服务即可。

## docker 基本

docker 是容器技术，初学者很容易和虚拟机的概念混用，包括在使用的时候也是如此。这一点非常的重要，我在初用docker的时候，把软件都装在一个容器里面，也是看别人的博客入门给我这种感觉，其实不应该这样，这种做法更像一个虚拟机，可以用来做开发，比如打包一个完整项目的开发环境，大家用这个镜像可以马上运行项目。正确的用法应该是每个服务一个容器，彼此分类开来。

docker的概念围绕着容器container和镜像image展开，镜像是固化的文件，容器是镜像的实例，更多的可以查看我以前做的总结 [Docker 容器在 web 开发中的运用](http://www.liuzhidream.com/2018/10/22/Docker/Docker/)

### 数据卷

数据卷是为了做数据持久化，在容器内长生的数据随着容器删除就没了，这对应像mysql这样的需要持久化数据的服务当然是不允许的，所以有了数据卷的概念，把容器的一个目录和本机的一个目录映射，本机产生的数据容器可以使用，反之亦然。

### 网络与端口

能映射数据也能映射端口，映射端口的意思就是本机的端口和容器是互通的，访问本机的端口就是访问容器的端口。

网络是比较重要的概念，不了解网络的机制让容器之间通信变得头疼，很容易卡在这里，可以使用docker来创建网络，处在同一网络中的容器可以互相通信，这个时候其实容器处在一个局域网中，只要知道ip也能通信，但是通常不这么做。

### Dockerfile

Dockerfile是构建镜像的文件，通过编写Dockerfile可以创建自定义的镜像。一般镜像可以从网络下载，或者使用别人创建的，自己想要制作镜像就要使用Dockerfile，因为你可能需要很多定制，比如构建一个Python环境，并且安装依赖，安装像vim，wget等常用的软件，因为官方镜像都是精简的，默认只有Python环境。

### docker-compose

docker-compose是容器编排工具，用来组织多个容器如何运行的，是必备的工具，本项目就是使用docker-compose一次运行所有服务，不再像使用 `docker run` 这样的形式，一次跑一个容器。除了运行方便，还提供了很多配置，可以把各种参数都一次写入docker-compose.yaml文件，相比原生方式更加灵活，管理更方便。

本项目都是使用docker-compose，这个工具需要单独安装，推荐使用二进制安装的形式。

## 开发中使用docker

在开发中可以使用docker来运行除Python环境本身的服务，其实你把Python环境用docker来运行也是可以的，这样把项目代码提供数据卷映射到容器，修改本地代码，使用容器来运行它，不过我要配合IDE来开发，就不把Python环境跑在docker中了。

docker-compose.yaml 文件

```yaml
version: '3'
services:

  redis:
    image: redis:latest
    container_name: redis-web
    ports:
      - 6379:6379
    volumes:
      - ~/redis/data:/data

  rabbitmq:
    image: rabbitmq:latest
    container_name: rabbitmq-web
    hostname: rabbit
    environment:
      - RABBITMQ_DEFAULT_USER=developer
      - RABBITMQ_DEFAULT_PASS=dev1234
    ports:
      - 5672:5672
      - 15672:15672

  elasticsearch:
    image: elasticsearch:5-alpine
    restart: always
    container_name: elasticsearch-web
    ports:
      - 9200:9200
      - 9300:9300
    environment:
      - discovery.type=single-node
      - bootstrap.memory_lock=true
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms256m -Xmx256m -XX:+AssumeMP"
```

使用官方提供的镜像，这里有个套路，就是你不知道某个镜像要怎么用的时候，比如像elasticsearch你不知道它要映射什么端口，Redis数据目录是在哪个路径，可以去Docker官方查找，对应镜像都有这些信息。

使用命令 `docker-compose up -d` 就可以运行容器了，可以在本机访问9200测试elasticsearch是否已经提供正常服务。

## 部署中使用docker

部署的时候，需要构建python环境的镜像，就要使用到Dockerfile。

```yaml
FROM python:3.6
LABEL author="liuzhi<1441765847.com>"

# 换源，Python镜像基于Debian，使用阿里的Debian源
RUN rm /etc/apt/sources.list
COPY sources.list /etc/apt/sources.list

# 运行命令，安装常用软件
RUN apt-get update \
    && ln -sf /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime \
    # 有时候会提示需要安装apt-utils，不过我这里是基于Debian的
    # 这东西还装不上，没有apt-utils，安装不了第三方包，wget安装失败了
    # 进入容器安装wget
    # && apt-get install -y apt-utils \
    # && apt-get install -y wget \
    && apt-get install -y zsh \
    && chsh -s /bin/zsh root \
    && apt-get install -y curl \
    && apt-get install -y git \
    && apt-get install -y vim 
# 安装zsh的扩展，使用了 || ，不用直接安装镜像创建会失败
RUN sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"  || true

# 把当前目录文件复制到容器的/code目录，主要是为了复制requirements.txt
ADD . /docker_file
WORKDIR docker_file
RUN pip install -r requirements.txt
#CMD ["python", "app.py"]
```

主要就是组织命令，这里安装了zsh工具，并修改默认shell为zsh，安装了curl，git，vim软件。

需要注意的是镜像源的选择，这里是基于已有的Python镜像来制作自己的镜像的，使用的是python3.6，官方有很多镜像会带alpine后缀，这是说明该镜像是基于Linux-alpine版本的，就是Linux精简版体积很小，所以砍了很多东西，推荐代码环境使用标准版，其它服务使用alpine版本。

docker-compose.yaml 文件

```yaml
version: '3'
services:

  app:
    image: python_flask:v1
    container_name: python-web
    volumes:
     - ~/PycharmProjects/flask_starlight:/code
    command: cd /code/flask_starlight
    command: python auto_app.py
    depends_on:
     - mysql
     - redis
     - rabbitmq
     - elasticsearch
    expose:
      - 5000
    networks:
      net_front:
        aliases:
          - python_server
      net_backend:
        aliases:
          - python_backend

  mysql:
    image: mysql:latest
    container_name: mysql-web
    volumes:
      - ~/dev-mysql:/var/lib/mysql
    networks:
      net_backend:
        aliases:
          - masterdb
    environment:
      MYSQL_ROOT_PASSWORD: root1234
    expose:
      - 3306

  redis:
    image: redis:latest
    container_name: redis-web
    expose:
      - 6379
    volumes:
      - ~/redis/data:/data
    networks:
      net_backend:
        aliases:
          - master_redis

  rabbitmq:
    image: rabbitmq:latest
    container_name: rabbitmq-web
    hostname: rabbit
    environment:
      - RABBITMQ_DEFAULT_USER=developer
      - RABBITMQ_DEFAULT_PASS=dev1234
    expose:
      - 5672
      - 15672
    networks:
      net_backend:
        aliases:
          - rabbitmq_server

  elasticsearch:
    image: elasticsearch:5-alpine
    restart: always
    container_name: elasticsearch-web
    expose:
      - 9200
      - 9300
    environment:
      - discovery.type=single-node
      - bootstrap.memory_lock=true
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms256m -Xmx256m -XX:+AssumeMP"
    networks:
      net_backend:
        aliases:
          - elasticsearch_server

  nginx:
    image: nginx
    container_name: nginx-web
    volumes:
      - ~/PycharmProjects/flask_starlight/compose_docker_dev/nginx:/etc/nginx
      - ~/PycharmProjects:/web
    ports:
      - 80:80
    environment:
      - NGINX_PORT=80
    depends_on:
      - app
    networks:
      - net_front

networks:
  net_front:
  net_backend:
```

这里的配置就是把所有服务都跑在docker里面，Python的环境先用Dockerfile构建好后，直接指定镜像。

部署相对于开发使用会比较复杂，开发只是把docker容器当成一个服务，提供正确的端口即可，部署需要组织各个服务之间的通信。

这里就要用到网络了，使用networks创建网络，这里创建了两个网络：

```
属于net_front网络的容器: nginx，app
属于net_backend网络的容器: app，mysql，redis，rabbitmq，elasticsearch
```

其实可以不使用network，因为docker-compose创建的容器会默认加入到一个以配置文件目录名生成的网络中，也就是它们默认已经处在一个局域网中了，这里使用network是为了让服务之间的关系清晰，通过自定义的网络分割容器，像nginx服务就不能访问mysql。

还有一点很重要的就是如何在一个容器中访问另外的容器，比如在nginx中，需要使用反向代理到Python容器，配置文件如下：

```
user nginx;          # nginx运行的用户及用户组
worker_processes 1; # worker进程个数

error_log /var/log/nginx/error.log warn; # 错误日志目录
pid       /var/run/nginx.pid;            # nginx启动后进程pid写入这个文件

events {
    worker_connections 65536; # 每个worker进行的最大连接数
}

http {

    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';
    
    access_log  /var/log/nginx/access.log  main; # 访问日志

    upstream frontends {
        server python_server:5000;
    }

    include /etc/nginx/conf.d/*.conf; # 导入其它配置

}
```

可以看到，nginx的反向代理是通过 `python_server:5000` 来访问的，而这个python_server是Python容器网络的别名，或者通过服务名称app也可以，即 `app:5000`。同样的，在Python容器中要访问mysql，flask的配置应该是这样的 `SQLALCHEMY_DATABASE_URI = 'mysql+pymysql://root:root1234@masterdb/starlight?charset=utf8mb4'` 通过对MySQL容器的网络别名masterdb来访问。

## 常用技巧

`docker-compose exec server_name` 通过docker-compose exec 加服务名称，可以和容器进行交互，比如 `docker-compose exec redis cat /etc/hosts` 查看redis服务的hosts文件。

连接容器：

`docker-compose exec app /bin/zsh`

查看日志：

`docker-compose exec logs app`
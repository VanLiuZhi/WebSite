---
weight: 1000
title: "Nginx 学习笔记"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Nginx 学习笔记"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Linux, Web, Note]
categories: [Web]

lightgallery: true

toc:
  auto: false
---

Nginx (engine x) 是一个高性能的HTTP和反向代理服务，也是一个IMAP/POP3/SMTP服务，静态文件应该交由 Web 服务器来处理，动态内容由应用服务器来处理。另外编译安装可以定制很多东西，有些包管理器安装的版本不能支持HTTPS，需要编译安装才行。关于HTTPS服务，由于自己创建的证书不被认可，浏览器会识别有风险，这就比较尴尬了，可以使用免费的SSL证书服务 [申请Let's Encrypt永久免费SSL证书](https://www.jianshu.com/p/3ae2f024c291)

<!-- more -->

## 基本命令

- nginx -s reload  ：修改配置后重新加载生效
- nginx -s reopen  ：重新打开日志文件
- nginx -c 配置文件路径 ：使用指定配置文件运行Nginx
- nginx -t 测试nginx配置文件是否正确

输出如下，可以知道配置文件所在位置
{% blockquote %}
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
{% endblockquote %}

- nginx -s stop  ：快速停止nginx
- nginx -s quit  ：完整有序的停止nginx

## 配置

{% blockquote %}
记得配置后面需要 `;` 结尾，修改配置后通过 nginx -t 测试配置正确性再启动Nginx
{% endblockquote %}

nginx的配置, 一般就是配server模块，该模块的全局定义，location 定义了正则的解析（向服务器请求各种资源，nginx应该如何处理），然后配合各种指令，理解各种指令很重要。默认的nginx配置会使用模块的方式，即一个基本配置文件，加上导入的其它配置文件。

```
...   #全局块

events {  #events块
   ...
}

http      #http块
{
    ...   #http全局块

    server        #server块
    { 
        ...       #server全局块
        location [PATTERN]   #location块
        {
            ...
        }
        location [PATTERN] 
        {
            ...
        }
    }
    server
    {
      ...
    }
    ...     #http全局块
}

```
每一个server块就是一个虚拟主机，如果使用官方的模块化的配置，那么就是主配置文件定义全局的，通过include导入各个server，整体的结果就是通过增加server，比如有站点A，站点B，那么配置a.conf，b.conf，它们都导入到主配置文件中，整体就连接起来了。

## mime.types

在官方的配置文件中，导入了这个文件，跟进去查看，其实就是对文件类型的定义，配置文件内容如下：
```
include       /etc/nginx/mime.types;
default_type  application/octet-stream;
```
这里还定义了默认类型，如果不写，默认为text/plain，web开发中，对于请求的资源都是有定义的，资源类型的定义决定了浏览器应该如果处理它

### location 配置

location 后面通过正则来匹配路由，将匹配到的请求交由该location来处理。

**1. root**：root指令指明了被匹配的路由要去查询的地址，以官方默认配置为例
```
location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
```
当路由是 / 的时候，搜索路径为 `/usr/share/nginx/html`，这里的html其实是个文件夹，就是去html下查找，文件夹内容为

```
root@8c67bbe80ff5:/usr/share/nginx/html# ls -l
total 8
-rw-r--r-- 1 root root 494 Dec 25 09:56 50x.html
-rw-r--r-- 1 root root 612 Dec 25 09:56 index.html
```
如果不是 `/` 而是 `/img`，搜索路径为 `/usr/share/nginx/html/img`

**2. alias**：是用来取别名，假设设置的匹配为 `/img`，`alias /usr/share/nginx/html`，url为 `http://www.hello.com/img/a.jpg` 搜索路径为`/usr/share/nginx/html/a.jpg`，可以看到没像root一样在html后面加路径，就是因为alias是取别名，访问 `/img` 变成访问取的别名。

## 关于监听端口

这也是一个容易被忽视的概念，但也是web开发很基础的概念。

首先要知道，通过域名解析，只能到ip，默认的端口是80，这是由http服务默认的，因为HTTP默认端口是80，HTTPS默认端口是443。

假设域名是 www.liuzhidream.com 解析到ip 123.123.1.1 由浏览器发起的http请求 www.liuzhidream.com 实际是 www.liuzhidream.com:80，对应ip就是 123.123.1.1:80。

所以要在nginx上监听80端口，如果你想要监听其它端口，那么通过域名访问就要显示的指定端口了，或者通过其它方法：

1. CDN 
2. url转发

总之80是默认的HTTP端口，不想使用这个端口就要手动处理请求。

## 配置文件参考

主配置文件

```
user nginx;          # nginx运行的用户及用户组
worker_processes 1; # worker进程个数

error_log /var/log/nginx/error.log warn; # 错误日志目录
pid       /var/run/nginx.pid;            # nginx启动后进程pid写入这个文件

events {
    worker_connections 65536; # 每个worker进行的最大连接数
}

http {

    include       /etc/nginx/mime.types; # 导入解析文件类型
    default_type  application/octet-stream;

    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"'; # 日志格式
    
    access_log  /var/log/nginx/access.log  main; # 访问日志

    upstream frontends { 
        server app:5000; # 反向代理配置
    }

    include /etc/nginx/conf.d/*.conf; # 导入其它配置

}
```

server 部分

{% codeblock web-python.conf %}
server { 

	listen 80; # 监听端口
	server_name test.liuzhi.com;
	keepalive_timeout 5; # keepalive超时时间，默认是75秒

	location ^~ /static/ { # /static/ 这种路径通常放了静态文件，由nginx直接serve
		root /web/flask_starlight; # 静态文件存储的位置
	}

	location ~* \.(woff|eot|ttf|svg|mp4|webm|jpg|jpeg|png|gif|ico|css|js)$ {
		expires 30d; # 静态文件由于变化很小，超时时间设置的很长
	}

	location / {
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Scheme $scheme;
		proxy_redirect off;
		proxy_pass http://frontends;
	}

}
{% endcodeblock %}

{% codeblock blog.conf %}
server {
    listen       80;
    server_name  www.liuzhidream.com;

    location / {
        root   /root/blog/public;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
{% endcodeblock %}
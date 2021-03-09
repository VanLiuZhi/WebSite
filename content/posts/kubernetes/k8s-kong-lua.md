---
weight: 1102
title: "K8s Kong Lua"
date: 2020-08-16T14:00:00+08:00
lastmod: 2020-08-16T14:00:00+08:00
draft: false
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "K8s Kong Lua"
# featuredImagePreview: "images/Kubernetes/install.png"
# featuredImage: "/images/Kubernetes/install.png"
resources:
- name: "base-image"
  src: "/images/base-image.jpg"

tags: [Linux, Docker, Note]
categories: [Kubernetes] 

lightgallery: true

toc:
  auto: false
---




<!--more-->

## lua，kong，nginx，OpenResty 之间的关系

nginx是我们常用的一个网关服务，但是nginx可定制化不高，配置文件的配置也不方便，于是便有了OpenResty，它通过lua脚本的扩展了nginx的功能，利用lua脚本语言编程的优势可以实现更多功能。
lua可以看成是对C++的一种封装，它是很轻量的，一些业务逻辑等使用C++这样的中层语言开发难度比较大，而lua这种高级语言很适合做这种事情

而Kong是在OpenResty的基础上扩展的，提供了更加丰富的功能，适合作为云原生的网关应用

## 如何修改kong的lua源码

我们有些需求无法满足的时候，可以通过修改lua源码来实现。重新打成镜像即可

如果从源码构建，官方的方式比较复杂。官方的Dockerfile是从rpm包构建的。也就是我们需要先传rpm

涉及到的项目
```
Kong/docker-kong        镜像构建
Kong/kong-build-tools   源码编译工具
```

如果只是改lua脚本不需要这么复杂的方式，可以从官方镜像继承，然后把修改后的lua脚本替换，这种方式就非常容易了

```
lua不需要编译，直接由解释器执行即可。python会先翻译成中间文件，也就是字节码，之后再由解释器执行
```

dockerfile构建，handler.lua就是我们修改过的源码

```dockerfile
FROM hub.eos.h3c.com/kong/kong:2.2

LABEL maintainer="lys3415"

COPY ./handler.lua /usr/local/share/lua/5.1/kong/plugins/cors/
```

修改部分，kong2.2版本

```lua
function CreateUUID()
  local template ="xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
  d = io.open("/dev/urandom", "r"):read(4)
  math.randomseed(os.time() + d:byte(1) + (d:byte(2) * 256) + (d:byte(3) * 65536) + (d:byte(4) * 4294967296))
  return string.gsub(template, "x", function (c)
        local v = (c == "x") and math.random(0, 0xf) or math.random(8, 0xb)
        return string.format("%x", v)
        end)
end

function CorsHandler:access(conf)
  if kong.request.get_method() ~= "OPTIONS"
     or not kong.request.get_header("Origin")
     or not kong.request.get_header("Access-Control-Request-Method")
  then
    kong.service.request.set_header("trace",CreateUUID())
    local konghost = os.getenv("HOSTNAME")
    if konghost then
      kong.response.set_header("Test", "true")
    else
      kong.response.set_header("Test", "false")
    end
    kong.response.set_header("Kong-Host", konghost)
    return
  end
... 省略
```

## 发现的问题

我们在Cors插件中，注入一个trace给请求头，带给下一个服务，并且把HOSTNAME注入到响应头中

一切准备就绪，重新编译源码并开启Cors插件，结果发现请求调用报错了，查看日志原因在`os.getenv("HOSTNAME")`，获取到的是nil，而我们把nil设置到请求头当然不行

`os.getenv` 的解释如下：

```sh
os.getenv()
原型：os.getenv (varname)
解释：返回当前进程的环境变量varname的值,若变量没有定义时返回nil
```

这就很奇怪了，为什么会获取不到环境变量呢。我们执行命令查看一下环境变量 `kubectl -n kong2 exec -it ingress-kong2-69d7985b5b-sp9mz -c proxy env | grep HOSTNAME`
发现HOSTNAME是有的

带着疑问查找了一些文档，终于找到了原因

```
在项目中有时会遇到使用系统环境变量的问题，但是直接使用 os.getenv() 是不可行的，不仅是在 Kong 中，在 OpenResty 也都是不可以的，原因是 Kong 是基于 OpenResty ，OpenResty 是基于 Nginx 的，而Nginx在启动的时候，会把环境中所有的环境变量都清除掉，我们可以从Nginx的官方文档中看到这段描述：http://nginx.org/en/docs/ngx_core_module.html#env
```

查看了官方文档，应该是出于安全的考虑

```
By default, nginx removes all environment variables inherited from its parent process except the TZ variable. This directive allows preserving some of the inherited variables, changing their values, or creating new environment variables. These variables are then:

inherited during a live upgrade of an executable file;
used by the ngx_http_perl_module module;
used by worker processes. One should bear in mind that controlling system libraries in this way is not always possible as it is common for libraries to check variables only during initialization, well before they can be set using this directive. An exception from this is an above mentioned live upgrade of an executable file.
```

## 解决环境变量的问题

依照文档的说法，我们要把会用到的环境变量在nginx.conf中声明，然后重启一下Nginx，就可以在 Lua 代码中使用 os.getenv("USERNAME") 获取环境变量

```
events{
  ...
}
env PATH;
env USERNAME;
```

没办法，进入kong的proxy容器，找到配置文件

```
bash-5.0# find / -name nginx.conf
/tmp/resty_QDVFcwYsXg/conf/nginx.conf
/tmp/resty_fRHoKlguip/conf/nginx.conf
/usr/local/kong/nginx.conf
/usr/local/openresty/nginx/conf/nginx.conf
```

文件很多，镜像在启动的时候肯定会加载配置文件的，查看Dockerfile文件，应该是通过docker-entrypoint.sh脚本启动的

```Dockerfile
ENTRYPOINT ["/docker-entrypoint.sh"]

EXPOSE 8000 8443 8001 8444 $EE_PORTS

STOPSIGNAL SIGQUIT

CMD ["kong", "docker-start"]
```

在脚本中，果然找到了启动命令相关的内容，通过`-p`指定目录前缀，`-c`指定配置文件。`-p`的路径是`/usr/local/kong`
可以确定配置文件是`/usr/local/kong/nginx.conf`

```sh
if [[ "$1" == "kong" ]]; then
  PREFIX=${KONG_PREFIX:=/usr/local/kong}
  file_env KONG_PG_PASSWORD
  file_env KONG_PG_USER
  file_env KONG_PG_DATABASE

  if [[ "$2" == "docker-start" ]]; then
    kong prepare -p "$PREFIX" "$@"

    ln -sf /dev/stdout $PREFIX/logs/access.log
    ln -sf /dev/stdout $PREFIX/logs/admin_access.log
    ln -sf /dev/stderr $PREFIX/logs/error.log

    exec /usr/local/openresty/nginx/sbin/nginx \
      -p "$PREFIX" \
      -c nginx.conf
  fi
fi
```

再次编译新的Docker镜像，开启插件，测试。发现问题并没有得到解决，仍然获取不到环境变量

原来在 Kong 里面，我们看到Kong生成的 Nginx配置文件是不可修改的，即使你修改了，Kong 还是会生成恢复成修改前，可以查看这个 issues https://github.com/Kong/kong/issues/3473

官方文档也有详细的说明

```
Custom Nginx templates
Kong can be started, reloaded and restarted with an --nginx-conf argument, which must specify an Nginx configuration template. Such a template uses the Penlight templating engine, which is compiled using the given Kong configuration, before being dumped in your Kong prefix directory, moments before starting Nginx.

The default template can be found at: https://github.com/kong/kong/tree/master/kong/templates. It is split in two Nginx configuration files: nginx.lua and nginx_kong.lua. The former is minimalistic and includes the latter, which contains everything Kong requires to run. When kong start runs, right before starting Nginx, it copies these two files into the prefix directory, which looks like so:
```

通过查阅官方文档，我们终于明白了，这个nginx.conf是通过模板渲染生成的，使用了`Penlight templating engine`这个模板引擎

- 解决方案1

我们可以创建一个custom-nginx.conf文件，把对env的声明写到里面，然后在启动命令处做修改，加上`--nginx-conf custom-nginx.conf`
不过这种方式还要改`docker-entrypoint.sh`

- 解决方案2

既然nginx.conf是通过模板渲染出来的，我们直接改这个模板就行了，通过find命令找到模板 `/usr/local/share/lua/5.1/kong/templates/nginx.lua`

加上env的声明

```lua
return [[
pid pids/nginx.pid;
error_log ${{PROXY_ERROR_LOG}} ${{LOG_LEVEL}};

# injected nginx_main_* directives
> for _, el in ipairs(nginx_main_directives) do
$(el.name) $(el.value);
> end

events {
    # injected nginx_events_* directives
> for _, el in ipairs(nginx_events_directives) do
    $(el.name) $(el.value);
> end
}

# 环境变量声明
env HOSTNAME;
env MY_POD_NAME;

> if role == "control_plane" or #proxy_listeners > 0 or #admin_listeners > 0 or #status_listeners > 0 then
http {
    include 'nginx-kong.conf';
}
> end

> if #stream_listeners > 0 then
stream {
    include 'nginx-kong-stream.conf';
}
> end
]]
```

再次编译，测试

```s
85.0.4183.83 Safari/537.36"
2020/12/31 03:55:09 [notice] 21#0: *1573 [lua] handler.lua:208: ===================== konghost:, client: 10.90.16.231, server: kong, request: "GET /eos/healthy HTTP/1.1", host: "myapp.h3c.com"
2020/12/31 03:55:09 [notice] 21#0: *1573 [lua] handler.lua:209: k8s-05, client: 10.90.16.231, server: kong, request: "GET /eos/healthy HTTP/1.1", host: "myapp.h3c.com"
2020/12/31 03:55:09 [notice] 21#0: *1573 [lua] handler.lua:210: ingress-kong2-549ffbfdc6-phs8l, client: 10.90.16.231, server: kong, request: "GET /eos/healthy HTTP/1.1", host: "myapp.h3c.com"
```

HOSTNAME终于可以读取到了

## 关于配置文件

既然Kong会重置配置文件，那一般需求是不是都要改模板？

显然是不需要的，修改模板只是一个另类的思路，官方提供了解决方案

有一个kong.conf的文件，比如下面这段是默认的配置文件内容，通过修改它，会自动注入修改的配置到nginx.conf中，我们只用改这个配置文件就行了，比如nginx service段的配置，修改对应kong.conf的配置会注入到最终的nginx.conf中

```s
#nginx_events_worker_connections = auto
                                 # Sets the maximum number of simultaneous
                                 # connections that can be opened by a worker process.
                                 #
                                 # The special and default value of `auto` sets this
                                 # value to `ulimit -n` with the upper bound limited to
                                 # 16384 as a measure to protect against excess memory use.
                                 #
                                 # See http://nginx.org/en/docs/ngx_core_module.html#worker_connections
```

如果上述配置不满足需求，就用方案1的做法，定义一个自己的conf，通过`--nginx-conf custom-nginx.conf`生效

更多参考官方配置文件:

https://docs.konghq.com/2.2.x/configuration/

## 参考

https://docs.konghq.com/2.2.x/configuration/

https://102no.com/2019/09/02/kong-getenv/

http://nginx.org/en/docs/ngx_core_module.html#env

https://github.com/openresty/lua-nginx-module/issues/601

---
weight: 1
title: "Web Performance Test"
subtitle: "Web Performance Test"
date: 2021-01-26T17:14:26+08:00
lastmod: 2021-01-26T17:14:26+08:00
draft: true
author: "VanLiuZhi"
authorLink: "https://www.liuzhidream.com"
description: "Web Performance Test"
# featuredImagePreview: "images/base-image.jpg"
# featuredImage: "/images/base-image.jpg"
# resources:
# - name: "featured-image"
#   src: "images/base-image.jpg"

tags: 
categories: 

lightgallery: true

toc:
  # 自动展开目录
  auto: false
---



<!--more-->

## TIME_WAIT 问题(端口优化)

“Cannot assign requested address.”是由于linux分配的客户端连接端口用尽，无法建立socket连接所致，
虽然socket正常关闭，但是端口不是立即释放，而是处于TIME_WAIT状态，默认等待60s后才释放
是客户端的问题不是服务器端的问题。通过netstat，的确看到很多TIME_WAIT状态的连接

client端频繁建立连接，而端口释放较慢，导致建立新连接时无可用端口

- 修改端口打开配置

```s
1. 调低端口释放后的等待时间，默认为60s，修改为15~30s
sysctl -w net.ipv4.tcp_fin_timeout=30
2. 开启对于TCP时间戳的支持,若该项设置为0，则下面一项设置不起作用
sysctl -w net.ipv4.tcp_timestamps=1
3. 表示开启TCP连接中TIME-WAIT sockets的快速回收
sysctl -w net.ipv4.tcp_tw_recycle=1
```

net.ipv4.tcp_tw_recycle = 1 这个功能打开后，确实能减少TIME-WAIT状态，但是打开这个参数后，会导致大量的TCP连接建立错误，从而引起网站访问故障


- 增加可用端口

```s
$ sysctl -a |grep port_range
net.ipv4.ip_local_port_range = 50000    65000      -----意味着50000~65000端口可用

修改参数：
$ vi /etc/sysctl.conf
net.ipv4.ip_local_port_range = 10000    65000      -----意味着10000~65000端口可用

改完后，执行命令“sysctl -p”使参数生效，不需要reboot

配置保留端口，防止端口占用

`net.ipv4.ip_local_port_range =1024 65535`
`net.ipv4.ip_local_reserved_ports =8080,8081,9000-9010`
```

windows也会报这个问题，大致描述是：An operation on a socket could not be performed because the system lacked sufficient buffer space or because a queue was full
也是去修改配置，只是改的方式不一样，建议压测机用Linux

## 内核优化

vim /etc/sysctl.conf

主要是和tcp相关的优化，这部分的优化是比较复杂的，通常情况下，修改端口范围满足压测机，修改连接数满足服务端即可开始压测

内核的优化是比较复杂的配置

```
我们可以直接修改内核参数，它会作用于内存，并且每次重新启动的时候，都去/etc/sysctl.conf目录读取配置
所以我们可以通过命令 sysctl -w 直接修改，也可以 修改 /proc 目录下的虚拟文件映射，但是这都是临时修改，我们可以改文件，或者

# sysctl settings are defined through files in
# /usr/lib/sysctl.d/, /run/sysctl.d/, and /etc/sysctl.d/

按照提示去改对于的文件

假如我们想重置内核配置，只要制空配置文件，然后重启就行(重启要谨慎)，因为一些写到内核的配置不会因为重新加载 /etc/sysctl.conf 被覆盖，需要显示的覆盖才行
```

1. timewait的数量，默认是180000。(Deven:因此如果想把timewait降下了就要把tcp_max_tw_buckets值减小)
net.ipv4.tcp_max_tw_buckets = 6000

2. 允许系统打开的端口范围。
net.ipv4.ip_local_port_range = 1024 65000

3. 启用TIME-WAIT状态sockets快速回收功能;用于快速减少在TIME-WAIT状态TCP连接数。1表示启用;0表示关闭。但是要特别留意的是：这个选项一般不推荐启用，因为在NAT(Network Address Translation)网络下，会导致大量的TCP连接建立错误，从而引起网站访问故障。
net.ipv4.tcp_tw_recycle = 0

实际上，net.ipv4.tcp_tw_recycle功能的开启，一般需要net.ipv4.tcp_timestamps（一般系统默认是开启这个功能的）这个开关开启后才有效果；
当tcp_tw_recycle 开启时（tcp_timestamps 同时开启，快速回收 socket 的效果达到），对于位于NAT设备后面的 Client来说，是一场灾难！！会导致到NAT设备后面的Client连接Server不稳定（有的 Client 能连接 server，有的 Client 不能连接 server）。

tcp_tw_recycle这个功能，其实是为内部网络（网络环境自己可控 ” -不存在NAT 的情况）设计的，对于公网环境下，不宜使用。通常来说，回收TIME_WAIT状态的socket是因为“无法主动连接远端”，因为无可用的端口，而不应该是要回收内存（没有必要）。也就是说，需求是Client的需求，Server会有“端口不够用”的问题吗？除非是前端机，需要大量的连接后端服务，也就是充当着Client的角色。

正确的解决这个总是办法应该是：

```
net.ipv4.ip_local_port_range = 9000 6553            #默认值范围较小
net.ipv4.tcp_max_tw_buckets = 10000                 #默认值较小，还可适当调小
net.ipv4.tcp_tw_reuse = 1 
net.ipv4.tcp_fin_timeout = 10 
```

4. 开启重用功能，允许将TIME-WAIT状态的sockets重新用于新的TCP连接。这个功能启用是安全的，一般不要去改动！
net.ipv4.tcp_tw_reuse = 1

5. 开启SYN Cookies，当出现SYN等待队列溢出时，启用cookies来处理。
net.ipv4.tcp_syncookies = 1

6. web应用中listen函数的backlog默认会给我们内核参数的net.core.somaxconn限制到128，而nginx定义的NGX_LISTEN_BACKLOG默认为511，所以有必要调整这个值。
net.core.somaxconn = 262144

7. 每个网络接口接收数据包的速率比内核处理这些包的速率快时，允许送到队列的数据包的最大数目。
net.core.netdev_max_backlog = 262144

8. 系统中最多有多少个TCP套接字不被关联到任何一个用户文件句柄上。如果超过这个数字，孤儿连接将即刻被复位并打印出警告信息。这个限制仅仅是为了防止简单的DoS攻击，不能过分依靠它或者人为地减小这个值，更应该增加这个值(如果增加了内存之后)。
net.ipv4.tcp_max_orphans = 262144

9. 记录的那些尚未收到客户端确认信息的连接请求的最大值。对于有128M内存的系统而言，缺省值是1024，小内存的系统则是128。
net.ipv4.tcp_max_syn_backlog = 262144

10. 时间戳可以避免序列号的卷绕。一个1Gbps的链路肯定会遇到以前用过的序列号。时间戳能够让内核接受这种“异常”的数据包。
net.ipv4.tcp_timestamps = 1

有不少服务器为了提高性能，开启net.ipv4.tcp_tw_recycle选项，在NAT网络环境下，容易导致网站访问出现了一些connect失败的问题。
建议：
关闭net.ipv4.tcp_tw_recycle选项，而不是net.ipv4.tcp_timestamps；
因为在net.ipv4.tcp_timestamps关闭的条件下，开启net.ipv4.tcp_tw_recycle是不起作用的；而net.ipv4.tcp_timestamps可以独立开启并起作用。

11. 为了打开对端的连接，内核需要发送一个SYN并附带一个回应前面一个SYN的ACK。也就是所谓三次握手中的第二次握手。这个设置决定了内核放弃连接之前发送SYN+ACK包的数量。
net.ipv4.tcp_synack_retries = 1

12. 在内核放弃建立连接之前发送SYN包的数量。
net.ipv4.tcp_syn_retries = 1

13. 如果套接字由本端要求关闭，这个参数 决定了它保持在FIN-WAIT-2状态的时间。对端可以出错并永远不关闭连接，甚至意外当机。缺省值是60秒。2.2 内核的通常值是180秒，你可以按这个设置，但要记住的是，即使你的机器是一个轻载的WEB服务器，也有因为大量的死套接字而内存溢出的风险，FIN- WAIT-2的危险性比FIN-WAIT-1要小，因为它最多只能吃掉1.5K内存，但是它们的生存期长些。
net.ipv4.tcp_fin_timeout = 30

14. 当keepalive起用的时候，TCP发送keepalive消息的频度。缺省是2小时。
net.ipv4.tcp_keepalive_time = 30

15. 缓冲区配置

`net.ipv4.tcp_rmem` 用来配置读缓冲的大小，三个值，第一个是这个读缓冲的最小值，第三个是最大值，中间的是默认值。我们可以在程序中修改读缓冲的大小，但是不能超过最小与最大。为了使每个socket所使用的内存数最小，我这里设置默认值为4096

`net.ipv4.tcp_wmem` 用来配置写缓冲的大小。读缓冲与写缓冲在大小，直接影响到socket在内核中内存的占用

`net.ipv4.tcp_mem` 则是配置tcp的内存大小，其单位是页，而不是字节。当超过第二个值时，TCP进入 pressure模式，此时TCP尝试稳定其内存的使用，当小于第一个值时，就退出pressure模式。当内存占用超过第三个值时，TCP就拒绝分配 socket了，查看dmesg，会打出很多的日志“TCP: too many of orphaned sockets”

`net.ipv4.tcp_max_orphans` 这个值也要设置一下，这个值表示系统所能处理不属于任何进程的 socket数量，当我们需要快速建立大量连接时，就需要关注下这个值了。当不属于任何进程的socket的数量大于这个值时，dmesg就会看 到”too many of orphaned sockets”

另外，附上LINUX 查看tcp连接数及状态，以及状态详请，参见： http://blog.sina.com.cn/s/blog_623630d50101r93l.html

以下是一个常用的内核参数的标准配置

```s
net.ipv4.ip_forward = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.default.accept_source_route = 0
kernel.sysrq = 0
kernel.core_uses_pid = 1
net.ipv4.tcp_syncookies = 1            //这四行标红内容，一般是发现大量TIME_WAIT时的解决办法
kernel.msgmnb = 65536
kernel.msgmax = 65536
kernel.shmmax = 68719476736
kernel.shmall = 4294967296
net.ipv4.tcp_max_tw_buckets = 6000
net.ipv4.tcp_sack = 1
net.ipv4.tcp_window_scaling = 1
net.ipv4.tcp_rmem = 4096 87380 4194304
net.ipv4.tcp_wmem = 4096 16384 4194304
net.core.wmem_default = 8388608
net.core.rmem_default = 8388608
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.core.netdev_max_backlog = 262144
net.core.somaxconn = 262144
net.ipv4.tcp_max_orphans = 3276800
net.ipv4.tcp_max_syn_backlog = 262144
net.ipv4.tcp_timestamps = 1            //在net.ipv4.tcp_tw_recycle设置为1的时候，这个选择最好加上
net.ipv4.tcp_synack_retries = 1
net.ipv4.tcp_syn_retries = 1
net.ipv4.tcp_tw_recycle = 1           //开启此功能可以减少TIME-WAIT状态，但是NAT网络模式下打开有可能会导致tcp连接错误，慎重。
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_mem = 94500000 915000000 927000000
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 30
net.ipv4.ip_local_port_range = 1024 65000
net.ipv4.ip_conntrack_max = 6553500
```

### TCP连接队列长度

编辑文件 /etc/sysctl.conf，添加如下内容：

`# The length of the syn quene`
`net.ipv4.tcp_max_syn_backlog =65535`

`# The length of the tcp accept queue`
`net.core.somaxconn =65535`

其中 tcp_max_syn_backlog用于指定半连接SYN队列长度，当新连接到来时，系统会检测半连接SYN队列，如果队列已满，则无法处理该SYN请求，
并在 /proc/net/netstat中的 ListenOverflows和 ListenDrops中增加统计计数somaxconn用于指定全连接ACCEPT队列长度，当该队列满了以后，客户端发送的ACK包将无法被正确处理，并返回错误"connection reset by peer"Nginx则会记录一条error日志"no live upstreams while connecting to upstreams"如果出现以上错误，我们需要考虑增大这两项的配置

### net.ipv4.ip_local_port_range

这个是可用端口范围

内核3.2之前 这个值决定了客户端的一个 ip 可用的端口数量，即一个 ip 最多只能创建 60K 多一点的连接（1025-65535），如果要突破这个限制需要客户端机器绑定多个 ip

内核3.2之后 这个值决定的是 socket 四元组中的本地端口数量，即一个 ip 对同一个目标 ip+port 最多可以创建 60K 多一点连接，只要目标 ip 或端口不一样就可以使用相同的本地端口，不一定需要多个客户端 ip 就可以突破端口数量限制

https://mozillazg.com/2019/05/linux-what-net.ipv4.ip_local_port_range-effect-or-mean.html

## 文件描述符

linux系统对文件描述符的限制有两个级别

系统级别：整个操作系统上所有进程能打开的文件描述符数量的总和。使用cat /proc/sys/fs/file-max查看，默认值是根据内存大小，系统自动设置的，一般为内存大小（KB）的10%，shell下可以这样计算grep -r MemTotal /proc/meminfo | awk '{printf("%d",$2/10)}'（可能有各种其他原因导致file-max没有设置为内存的10%）

hard和soft两个值都代表什么意思呢？
soft是一个警告值，而hard则是一个真正意义的阀值，超过就会报错。也就是一般说的软，硬设置

配置建议: 我们直接改一个用户层面，然后保证用户层面小于系统层面的值

### 临时修改

- 修改系统最大值

查看：`cat /proc/sys/fs/file-max`

临时修改：`echo 6553500 > /proc/sys/fs/file-max` （这种是通过写入参数到虚拟文件映射改变系统配置，针对proc路径下的文件）

- 修改单进程打开最大值

cat /proc/sys/fs/nr_open
echo 102400 > /proc/sys/fs/nr_open

nr_open的描述与file-max十分相近，不仔细看几乎分辨不出区别。
重点在于，file-max是对所有进程（all processes）的限制，而nr_open是对单个进程（a process）的限制

在CentOS7下（其他系统还未测试过），nofile的值一定不能高于nr_open，否则用户ssh登录不了系统，所以操作时务必小心：可以保留一个已登录的root会话，然后换个终端再次尝试ssh登录，万一操作失败，还可以用之前保留的root会话抢救一下

### 永久配置

修改了/etc/security/limits.conf和/etc/sysctl.conf两个配置文件，它们的区别在于limits.conf是用户层面的限制，而sysctl.conf是针对整个系统层面的限制

1.系统层级的限制 编辑文件 /etc/sysctl.conf，添加如下内容：

`fs.file-max =10000000`
`fs.nr_open =10000000`

/proc/sys/fs/file-max 查看时，系统会根据机器配置计算出一个预期值，通过这个配置修改我们可以覆盖它

2.用户层级的限制 编辑文件 /etc/security/limits.conf，添加以下内容：

`*      hard   nofile      1000000`
`*      soft   nofile      1000000`

更为详细的配置

```s
root soft nofile 1040000
root hard nofile 1040000
 
root soft nofile 1040000
root hard nproc 1040000
 
root soft core unlimited
root hard core unlimited
 
* soft nofile 1040000
* hard nofile 1040000
 
* soft nofile 1040000
* hard nproc 1040000
 
* soft core unlimited
* hard core unlimited
```

这里我们只要保证用户层级限制不大于系统层级限制就可以了，否则可能会出现无法通过SSH登录系统的问题。修改完毕执行如下命令：

`sysctl -p`
可以通过执行命令 ulimit -u查看是否修改成功

```s
配置含义

* soft nofile 100000
* hard nofile 100000
用户/组 软/硬限制 需要限制的项目 限制的值
```

## 如何标识一个TCP连接？

在确定最大连接数之前，先来看看系统如何标识一个tcp连接。系统用一个4四元组来唯一标识一个TCP连接：{local ip, local port,remote ip,remote port}

## client最大tcp连接数（理论上）

client每次发起tcp连接请求时，除非绑定端口，通常会让系统选取一个空闲的本地端口（local port），该端口是独占的，不能和其他tcp连接共享。tcp端口的数据类型是unsigned short，因此本地端口个数最大只有65536，端口0有特殊含义，不能使用，这样可用端口最多只有65535，所以在全部作为client端的情况下，最大tcp连接数为65535，这些连接可以连到不同的server ip

## server最大tcp连接数

server通常固定在某个本地端口上监听，等待client的连接请求。不考虑地址重用（unix的SO_REUSEADDR选项）的情况下，即使server端有多个ip，本地监听端口也是独占的，因此server端tcp连接4元组中只有remote ip（也就是client ip）和remote port（客户端port）是可变的，因此最大tcp连接为客户端ip数×客户端port数，对IPV4，不考虑ip地址分类等因素，最大tcp连接数约为2的32次方（ip数）×2的16次方（port数），也就是server端单机最大tcp连接数约为2的48次方

实际的连接数受到服务器限制，经验值：每个socket占用内存在15~20k之间

## 带宽测试

iperf3 工具进行带宽测试，需要把这个软件安装到服务端和客户端，或者可以是 服务端和 压测端

在其中一端启动监听服务，另一端去请求服务，以此测试带宽上限

```s
wget https://iperf.fr/download/source/iperf-3.1.3-source.tar.gz
tar zxvf iperf-3.1.3-source.tar.gz
cd iperf-3.1.3
./configure
make
make install

或者下面的方式，不行具体看官网版本

#wget http://downloads.es.net/pub/iperf/iperf-3.0.6.tar.gz
#tar zxvf iperf-3.0.6.tar.gz
# cd iperf-3.0.6
# sh configure
# make &&  make install
```

iperf3 -c  hostname  # 请求服务端
iperf3 -s            # 服务端启动监听

## nginx 容器启动参考

mkdir -p /home/nginx/www /home/nginx/logs /home/nginx/conf

-v /home/nginx/www:/usr/share/nginx/html -v /home/nginx/conf:/etc/nginx/conf.d -v /home/nginx/logs:/var/log/nginx

docker run --rm -d -p 8088:80 -v /home/nginx/www:/usr/share/nginx/html -v /home/nginx/conf:/etc/nginx/conf.d -v /home/nginx/logs:/var/log/nginx -v /home/nginx/base/nginx.conf:/etc/nginx/nginx.conf --name nginx-test-web nginx

docker run --rm -d -p 8088:80 -v /home/nginx/conf:/etc/nginx/conf.d --name nginx-test-web nginx

docker run --rm -d -p 8088:80 --name nginx-test-web nginx

## wrk

wrk 默认是 http1.1 协议，自带长连接，每次请求不需要再去三次握手建立 tcp 连接

-c：总的连接数（每个线程处理的连接数=总连接数/线程数）
-d：测试的持续时间，如2s(2second)，2m(2minute)，2h(hour)，默认为s
-t：需要执行的线程总数，默认为2，一般线程数不宜过多. 核数的2到4倍足够了. 多了反而因为线程切换过多造成效率降低
-s：执行Lua脚本，这里写lua脚本的路径和名称，后面会给出案例
-H：需要添加的头信息，注意header的语法，举例，-H “token: abcdef”
—timeout：超时的时间
—latency：显示延迟统计信息

使用方法: wrk <选项> <被测HTTP服务的URL>                            
  Options:                                            
    -c, --connections <N>  跟服务器建立并保持的TCP连接数量  
    -d, --duration    <T>  压测时间           
    -t, --threads     <N>  使用多少个线程进行压测   
                                                      
    -s, --script      <S>  指定Lua脚本路径       
    -H, --header      <H>  为每一个HTTP请求添加HTTP头      
        --latency          在压测结束后，打印延迟统计信息   
        --timeout     <T>  超时时间     
    -v, --version          打印正在使用的wrk的详细版本信息
                                                      
  <N>代表数字参数，支持国际单位 (1k, 1M, 1G)
  <T>代表时间参数，支持时间单位 (2s, 2m, 2h)

  Running 30s test @ http://www.baidu.com （压测时间30s）
  12 threads and 400 connections （共12个测试线程，400个连接）
			  （平均值） （标准差）  （最大值）（正负一个标准差所占比例）
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    （延迟）
    Latency   386.32ms  380.75ms   2.00s    86.66%
    (每秒请求数)
    Req/Sec    17.06     13.91   252.00     87.89%
  Latency Distribution （延迟分布）
     50%  218.31ms
     75%  520.60ms
     90%  955.08ms
     99%    1.93s 
  4922 requests in 30.06s, 73.86MB read (30.06s内处理了4922个请求，耗费流量73.86MB)
  Socket errors: connect 0, read 0, write 0, timeout 311 (发生错误数)
Requests/sec:    163.76 (QPS 163.76,即平均每秒处理请求数为163.76)
Transfer/sec:      2.46MB (平均每秒流量2.46MB)

参考

https://www.cnblogs.com/quanxiaoha/p/10661650.html



## 几个配置文件和常用命令

/etc/sysctl.conf
/etc/security/limits.conf

这是虚拟文件映射，可以通过文件获知系统参数：/proc/sys/fs/file-max

查看配置

sysctl -a |grep net

ulimit -n

## nginx

`worker_processes` 默认的Nginx只有一个master进程一个worker进程，我们需要对其进行修改，可以设置为指定的个数，也可以设置为 auto，即系统的CPU核数。更多的worker数量将导致进程间竞争cpu资源，从而带来不必要的上下文切换。因此这里我们将它设置为cpu的核数即可：worker_processes auto

`worker_connections` 每个worker可以处理的并发连接数，默认值512不是很够用，我们适当将它增大：worker_connections 4096

`worker_rlimit_nofile` 系统打开文件描述符数量，可以设置大一点1000000

## nginx 长连接

使用长连接能实现较少的连接数处理更多的请求
比如只使用100个连接，处理1000的并发请求，这在应对高并发的时候会非常有用，因为TCP的连接的创建需要消耗资源

当然这也是有局限性的，假如需要满足100W请求同时连接服务器，服务只能连接10W，这显然是不够的(要实现这点并不难，通过优化内核可以提高连接数，最好CPU核心数要非常高以保证性能)

- 支持keep alive长连接
当使用nginx作为反向代理时，为了支持长连接，需要做到两点：

从client到nginx的连接是长连接
从nginx到server的连接是长连接
从HTTP协议的角度看，nginx在这个过程中，对于客户端它扮演着HTTP服务器端的角色。而对于真正的服务器端（在nginx的术语中称为upstream）nginx又扮演着HTTP客户端的角色。

- 保持和client的长连接
为了在client和nginx之间保持上连接，有两个要求：

client发送的HTTP请求要求keep alive
nginx设置上支持keep alive

综合上述两点，我们要做的就是让：

`客户端`和`nginx`是长连接，通过`HTTP配置`来实现
`nginx`和`upstream(服务端server)`是长连接，通过配置`upstream`和`server的location`

### HTTP配置

```
http {
    keepalive_timeout  120s;
    keepalive_requests 10000;
}
```

keepalive_timeout: 设置keep-alive客户端连接在服务器端保持开启的超时值。值为0会禁用keep-alive客户端连接
keepalive_requests: 指令用于设置一个keep-alive连接上可以服务的请求的最大数量。当最大请求数量达到时，连接被关闭。默认是100
也就是说，当设置100的时候，nginx会为每个连接计数，当开启的这个长连接处理100请求后，就会被关闭，如果此时来的是1000的请求，那么会创建10次长连接

连接的创建到回收是有时间的，没被回收的连接就会处于TIME_WAIT状态

### 保持和server的长连接

HTTP协议中对长连接的支持是从1.1版本之后才有的，因此最好通过proxy_http_version指令设置为"1.1"，而"Connection" header应该被清理。清理的意思，我的理解，是清理从client过来的http header，因为即使是client和nginx之间是短连接，nginx和upstream之间也是可以开启长连接的。这种情况下必须清理来自client请求中的"Connection" heade 参考配置如下：

```
upstream BACKEND {
    keepalive 300;
    server 127.0.0.1:8081;
}
server {
    listen 8080;
    location /{
        proxy_pass http://BACKEND;
        proxy_http_version 1.1;
        proxy_set_header Connection"";
    }
}
```

其中 keepalive既非timeout，也不是连接池数量，官方解释如下：

可以看出它的意思是“`最大空闲长连接数量`”，超出这个数量的空闲长连接将被回收，当请求数量稳定而平滑时，空闲长连接数量将会非常小（接近于0），而现实中请求数量是不可能一直平滑而稳定的，当请求数量有波动时，空闲长连接数量也随之波动：

当空闲长连接数量大于配置值时，将会导致大于配置值的那部分长连接被回收；

当长连接不够用时，将会重新建立新的长连接。

因此，如果这个值过小的话，就会导致连接池频繁的回收、分配、再回收。为了避免这种情况出现，可以根据实际情况适当调整这个值，在我们实际情况中，目标QPS为6000，Web服务响应时间约为200ms，因此需要约1200个长连接，而 keepalive值取长连接数量的10%~30%就可以了，这里我们取300，如果不想计算，直接设为1000也是可行的

`这是一个特别特别需要注意的地方，因为它和上面HTTP配置不是一个意思，一般我们设置1000偷个懒即可`

### 参考

https://skyao.gitbooks.io/learning-nginx/content/documentation/official_document.html
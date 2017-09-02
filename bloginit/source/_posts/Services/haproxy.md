---
title: HAProxy安装与配置
description: HAProxy是基于TCP和HTTP的高可用负载均衡软件。
toc: true
categories:
  - Services
tags:
  - haproxy
  - load balance
date: 2017-08-09 23:13:19
updated: 2017-08-09 23:12:19
---
# HAProxy

官网：http://www.haproxy.org/

**HAProxy简介**

* 提供高可用性、负载均衡。
* 基于TCP（第四层）和HTTP（第七层）应用的代理。
* 特别适用于负载特大的web站点，这些站点通常又需要会话保持或七层处理。
* HAProxy实现了一种事件驱动、单一进程模型，此模型支持非常大的并发连接数。
* 支持全透明代理，支持连接拒绝。
* 支持虚拟主机的免费、快速并且可靠的解决方案。

## Install

```bash
VERSION=1.7.8
INSTALLPATH=/usr/local
wget http://www.haproxy.org/download/${VERSION%.*}/src/haproxy-${VERSION?empty}.tar.gz
cd haproxy-${VERSION?empty}
# 查看系统内核, 选择参数, see README.
uname -r
make TARGET=linux2628 PREFIX=${INSTALLPATH?empty}
make install PREFIX=${INSTALLPATH?empty}
```

## Uninstall

```bash
VERSION=1.7.8
INSTALLPATH=/usr/local
cd haproxy-${VERSION?empty}
make uninstall PREFIX=${INSTALLPATH?empty}
```

## Configuration

haproxy的配置文件由两部分组成：全局配置和代理配置。
共分为五段：global，defaults，listen，frontend，backend。

1. 全局配置
    * global    参数是进程级的，通常和操作系统(OS)相关。
2. 代理配置
    * defaults  用于为所有frontend，backend，listen配置段提供默认参数，这默认配置参数可由下一个defaults重新设定。
    * listen    通过关联frontend和backend定义了一个完整的代理，通常只对TCP流量有用。
    * frontend  用于定义一系列监听的套接字，这些套接字可接受客户端请求并与之建立连接。
    * backend   用于定义一系列后端服务器，代理将会将对应客户端的请求转发至这些服务器。

### (1)全局配置参数

global参数

```

globbal
        daemon    # 守护进程运行
        user nobody
        group nobody
        #uid 99
        #gid 99
        #node Haproxy_name    # 定义当前节点的名称，用于HA场景中多HA进程共享同一个IP地址时。
        #description strings    # 当前实例的描述信息。
        #chroot /usr/share/haproxy    # HA的工作目录的chroot()路径。
        pidfile /var/run/haproxy.pid    # HA进程的PID文件。
        nbproc 10    # 指定启动的进程的个数(默认1)，只能用于守护进程模式的haproxy。一般只在单进程仅能打开少数文件描述符的场景中才使用多进程模式。
        #ulimit-n 819200    # 设定每进程所能够打开的最大文件描述符数目，默认情况下其会自动进行计算，因此不推荐修改此选项。
        maxconn 20480    # 每个haproxy进程所接受的最大并发连接数,“ulimit -n”自动计算的结果是参照此参数设定的。
        #tune.maxaccept 100    # haproxy进程内核调度运行时一次性可以接受的连接的个数，较大的值可以带来较大的吞吐率，默认在单进程模式下为100，多进程模式下为8。设定为-1可以禁止此限制，一般不建议修改。
        #debug    # Debug相关的参数
        quiet    # Debug相关的参数
```

### (2)代理配置

*所有代理的名称只能使用大写字母、小写字母、数字、-(中线)、_(下划线)、.(点号)和:(冒号)。此外，ACL名称会区分字母大小写。*

```
#######################################
defaults
        mode http    # 默认的模式mode {tcp|http|health}，tcp是4层，http是7层，health只会返回OK。
        timeout check 10s    # 健康状态监测时的超时时间，过短会误判，过长资源消耗
        timeout client 1m    # 客户端非活动状态的超时时长
        timeout http-keep-alive 10s    # 定义保持连接的超时时长
        timeout http request 10s    # 在客户端建立连接但不请求数据时，关闭客户端连接
        timeout connect 10s  # 定义haproxy将客户端请求转发至后端服务器所等待的超时时长
        timeout server 1m    # 客户端与服务器端建立连接后，等待服务器端的超时时长，
        timeout queue 1m    # 等待最大时长
        #option httpclose    # 每次请求完毕后主动关闭http通道,haproxy不支持keep-alive,只能模拟这种模式的实现
        option http-server-close    # 在使用长连接时，为了避免客户端超时没有关闭长连接，此功能可以使服务器端关闭长连接
        option forwardfor    # 如果后端服务器需要获得客户端真实ip需要配置的参数，可以从Http Header中获得客户端ip
        #log global
        #log 127.0.0.1 local0 err    # [err warning info debug]  
        #option httplog    # 日志类别,采用httplog  
        #option dontlognull    # 不记录健康检查日志信息  

#######################################
listen admin_stat
        bind *:9999
        mode http
        stats enable
        stats uri /admin?stats
        stats realm HAProxy\ login    # 统计页面密码框上提示文本
        stats auth admin:password123    # 设置监控页面的用户和密码，配置多行可以设置多个用户名
        #stats admin if TRUE    # 设置手工启动/禁用，后端服务器(haproxy-1.4.9以后版本)
        stats refresh 10s    # 统计页面自动刷新时间
        #stats hide-version    # 隐藏统计页面上HAProxy的版本信息  
        acl admin_src src 192.168.1.33/32
        acl admin_src src 172.16.1.22/32
        block if ! admin_src

#######################################
# 使用listen做tcp转发配置
listen somename_tcp_8080
        bind 0.0.0.0:8080
        mode tcp
        option tcplog    # 日志类别,采用tcplog
        maxconn 50480
        retries 3
        #balance roundrobin    # 负载均衡算法: 
                               #   roundrobin(基于权重进行轮询)，权重可以在运行时进行调整，但每个后端服务器最多只接受4128个连接。
                               #   leastconn（WLC）：适用于长连接的会话，新的连接请求被派发至具有最少连接数目的后端服务器。
                               #   source(类似于nginx的ip_hash)：使同一个客户端IP的请求始终被派发至某特定的服务器(后端节点变化时重新计算)。
        #option redispatch    # 当serverId对应的服务器挂掉后，强制定向到其他健康的服务器
        #option abortonclose    # 当服务器负载很高的时候，自动结束掉当前队列处理比较久的链接  
        #option redispatch       # 当serverId对应的服务器挂掉后，强制定向到其他健康的服务器
        server tcp_server1 192.168.1.101:8888
        server tcp_server2 192.168.1.102:8888

#######################################
# 使用frontend做tcp转发
frontend transport_tcp_8090
        bind :8090
        mode tcp
        default_backend tcp_8090_default_back

backend tcp_8090_default_back
        mode tcp
        maxconn 50480
        timeout connect 10s
        timeout server 1m
        option tcplog
        option redispatch
        balance roundrobin
        #option abortonclose    # 当服务器负载很高的时候，自动结束掉当前队列处理比较久的链接  
        #option redispatch       # 当serverId对应的服务器挂掉后，强制定向到其他健康的服务器
        server tcp-199 192.168.1.199:80

#######################################
frontend http_80
        bind *:80
        mode http
        acl mysave hdr_beg(host) -i mysave.baidu.com    # 匹配请求报文的首部(host)内容，-i忽略大小写。
        acl api_qq_com hdr_beg(host) -i api.qq.com
	    acl callback url_beg -i /change/?ac=callback    # 匹配url以制定字符串开头，-i忽略大小写。 
        
        use_backend callbacksrv if callback mysave
        use_backend save if mysave
        use_backend api_qq_com_srv if api_qq_com

backend callbacksrv
        balance roundrobin
        option httpchk GET /ok.html HTTP/1.1\r\nHost:\ mysave.baidu.com    # 检测后端服务器存活状态
	    server call-139 192.168.1.139:8085 check inter 2000 rise 2 fall 5
        server call-60  192.168.1.60:8085  check inter 2000 rise 2 fall 5 backup

backend save
        balance roundrobin
        option httpchk GET /ok.html HTTP/1.1\r\nHost:\ mysave.baidu.com
        server save-67 192.168.1.67:8081 check inter 2000 rise 2 fall 3 weight 3
        server save-138  192.168.1.138:8080  check inter 2000 rise 2 fall 5 weight 5
        server save-14 192.168.1.14:8080 check inter 2000 rise 2 fall 5 weight 3 backup

backend api_qq_com_srv
        balance roundrobin
        option httpchk GET /index.php HTTP/1.1\r\nHost:\ api.qq.com
        server S38_hapi_8081  192.168.1.38:8081  check inter 2000 rise 2 fall 5 weight 7
        server S209_hapi_8081 192.168.1.209:8081 check inter 2000 rise 2 fall 5 weight 7
        server S61_hapi_8081  192.168.1.61:8081  check inter 2000 rise 2 fall 5 weight 2 backup
```

### 其他

时间参数值的表示：

```
us: 微秒(microseconds)
ms: 毫秒(milliseconds)
s: 秒(seconds)
m: 分钟(minutes)
h：小时(hours)
d: 天(days)

一般默认是ms
```

acl规则配置参考：
http://www.ttlsa.com/linux/haproxy-study-tutorial/

## running haproxy process

```bash
# 检查haproxy的配置文件是否正确：
/usr/local/sbin/haproxy -f /usr/local/etc/haproxy.conf -c && echo ‘haproxy check ok.’

# 启动haproxy进程：
/usr/local/sbin/haproxy -f /usr/local/etc/haproxy.conf

# 停止haproxy进程：
killall haproxy
```

# Refrence

* http://www.ttlsa.com/linux/haproxy-study-tutorial/
* http://www.cnblogs.com/Richardzhu/p/3344676.html
* http://blog.csdn.net/zhu_tianwei/article/details/41117323



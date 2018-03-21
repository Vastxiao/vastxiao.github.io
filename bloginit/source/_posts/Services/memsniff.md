---
title: memcache流量分析工具：memsniff
description: memsniff是一个基于go语言编写的开源的Memcached流量分析工具。
toc: true
categories:
  - Services
tags:
  - database
  - tools
  - memcache
date: 2017-11-26 13:00:00
updated: 2017-11-26 13:00:00
---

# memsniff

memsniff是一个开源的Memcached流量分析工具。

Github: https://github.com/box/memsniff

## install

**(1)依赖**

```bash
# redhat/centos
yum install libpcap-devel

# ubuntu
apt-get install libpcap-dev
```
**(2)golang**

memsniff是go写的，要安装golang：

```bash
# https://golang.org/doc/install#install

VERSION=1.9.2
TAG=linux-amd64
wget https://storage.googleapis.com/golang/go${VERSION?empty}.${TAG?empty}.tar.gz --no-check-certificate
[ $? -eq 0 ] && tar -zxvf go${VERSION?empty}.${TAG?empty}.tar.gz -C /usr/local/

echo 'export GOROOT=/usr/local/go' > /etc/profile.d/go.sh
echo 'export PATH=$PATH:/usr/local/go/bin' >> /etc/profile.d/go.sh
source /etc/profile.d/go.sh
```

**(3)memsniff install**

```bash
# 用go命令直接安装(需要bash环境的git命令)
go get github.com/box/memsniff
go build github.com/box/memsniff

# 无法直接用go命令安装的用以下方法(google被墙)
GO_HOME_DIR=/usr/local/go
git clone https://github.com/box/memsniff.git
mkdir -p ${GO_HOME_DIR?err}/src/github.com/box  # 注意go语言安装路径
mv memsniff /usr/local/go/src/github.com/box
go build github.com/box/memsniff
mv memsniff ${GO_HOME_DIR?err}/bin/
memsniff --version
```

## usage

```
memsniff -h

Usage of memsniff:
      --analysisworkers int   number of analysis workers (default 32)
      --assemblyworkers int   number of TCP assembly workers (default 8)
  -b, --buffersize int        MiB of kernel buffer for packet data (default 8)
      --cumulative            accumulate keys over all time instead of an interval
  -f, --filter string         regex pattern of cache keys to track
  -i, --interface string      network interface to sniff
  -n, --interval int          report top keys every this many seconds (default 1)
      --nodelay               replay from file at maximum speed instead of rate of original capture
      --nogui                 disable interactive interface
  -p, --ports intSlice        memcached ports to listen on (default [11211])
      --profile stringSlice   profile types to store (one or more of cpu, heap, block)
  -r, --read string           file to read (- for stdin)
  -t, --top int               number of keys to report (default 100)
      --version               display version information

# 使用：s
memsniff -i eth0 -p 12000
```

# 资料原文

http://mp.weixin.qq.com/s/A4pUAZrxWmFrjN5i55shwQ


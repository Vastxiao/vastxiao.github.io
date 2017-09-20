---
title: Scribe 安装使用
description: Scribe是Facebook开源的日志收集工具，centos采坑安装真是累!(facebook好些工具安装都好麻烦= =!)
toc: true
categories:
  - Services
tags:
  - tool
  - logs
  - scribe
date: 2017-09-20 19:53:04
updated: 2017-09-20 19:53:04
---

# scribe

Scribe是Facebook开源的日志收集工具。

github: https://github.com/facebookarchive/scribe

## 源码安装

**centos依赖**

```bash
yum install libevent-devel
yum install boost-devel
yum install python-devel
yum install git

# https://gist.github.com/stableShip/4a6acb51614920d7c24da7a2aaa89a72
# http://thrift.apache.org/docs/install/centos
```

**thrift+fb303**

```bash
# thrift
VERSION=0.5.0
wget http://archive.apache.org/dist/incubator/thrift/${VERSION?err}-incubating/thrift-${VERSION?err}.tar.gz
tar -zxvf thrift-${VERSION?err}.tar.gz
cd thrift-${VERSION?err}
#./configure --prefix=/usr/local/thrift-${VERSION?err}
./configure --prefix=/usr/local/thrift
make && make install
echo '/usr/local/thrift/lib' > /etc/ld.so.conf.d/thrift.conf
ldconfig
#[ $? -eq 0 ] && ln thrift-${VERSION?err} /usr/local/thrift
# fb303
cd contrib/fb303
./bootstrap.sh
./configure --prefix=/usr/local/fb303 --with-thriftpath=/usr/local/thrift
make
make install
```

**scribe install**

```bash
git clone https://github.com/facebookarchive/scribe.git
cd scribe
./bootstrap.sh --with-boost-filesystem=boost_filesystem --prefix=/usr/local/scribe --with-thriftpath=/usr/local/thrift --with-fb303path=/usr/local/fb303
make
make install
```

## scribe配置

scribe配置文件样例在源码包中：

```
# SCRIBE_SRC
scribe/examples/
├── example1.conf
├── example2central.conf
├── example2client.conf
├── hdfs_example2.conf
├── hdfs_example.conf
├── README
├── scribe_cat
└── scribe_ctrl
```

## scrib进程启动

命令：``/usr/local/scribe/bin/scribed [-p port] [-c config_file]``

```bash
# 在终端前端启动
/usr/local/scribe/bin/scribed -c /etc/scribe.conf

# 以守护进程启动
nohup /usr/local/scribe/bin/scribed -c /etc/scribe.conf &>/tmp/scribe.log &

# 停用进程，直接kill
killall scribed
```

## 资料

**``configure: error: Could not link against  !``**

http://running.iteye.com/blog/1983467


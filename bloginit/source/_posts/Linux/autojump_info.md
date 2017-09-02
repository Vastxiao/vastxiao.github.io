---
title: linux 超级方便的目录命令行工具：autojump
description: autojump是一个命令行工具，它允许你可以直接跳转到你喜爱的目录，而不用管你当前处于哪个目录。
toc: true
categories:
  - Linux
tags:
  - tools
date: 2017-04-14 22:17:02
updated: 2017-04-14 22:17:02
---

# autojump

autojump是一个命令行工具，它允许你可以直接跳转到你喜爱的目录，而不用管你现在身在何处。

github: https://github.com/wting/autojump

## 安装

Ubuntu/Debian上安装autojump：

```sh
sudo apt-get install autojump
```

Centos/Redhat上安装autojump：

```sh
# centos7直接安装新的2版本的autojump
# centos6/centos5的yum安装默认是旧的1.9版本，还请用源码安装新版本。
sudo yum install autojump
```

源码安装autojump：

```sh
VERSION=22.5.1
wget -O autojump-v${VERSION?unk?}.tar.gz https://github.com/wting/autojump/archive/release-v${VERSION?unk?}.tar.gz
tar -zxf autojump-v${VERSION?unk?}.tar.gz
cd autojump-release-v${VERSION?unk?}
# 如果不使用-d选项，默认安装到~/.autojump/目录中
./install.py -d /usr/local/autojump
#./uninstall.py -d /usr/local/autojump
```

## 配置

autojump安装完成后, 使用前需要先加载autojump环境:

```sh
# ubuntu 加载autojump环境:
. /usr/share/autojump/autojump.sh

# centos7 加载autojump环境:
. /usr/share/autojump/autojump.bash

# 源码安装的，加载环境变量：
# (路径需要根据源码安装位置) 
AUTOJUMP_HOME=/usr/local/autojump
export PATH=${AUTOJUMP_HOME?unknown?}/bin:$PATH
source ${AUTOJUMP_HOME?unknown?}/etc/profile.d/autojump.sh
unset AUTOJUMP_HOME
```

推荐将加载命令写入环境变量文件~/.bashrc中.

## 命令使用

```
usage: autojump [-h] [-a DIRECTORY] [-i [WEIGHT]] [-d [WEIGHT]] [-b]
                [--complete] [--purge] [-s] [-v]
                [DIRECTORY [DIRECTORY ...]]

Automatically jump to directory passed as an argument.

positional arguments:
  DIRECTORY             directory to jump to

optional arguments:
  -h, --help            show this help message and exit
  -a DIRECTORY, --add DIRECTORY
                        manually add path to database
  -i [WEIGHT], --increase [WEIGHT]
                        manually increase path weight in database
  -d [WEIGHT], --decrease [WEIGHT]
                        manually decrease path weight in database
  -b, --bash            enclose directory quotes to prevent errors
  --complete            used for tab completion
  --purge               delete all database entries that no longer exist on
                        system
  -s, --stat            show database entries and their key weights
  -v, --version         show version information and exit
```

## 参考

http://mp.weixin.qq.com/s/2IqlmInKPWjLtmzquKLkzw


---
title: tmux 编译安装指南
description: 自动化命令从github安装tmux工具。
toc: true
categories:
  - Linux
tags:
  - tmux
  - tools
date: 2018-03-22 00:02:00
updated: 2018-03-22 00:02:00
---


# tmux

**官网：**
https://tmux.github.io/

## tmux 从源码编译安装

### tmux软件依赖：

- libevent
- ncurses

#### 低版本的依赖可直接安装

```bash
# yum 安装到/usr下 :
yum install libevent-devel ncurses-devel
```

#### 高版本的依赖需要编译安装

```bash
LIBEVENT_BRANCH=release-2.0.22-stable
#LIBEVENT_HOME=/usr/local/libevent
LIBEVENT_BRANCH=${LIBEVENT_BRANCH:-"master"}
LIBEVENT_HOME=${LIBEVENT_HOME:-"/usr/local/libevent-${LIBEVENT_BRANCH}"}
git clone https://github.com/libevent/libevent.git
cd libevent
git checkout ${LIBEVENT_BRANCH?unknown}
./configure --prefix=${LIBEVENT_HOME%/}
make && make install

cd ../

NCURSES_BRANCH=v5.9
#NCURSES_HOME=/usr/local/ncurses
NCURSES_BRANCH=${NCURSES_BRANCH:-"master"}
NCURSES_HOME=${NCURSES_HOME:-"/usr/local/ncurses-${NCURSES_BRANCH}"}
git clone https://github.com/mirror/ncurses.git
cd ncurses
git checkout ${NCURSES_BRANCH?unknown}
./configure --prefix=${NCURSES_HOME%/}
make
# ../include/curses.h:1594:56: note: in definition of macro ‘mouse_trafo’解决方法：http://blog.csdn.net/velanjun/article/details/53102184
# make --with-termlib
make install

cd ../
```

### tmux编译安装：

**1.从github克隆编译安装**

```bash
#LIBEVENT_HOME=/usr/local/libevent
#NCURSES_HOME=/usr/local/ncurses
TMUX_HOME=
LIBEVENT_HOME=${LIBEVENT_HOME:-'/usr'}
NCURSES_HOME=${NCURSES_HOME:-'/usr'}
TMUX_HOME=${TMUX_HOME:-'/usr/local/tmux'}
git clone https://github.com/tmux/tmux.git
#git checkout 2.6
#git clone -b 2.6 https://github.com/tmux/tmux.git
cd tmux
sh autogen.sh
CFLAGS="-I${LIBEVENT_HOME%/}/include -I${LIBEVENT_HOME%/}/include/event2 -I${NCURSES_HOME%/}/include -I${NCURSES_HOME%/}/include/ncurses" \
LDFLAGS="-L${LIBEVENT_HOME%/}/lib -L${NCURSES_HOME%/}/lib" \
./configure --prefix=${TMUX_HOME%/}
make
make install
```

**2.将下面内容添加到环境变量文件中(~/.bashrc)**

```bash
ENV_FILE=
ENV_FILE=${ENV_FILE:-'~/.bashrc'}
echo "TMUX_HOME=${TMUX_HOME%/}" >> ${ENV_FILE}
echo 'export PATH=$TMUX_HOME/bin:$PATH' >> ${ENV_FILE}
echo 'unset TMUX_HOME' >> ${ENV_FILE}
echo "export LD_LIBRARY_PATH=${LIBEVENT_HOME%/}/lib:${NCURSES_HOME%/}/lib:"'$LD_LIBRARY_PATH' >> ${ENV_FILE}
echo 'ok...'
```

## 资料

tmux入门介绍: http://blog.jobbole.com/87278/


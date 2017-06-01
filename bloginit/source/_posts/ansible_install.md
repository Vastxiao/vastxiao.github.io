---
title: ansible安装
description: 自动化工具ansible的几种安装方式。
toc: true
categories:
  - linux
tags:
  - tools
  - ansible
date: 2017-06-1 22:35:38
updated: 2017-06-1 22:35:38
---


# ansible install

## Pre-install

### 对管理主机的要求

只要机器上安装了 Python 2.6 或 Python 2.7 (windows系统不可以做控制主机),都可以运行Ansible.

自2.0版本开始,ansible使用了更多句柄来管理它的子进程,对于OS X系统,你需要增加ulimit值才能使用15个以上子进程,否则你可能会看见”Too many open file”的错误提示.方法：
```
sudo launchctl limit maxfiles 1024 2048
```

### 对托管节点的要求

通常我们使用 ssh 与托管节点通信，默认使用 sftp.如果 sftp 不可用，可在 ansible.cfg 配置文件中配置成 scp 的方式. 

在托管节点上也需要安装 Python 2.4 或以上的版本.如果版本低于 Python 2.5 ,还需要额外安装一个模块: python-simplejson.

没安装python-simplejson,也可以使用Ansible的”raw”模块和script模块,
因此从技术上讲,你可以通过Ansible的”raw”模块安装python-simplejson,之后就可以使用Ansible的所有功能了.

Red Hat Enterprise Linux, CentOS, Fedora, and Ubuntu 等发行版都默认安装了 2.X 的解释器,包括几乎所有的Unix系统也是如此.

你可以使用 ‘raw’ 模块在托管节点上远程安装Python 2.X.例如：
```sh
# 这条命令可以通过远程方式在托管节点上安装 Python 2.X 和 simplejson 模块.
ansible myhost --sudo -m raw -a "yum install -y python2 python-simplejson"
```

## Install

ansible只需要安装在管理主机上。

### 从源码安装的步骤(使用git)

```sh
git clone git://github.com/ansible/ansible.git --recursive
cd ./ansible
# 安装到当前源码路径：
source ./hacking/env-setup

#如果没有安装pip, 请先安装对应于你的Python版本的pip:
sudo easy_install pip
# 以下的Python模块也需要安装 
sudo pip install paramiko PyYAML Jinja2 httplib2 six
```

注意,当更新ansible版本时,不只要更新git的源码树,也要更新git中指向Ansible自身模块的 “submodules” (不是同一种模块)

```
git pull --rebase
git submodule update --init --recursive
```

一旦运行env-setup脚本,就意味着Ansible从源码中运行起来了.默认的inventory文件是 /etc/ansible/hosts.inventory文件.

### 通过Yum安装RPMs适用于 EPEL 6, 7

```sh
# install the epel-release RPM if needed on CentOS, RHEL, or Scientific Linux
# 阿里yum源：
#wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-6.repo
#wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo

sudo yum install ansible
```

### 通过Apt(Ubuntu)安装最新发布版本

```sh
# 在早期Ubuntu发行版中, “software-properties-common” 名为 “python-software-properties”.
sudo apt-get install software-properties-common
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update
sudo apt-get install ansible
```

### 通过Pip安装最新发布版本

```
# 若你还没有安装 pip,可执行如下命令安装:
sudo easy_install pip
# 安装Ansible:
sudo pip install ansible
```

### 发行版的Tarball

http://releases.ansible.com/ansible/




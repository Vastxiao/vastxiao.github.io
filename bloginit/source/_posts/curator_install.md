---
title: Elastic Curator 简介和安装
description: ES curator工具的多种安装方法。
toc: true
categories:
  - elastic
tags:
  - elastic
  - curator
  - 安装
date: 2017-03-26 19:17:02
updated: 2017-03-26 19:17:02
---

# curator

* curator是一个用于管理es中的索引和快照的工具。
* curator是用Python写的，可以作为命令行工具，也能作为python的API。

https://www.elastic.co/guide/en/elasticsearch/client/curator/current/index.html

## Features

* Add or remove indices (or both!) from an alias
* Change shard routing allocation
* Close indices
* Create index
* Delete indices
* Delete snapshots
* Open closed indices
* forceMerge indices
* Change the number of replicas per shard for indices
* Take a snapshot (backup) of indices
* Restore snapshots

## Version Compatibility

curator对es版本的支持情况：

https://www.elastic.co/guide/en/elasticsearch/client/curator/current/version-compatibility.html

## Installation

从Curator 4.2开始，需要使用python3版本。

### pip

```sh
# 如果python没装pip，安装pip：
wget https://bootstrap.pypa.io/get-pip.py --no-check-certificate
python get-pip.py

# 安装当前版本
pip install elasticsearch-curator

# 安装特定版本：
pip install -U elasticsearch-curator==3.5.1

# 升级curator到最新新版本
pip install -U elasticsearch-curator

# 升级curator到指定版本
pip install -U elasticsearch-curator==3.5.1
```

### APT repository

```sh
wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo 'deb http://packages.elastic.co/curator/4/debian stable main' > /etc/apt/sources.list.d/curator.list
sudo apt-get update && sudo apt-get install elasticsearch-curator
```

### YUM repository

```sh
# 添加Elastic' Signing Key：
rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch

# RHEL/CentOS 6: 写/etc/yum.repos.d/curator.repo
[curator-4]
name=CentOS/RHEL 6 repository for Elasticsearch Curator 4.x packages
baseurl=http://packages.elastic.co/curator/4/centos/6
gpgcheck=1
gpgkey=http://packages.elastic.co/GPG-KEY-elasticsearch
enabled=1

# RHEL/CentOS 7: 写/etc/yum.repos.d/curator.repo
[curator-4]
name=CentOS/RHEL 7 repository for Elasticsearch Curator 4.x packages
baseurl=http://packages.elastic.co/curator/4/centos/7
gpgcheck=1
gpgkey=http://packages.elastic.co/GPG-KEY-elasticsearch
enabled=1

yum install elasticsearch-curator
```

### Source Code

https://www.elastic.co/guide/en/elasticsearch/client/curator/current/python-source.html

```sh
wget https://pypi.python.org/packages/source/p/package/package-#.#.#.tar.gz
tar zxf package-#.#.#.tar.gz
cd package-#.#.#
python setup.py install
```

## Curator Commands

- curator
- curator_cli




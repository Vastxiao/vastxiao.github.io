---
title: 迅雷协议URL的解码方法:decodeThreadURL
description: 这是自制的一个在linux终端下解码迅雷资源链接的简单小脚本.
toc: true
categories:
  - Linux
tags:
  - tools
date: 2017-05-12 02:36:22
updated: 2017-05-12 02:36:22
---

# decodeThreadURL

这是一个解码迅雷资源链接的简单小工具。

## 介绍
迅雷下载协议是经过加密的,如：

``thunder://QUFodHRwOi8vYmJzLmJ0d3VqaS5jb20vam9iLnBocD9hY3Rpb249ZG93bmxvYWQmcGlkPXRwYyZ0aWQ9MzYxNjAxJmFpZD0yNDYxNzFaWg==``

可以用此工具把真正的url链接解析出来:

```sh
root@Orb:~# decodeThunderURL thunder://QUFodHRwOi8vYmJzLmJ0d3VqaS5jb20vam9iLnBocD9hY3Rpb249ZG93bmxvYWQmcGlkPXRwYyZ0aWQ9MzYxNjAxJmFpZD0yNDYxNzFaWg==
http://bbs.btwuji.com/job.php?action=download&pid=tpc&tid=361601&aid=246171
root@Orb:~#
```

想使用这个工具很简单,只需要下载decodeThunderURL并给可执行权限即可,到PATH搜索路径中比较方便:

```bash
wget https://vastxiao.github.io/downloads/decodeThunderURL
[ $? -eq 0 ] && mv decodeThunderURL /usr/local/bin/
[ $? -eq 0 ] && chmod +x /usr/local/bin/decodeThunderURL
```


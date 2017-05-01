---
title: 轻量级Linux命令行测速工具speedtest-cli
description: speedtest-cli是一个用Python编写的轻量级Linux命令行工具，在Python2.4至3.4版本下均可运行。
toc: true
categories:
  - linux
tags:
  - tools
date: 2017-05-1 21:12:32
updated: 2017-05-1 21:12:32
---
# speedtest 网速测试工具

Speedtest.net的工作原理并不复杂：
它在你的浏览器中加载JavaScript代码并自动检测离你最近的Speedtest.net服务器，
然后向服务器发送HTTP GET and POST请求来测试上行/下行网速。

## web官网测速

http://www.speedtest.net/

## speedtest-cli

speedtest-cli是一个用Python编写的轻量级Linux命令行工具，在Python2.4至3.4版本下均可运行。
它基于Speedtest.net的基础架构来测量网络的上/下行速率。

**Github:**https://github.com/sivel/speedtest-cli

**安装speedtest-cli很简单——只需要下载其Python脚本文件:**

```sh
DESTDIR=/usr/local/bin
[ $? -eq 0 ] && sudo wget -O ${DESTDIR?noset?}/speedtest-cli https://raw.githubusercontent.com/sivel/speedtest-cli/master/speedtest.py
[ $? -eq 0 ] && sudo chown root:root ${DESTDIR?noset?}/speedtest-cli
[ $? -eq 0 ] && sudo chmod a+rx ${DESTDIR?noset?}/speedtest-cli
```

**命令:**

```
speedtest-cli -h
usage: speedtest-cli [-h] [--no-download] [--no-upload] [--bytes] [--share]
                    [--simple] [--csv] [--csv-delimiter CSV_DELIMITER]
                    [--csv-header] [--json] [--list] [--server SERVER]
                    [--mini MINI] [--source SOURCE] [--timeout TIMEOUT]
                    [--secure] [--no-pre-allocate] [--version]
```

# 参考资料

http://mp.weixin.qq.com/s/SqghQ3fxW--jCl5RSyTTkA



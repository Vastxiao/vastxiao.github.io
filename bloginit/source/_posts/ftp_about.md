---
title: ftp协议 功能及工作模式
description: 纯理论知识，关于ftp协议的基本功能及两种工作模式的介绍。
toc: true
categories:
  - linux
tags:
  - ftp
date: 2017-06-09 22:09:28
updated: 2017-06-09 22:09:28
---

# FTP 介绍

* FTP (File transfer protocol).
* FTP 最主要的功能是在服务器与客户端之间进行文件传输.
* FTP协议使用的是明文传输方式.

## FTP 功能

### (1)命令记录与登录文件记录

* FTP 可以利用系统的 syslogd 来进行数据的纪录,
* 记录的数据包括了用户曾经下达过的命令与用户传输数据(传输时间、文件w大小等等)的纪录.
* FTP 日志可以的在 /var/log/ 查到. 

### (2)不同等级的用户身份：user, guest, anonymous

FTP 服务器在预设的情况下，依据使用者登入的情况而分为三种不同的身份:

* 实体账号,real user
* 访客, guest
* 匿名登录者, anonymous

### (3)限制用户活动的目录：(change root, 简称 chroot)

FTP 可以限制用户仅能在自己的家目录当中活动，使用者无法离开自己的家目录。
而且登入 FTP 后，显示的根目录就是自己家目录的内容。

##  FTP 的运作流程与使用到的端口

FTP 的传输使用的是 TCP 封包协议，TCP 在建立链接前会先进行三次握手。
FTP 服务的交互比较复杂，使用了两个TCP连接，分别是命令信道与数据流通道(ftp-data)。

**FTP 有两种工作模式**

### FTP主动连接模式 (active) 

```seq
Client->FtpServer: ClientPort:A - 建立tcp通道连接 - FtpServerPort:21
Client->FtpServer: ClientPort:A - 知服务器端使用active且连接的client端口号B - FtpServerPort:21
FtpServer-->Client: ClientPort:B - 建立tcp连接 - FtpServerPort:20
```

1. 客户端会随机取一个大于1024以上的端口(port A)来与FTP服务器端的(port 21)建立连接，连接后客户端可以通过这个连接对FTP服务器发送命令。
2. 在有数据传输的行为时，客户端通知FTP服务器端使用active,且告知连接的端口号(port B)。
3. FTP服务进行数据传输前通过客户端的端口号通告，使用(port 20)主动与客户端建立连接后，开始交互传输数据。

如此一来就成功的建立起命令与数据传输两个通道。不过,要注意的是，
数据传输通道是在有数据传输的行为时才会建立的通道，
并不是一开始连接到FTP服务器就立刻建立的通道。

### FTP被动连接模式 (passive) 

```seq
Client->FtpServer: ClientPort:A - 建立tcp通道连接 - FtpServerPort:21
Client->FtpServer: ClientPort:A - 向服务器端发送passive连接请求 - FtpServerPort:21
FtpServer-->Client: ClientPort:A - 服务器返回passvie连接端口号(Port X) - FtpServerPort:21
Client->FtpServer: ClientPort:B - 建立tcp通道连接 - FtpServerPort:X
```

1. 客户端会随机取一个大于1024以上的端口(port A)来与FTP服务器端的(port 21)建立连接，连接后客户端可以通过这个连接对FTP服务器发送命令。
2. 在有数据传输的行为时，客户端通过命令通道向FTP服务器端发送passive连接请求。
3. 在服务器支持被动连接时，服务器先启动一个端口监听(这个端口号可能是随机的，也可以自定义某一范围的端口号),并讲端口号返回给通知客户端。
4. 接收服务器通知的端口号后客户端随机取用一个大于1024的端口号与服务器建立tcp连接。


# 原文资料

http://blog.csdn.net/wave_1102/article/details/50651433




---
title: Mysql抓包工具：mysql-sniffer
description: MySQL Sniffer 是一个基于 MySQL 协议的抓包工具，实时抓取 MySQLServer 端的请求，并格式化输出。
toc: true
categories:
  - Services
tags:
  - database
  - tools
  - mysql
date: 2017-07-26 22:36:32
updated: 2017-11-26 12:53:00
---

# mysql-sniffer

MySQL Sniffer 是一个基于 MySQL 协议的抓包工具，实时抓取 MySQLServer 端的请求，并格式化输出。
输出内容包访问括时间、访问用户、来源 IP、访问 Database、命令耗时、返回数据行数、执行语句等。
有批量抓取多个端口，后台运行，日志分割等多种使用方式，操作便捷，输出友好。

GitHub:
https://github.com/Qihoo360/mysql-sniffer

参考资料：
http://mp.weixin.qq.com/s/ixKOepIMOORIdnxAsaQp6g

## 依赖

yum安装：

```
yum install cmake unzip -y
yum install glib2-devel libpcap-devel -y
#wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-6.repo
yum install libnet-devel -y
```

## 安装

建议在 CentOS6.2 及以上编译安装，并用 root 运行。

```sh
wget -O mysql-sniffer.zip https://github.com/Qihoo360/mysql-sniffer/archive/master.zip
unzip mysql-sniffer.zip
cd mysql-sniffer-master
mkdir proj
cd proj
cmake ../
make
cd bin/
cp mysql-sniffer /usr/local/bin
```

## 命令帮助

```
Usage mysql-sniffer [-d] -i eth0 -p 3306,3307,3308 -l /var/log/mysql-sniffer/ -e stderr
         [-d] -i eth0 -r 3000-4000
         -d daemon mode.
         -s how often to split the log file(minute, eg. 1440). if less than 0, split log everyday
         -i interface. Default to eth0
         -p port, default to 3306. Multiple ports should be splited by ','. eg. 3306,3307
            this option has no effect when -f is set.
         -r port range, Don't use -r and -p at the same time
         -l query log DIRECTORY. Make sure that the directory is accessible. Default to stdout.
         -e error log FILENAME or 'stderr'. if set to /dev/null, runtime error will not be recorded
         -f filename. use pcap file instead capturing the network interface
         -w white list. dont capture the port. Multiple ports should be splited by ','.
         -t truncation length. truncate long query if it's longer than specified length. Less than 0 means no truncation
         -n keeping tcp stream count, if not set, default is 65536. if active tcp count is larger than the specified count, mysql-sniffer will remove the oldest one
```

## 输出形式

输出格式为：时间，访问用户，来源 IP，访问 Database，命令耗时，返回数据行数，执行语句。

```
[root@xm211 xiaoyufeng]# mysql-sniffer -i lo -p 3306
2017-03-06 09:37:01      linxs   192.168.50.211  my_world                 0ms             0      SET NAMES utf8
2017-03-06 09:37:01      linxs   192.168.50.211  my_world                 0ms            56      select id, game_id from `game_recommend` where status = 1
2017-03-06 09:37:02      linxs   192.168.50.211  latedemo                 0ms             0      use latedemo
2017-03-06 09:37:02      linxs   192.168.50.211  latedemo                 0ms             0      SET NAMES 'utf8'
2017-03-06 09:37:02      linxs   192.168.50.211  latedemo                 1ms             0      SELECT `a`.* FROM `staff` AS `a`  WHERE `a`.`jobnumber` = '1892' AND `a`.`suid` = '146'
2017-03-06 09:37:02      linxs   192.168.50.211  latedemo                 0ms             1      INSERT INTO `staff` (`jobnumber`,`name`,`suid`) VALUES ('1892','王文文','146')
```

**注意：**这个工具只是抓取经过指定网口的指定端口数据包，这里的输出并没有显示连接了哪个目目标ip地址。


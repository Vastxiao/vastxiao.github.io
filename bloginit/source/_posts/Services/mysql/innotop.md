---
title: Mysql Innodb分析工具：innotop
description: Innotop 可用于实时查看mysql的当前运行状态。
toc: true
categories:
  - Services
tags:
  - database
  - tools
  - mysql
date: 2017-10-25 22:45:00
updated: 2017-10-25 22:45:00
---

# Innotop

innotop is a 'top' clone for MySQL with many features and flexibility.

- completely customizable; it even has a plugin interface
- monitors many servers at once and can aggregate across them

## Web

google(old)
https://code.google.com/archive/p/innotop/

github(new)
https://github.com/innotop/innotop

## Requirement

```bash
# perl-DBD-MySQL
yum install perl-DBD-MySQL -y
# perl-Time-HiRes
yum install perl-Time-HiRes -y
# perl-DBI
yum install -y perl-DBI
# perl-TermReadKey
yum install -y perl-TermReadKey

#yum install -y perl-DBD-MySQL perl-Time-HiRes perl-DBI perl-TermReadKey
```

## Install

RPM Install:

```bash
wget https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/innotop/innotop-1.9.1-1.el6.noarch.rpm
rpm -ivh innotop-1.9.1-1.el6.noarch.rpm
```

Source Install:

```bash
VERSION=1.11.1
wget -O innotop-${VERSION}.tar.gz https://github.com/innotop/innotop/archive/v${VERSION}.tar.gz
tar -zxvf innotop-${VERSION}.tar.gz
cd innotop-${VERSION}
perl Makefile.PL
make
make install
```

## Usage

```
innotop --help

Usage: innotop <options> <innodb-status-file>

  --askpass          Prompt for a password when connecting to MySQL
  --[no]color   -C   Use terminal coloring (default)
  --config      -c   Config file to read
  --count            Number of updates before exiting
  --delay       -d   Delay between updates in seconds
  --help             Show this help message
  --host        -h   Connect to host
  --[no]inc     -i   Measure incremental differences
  --mode        -m   Operating mode to start in
  --nonint      -n   Non-interactive, output tab-separated fields
  --password    -p   Password to use for connection
  --port        -P   Port number to use for connection
  --skipcentral -s   Skip reading the central configuration file
  --socket      -S   MySQL socket to use for connection
  --spark            Length of status sparkline (default 10)
  --timestamp   -t   Print timestamp in -n mode (1: per iter; 2: per line)
  --user        -u   User for login if not current user
  --version          Output version information and exit
  --write       -w   Write running configuration into home directory if no config files were loaded

innotop is a MySQL and InnoDB transaction/status monitor, like 'top' for
MySQL.  It displays queries, InnoDB transactions, lock waits, deadlocks,
foreign key errors, open tables, replication status, buffer information,
row operations, logs, I/O operations, load graph, and more.  You can
monitor many servers at once with innotop.
```

## Reference

http://www.cnblogs.com/ivictor/p/5101506.html  
http://blog.csdn.net/wyzxg/article/details/8609981



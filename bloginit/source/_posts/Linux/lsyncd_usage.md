---
title: lsync：目录文件实时同步工具
description: lsync是基于rsync封装的文件目录实时同步工具。
toc: true
categories:
  - Linux
tags:
  - tools
date: 2017-09-02 14:25:11
updated: 2017-09-02 14:25:11
---

# lsync

官网：https://axkibe.github.io/lsyncd/

* Lysncd 实际上是lua语言封装了inotify和rsync工具，采用了Linux内核（2.6.13及以后）里的inotify触发机制，然后通过rsync去差异同步，达到实时的效果。
* 最亮的特性：完美解决了inotify+rsync海量文件同步带来的文件频繁发送文件列表的问题 —— 通过时间延迟或累计触发事件次数实现。
* 它的配置方式很简单，lua本身就是一种配置语言，可读性非常强。
* lsyncd也有多种工作模式可以选择，本地目录cp，本地目录rsync，远程目录rsyncssh。
* 实现简单高效的本地目录同步备份（网络存储挂载也当作本地目录），一个命令搞定。

# 安装lsyncd

## apt源

```bash
apt-get install lsyncd
```

## yum源

```bash
# yum安装需要epel源
# 阿里epel源地址：http://mirrors.aliyun.com/help/epel
yum install lsyncd
```

yum源安装文件结构：

|  路径  |  说明  |
|--------|--------|
| /etc/lsyncd.conf | 主配置文件 |
| /etc/sysconfig/lsyncd | init环境变量和启动选项配置文件 |
| /etc/logrotate.d/lsyncd | 日志滚动配置文件 |
| /usr/share/doc/lsyncd-*/examples/ | 目录下有lsyncd.conf配置例子 |
| /etc/init.d/lsyncd | lsyncd的init启动脚本 |
| /usr/bin/lsyncd | lsyncd命令路径 |
| /var/run/lsyncd/ | 可放lsyncd.pid的目录 |
| /var/log/lsyncd/ | 默认的日志目录 |

## 源码编译

```bash
# 源码下载：https://axkibe.github.io/lsyncd/download/
# 编译方式及依赖：https://axkibe.github.io/lsyncd/manual/building/
yum install -y lua cmake rsync

VERSION=2.1.5
wget -O lsyncd-release-${VERSION?err}.tar.gz https://github.com/axkibe/lsyncd/archive/release-${VERSION?err}.tar.gz
tar -zxvf lsyncd-release-${VERSION?err}.tar.gz
cd lsyncd-release-${VERSION?err}
cmake .
make
sudo make install
```

# lsyncd配置(lsyncd.conf)

lsyncd的一个配置文件总体分类：

1. settings(global)：settings是全局进程设置。 
2. sync(layer 4)：sync里面是定义同步参数，可以有多个sync，各自的sync配置，互不影响。
3. onAction(layer 3): 用于定义sync触发的事件动作，定义后的Action应用到sync配置下。
（例如监控某个目录下的文件，根据触发的事件自己定义要执行的命令。）
4. AdvancedonAction(layer 2)：自定义事件模型，定义后可应用到layer3和layer4。
5. Inlets(layer 1)：自定义事件函数，一般在自定义事件模型(layer 2)中使用自定义事件函数。
5. 注释：--开头表示注释

## settings

```lua
settings {
    -- # 进程pid文件
    --pidfile = "/var/run/lsyncd/lsyncd.pid",
    -- # nodaemon为true(默认)表示不启用守护模式。
    --nodaemon   = false,
    -- # inotifyMode指定inotify监控的事件，默认是CloseWrite，还可以是Modify。
    inotifyMode = "CloseWrite",
    -- # 同步进程的最大个数。如maxProcesses=8，则最大能看到有8个rysnc进程。
    maxProcesses = 8,
    -- # statusFile定义状态文件。
    statusFile = "/var/run/lsyncd/lsyncd.status",
    -- # statusInterval将lsyncd的状态写入上面的statusFile的间隔，默认10秒。
    statusInterval = 10,
    -- # logfile
    logfile = "/var/log/lsyncd/lsyncd.log"
}
```

_注:_

* 在没有配置settings选项时也能通过lsyncd命令指定``lsyncd --help``
* 或者yum安装的，由/etc/init.d/lsyncd脚本启动的，也可通过/etc/sysconfig/lsyncd配置。
* 命令行制定的参数比配置文件中的settings优先生效。

## sync (layer 4)

https://axkibe.github.io/lsyncd/manual/config/layer4/

sync配置形式：

```lua
sync {
   -- # 选项
   --default.rsync,
   --source = "DIRNAME",
   --target = "DIRNAME"
   -- # 选项...
}
```

### sync选项：

__default.*__

sync一般第一个参数指定lsyncd以什么模式运行：rsync、rsyncssh、direct三种模式：

* default.direct ：本地目录间同步，使用cp、rm等命令完成差异文件备份。
* default.rsync ：本地目录间同步，使用rsync，也可以达到使用ssh形式的远程rsync效果，或daemon方式连接远程rsyncd进程；
* default.rsyncssh ：同步到远程主机目录，rsync的ssh模式，需要使用key来认证

__source__

source同步的源目录，使用绝对路径。

__target__

target定义目的地址.对应不同的模式有几种写法：

* /tmp/dest ：本地目录同步，可用于direct和rsync模式
* 172.29.88.223:/tmp/dest ：同步到远程服务器目录，可用于rsync和rsyncssh模式。
* 172.29.88.223::module ：同步到远程服务器目录，用于rsync模式

__init__

init 是一个优化选项，当init = false，
只同步进程启动以后发生改动事件的文件，
原有的目录即使有差异也不会同步。默认是true。

__delay__

delay 累计事件，等待rsync同步延时时间，默认15秒（最大累计到1000个不可合并的事件）。
也就是15s内监控目录下发生的改动，会累积到一次rsync同步，避免过于频繁的同步。
（可合并的意思是，15s内两次修改了同一文件，最后只同步最新的文件）

__excludeFrom__

排除选项，后面指定排除的文件``excludeFrom = "/etc/lsyncd.exclude"``。
排除规则：

```
- 监控路径里的任何部分匹配到一个文本，都会被排除，例如/bin/foo/bar可以匹配规则foo
- 如果规则以斜线/开头，则从头开始要匹配全部
- 如果规则以/结尾，则要匹配监控路径的末尾
- ?匹配任何字符，但不包括/
- *匹配0或多个字符，但不包括/
- **匹配0或多个字符，可以是/
```

__exclude__

排除选项，后面跟一个列表``exclude = { '.bak' , '.tmp' }``。
规则和excludeFrom一样。

__delete__

delete 为了保持target与souce完全同步，Lsyncd默认会delete = true来允许同步删除。
值：

- true
- false
- startup
- running

__rsync__

这个rsync选项用在default.rsync和default.rsyncssh模式。

rsync配置形式：

```lua
rsync = {
    -- # binary指定rsync命令路径
    binary = "/usr/bin/rsync",
    -- # compress 压缩传输默认为true。在带宽与cpu负载之间权衡，本地目录同步可以考虑把它设为false
    compress = false,
    -- # perms 默认保留文件权限
    perms = true,
    acls = true,
    xattrs = true,
    links = true,
    quiet = true,
    verbose = true,
    --ipv4 = true,
    --ipv6 = true,
    -- # 其它rsync的命令选项
    _extra = {"--bwlimit=2000", "--password-file=/etc/rsyncd_work.pass"},
    -- # 指定使用ssh方式进行rsync的ssh选项
    --rsh = "/usr/bin/ssh -p 22 -o StrictHostKeyChecking=no"
}
```

__ssh__

这个ssh配置用在default.rsyncssh模式中。

配置形式：

```lua
ssh = {
    binary = "/usr/bin/ssh",
    identityFile = "/path/to/id_rsa",
    port = 22,
    -- # options是ssh的-o选项指定的参数
    --options = {"RhostsRSAAuthentication no", "PasswordAuthentication no"},
    -- # _extra跟 rsync的_extra一样
    _extra = {"--bwlimit=2000"} 
}
```

## onAction (layer 3)

https://axkibe.github.io/lsyncd/manual/config/layer3/

事件的写法：

```lua
-- # bash是自定义的事件名
bash = {
    --# 可选项，覆盖上级配置:
    delay = 5,
    maxProcesses = 3,
    -- # 事件动作处理流程：
    onCreate = "cp -r ^sourcePathname ^targetPathname",
    onModify = "cp -r ^sourcePathname ^targetPathname",
    onDelete = "rm -rf ^targetPathname",
    onMove   = "mv ^o.targetPathname ^d.targetPathname",
    onStartup = '[[ if [ "$(ls -A ^source)" ]; then cp -r ^source* ^target; fi]]',
}

-- # 应用方式
--sync{bash, source="/path/to/src", target="/path/to/trg/"}
```

action说明：

| action | description |
|--------|-------------|
| onAttrib | 文件属性变化时触发。 |
| onCreate | 文件或目录被创建时触发。 |
| onModify | 文件被修改时触发。 |
| onDelete | 文件或目录被删除时触发。 |
| onMove | 文件或目录被移动时触发。 |
| onStartup | lsyncd程序启动时触发。 |

可用变量(variables)：

```
^source          # source目录的绝对路径
^target	         # target目录的绝对路径
^path            # 监测的目录相对路径，末尾有斜杠。
^pathname        # 监测的目录相对路径，末尾没有斜杠。
^sourcePath	     # 监测source的目录(带/)或文件的路径。
^sourcePathname  # 监测source的目录(不带/)或文件的路径。
^targetPath      # 监测targetPathname的目录(带/)或文件的路径。
^targetPathname  # 监测targetPathname的目录(不带/)或文件的路径。
```

## Advanced onAction (layer 2)

https://axkibe.github.io/lsyncd/manual/config/layer2/

## Inlets (layer 1)

https://axkibe.github.io/lsyncd/manual/config/layer1/

# 实用的配置实例

```lua
settings {
   --pidfile = "/var/run/lsyncd/lsyncd.pid",
   --nodaemon   = false,
   inotifyMode = "CloseWrite",
   maxProcesses = 8,
   statusFile = "/tmp/lsyncd.status",
   statusInterval = 10,
   logfile = "/var/log/lsyncd/lsyncd.log"
}



-- # 监测本地目录发生变化就用touch更新一下mtime时间。
flushModifyTime ={
    delay = 10,
    maxProcesses = 10,
    onCreate  = "touch ^sourcePathname",
    onModify  = "touch ^sourcePathname",
}
sync {
    flushModifyTime,
    source    = "/videos_store/video/",
    --delete    = false
}



-- # 本地目录同步，direct：cp/rm/mv。 适用：500+万文件，变动不大
sync {
    default.direct,
    source    = "/tmp/src",
    target    = "/tmp/dest",
    delay = 1
    maxProcesses = 1,
} 

-- # 本地目录同步，rsync模式：rsync
sync {
    default.rsync,
    source    = "/tmp/src",
    target    = "/tmp/dest1",
    excludeFrom = "/etc/rsyncd.d/rsync_exclude.lst",
    rsync     = {
        binary = "/usr/bin/rsync",
        archive = true,
        compress = true,
        bwlimit   = 2000
        } 
    }



-- # 远程目录同步，rsync模式 + rsyncd daemon
sync {
    default.rsync,
    source    = "/tmp/src",
    target    = "syncuser@172.29.88.223::module1",
    delete="running",
    exclude = { ".*", ".tmp" },
    delay = 30,
    init = false,
    rsync     = {
        binary = "/usr/bin/rsync",
        archive = true,
        compress = true,
        verbose   = true,
        password_file = "/etc/rsyncd.d/rsync.pwd",
        _extra    = {"--bwlimit=200"}
        }
    }


-- # 远程目录同步，rsync模式 + ssh shell
sync {
    default.rsync,
    source    = "/tmp/src",
    target    = "172.29.88.223:/tmp/dest",
    -- target    = "root@172.29.88.223:/remote/dest",
    -- 上面target，注意如果是普通用户，必须拥有写权限
    maxDelays = 5,
    delay = 30,
    -- init = true,
    rsync     = {
        binary = "/usr/bin/rsync",
        archive = true,
        compress = true,
        bwlimit   = 2000
        -- rsh = "/usr/bin/ssh -p 22 -o StrictHostKeyChecking=no"
        -- 如果要指定其它端口，请用上面的rsh
        }
    }



-- # 远程目录同步，rsync模式 + rsyncssh，效果与上面相同
sync {
    default.rsyncssh,
    source    = "/tmp/src2",
    host      = "172.29.88.223",
    targetdir = "/remote/dir",
    excludeFrom = "/etc/rsyncd.d/rsync_exclude.lst",
    -- maxDelays = 5,
    delay = 0,
    -- init = false,
    rsync    = {
        binary = "/usr/bin/rsync",
        archive = true,
        compress = true,
        verbose   = true,
        _extra = {"--bwlimit=2000"},
        },
    ssh      = {
        port  =  1234
        }
    }
```

# 进程使用

## lsyncd命令：

```
USAGE:
  runs a config file:
    lsyncd [OPTIONS] [CONFIG-FILE]

  default rsync behaviour:
    lsyncd [OPTIONS] -rsync [SOURCE] [TARGET]

  default rsync with mv's through ssh:
    lsyncd [OPTIONS] -rsyncssh [SOURCE] [HOST] [TARGETDIR]

  default local copying mechanisms (cp|mv|rm):
    lsyncd [OPTIONS] -direct [SOURCE] [TARGETDIR]

OPTIONS:
  -delay SECS         Overrides default delay times
  -help               Shows this
  -insist             Continues startup even if it cannot connect
  -log    all         Logs everything (debug)
  -log    scarce      Logs errors only
  -log    [Category]  Turns on logging for a debug category
  -logfile FILE       Writes log to FILE (DEFAULT: uses syslog)
  -nodaemon           Does not detach and logs to stdout/stderr
  -pidfile FILE       Writes Lsyncds PID into FILE
  -runner FILE        Loads Lsyncds lua part from FILE
  -version            Prints versions and exits
```

## 进程启动

```bash
lsyncd -pidfile /var/run/lsyncd.pid /etc/lsyncd.conf
# 注，要关掉直接kill进程。
```

## 启动相关报错：

**max_user_watches较小**

```
日志：
Fri Sep  1 11:31:20 2017 Error: Terminating since out of inotify watches.
Consider increasing /proc/sys/fs/inotify/max_user_watches

查看max_user_watches：
sysctl -a|grep max_user_watches

解决方法：改大fs.inotify.max_user_watches
vim /etc/sysctl.conf然后添加修改: fs.inotify.max_user_watches = 2001192
生效命令： sysctl -p
```

# 资料原文

http://seanlook.com/2015/05/06/lsyncd-synchronize-realtime/


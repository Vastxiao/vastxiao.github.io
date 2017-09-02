---
title: vsftp配置指南
description: vsftp的安装配置的详细指南。
toc: true
categories:
  - Linux
tags:
  - ftp
date: 2017-06-11 10:15:03
updated: 2017-06-11 10:15:03
---


# vsftpd

vsftpd提供linux的ftp服务。
vsftpd较为安全但相对其他ftp服务功能较少。

## 功能

1. vsftpd服务的启动者身份为一般用户，对Linux系统的权限较低。
2. vsftpd利用chroot()这个函数进行改换根目录的动作，使得系统工具不会被vsftpd这支服务所误用。
任何需要具有较高执行权限的vsftpd指令均以一支特殊的上层程序所控制，该上层程序享有的较高执行权限功能已经被限制的相当的低，并以不影响Linux本身的系统为准。
3. 绝大部分ftp会使用到的额外指令功能(dir, ls, cd …)都已经被整合到vsftpd主程序当中了，因此理论上vsftpd不需要使用到额外的系统提供的指令，所以在chroot的情况下，vsftpd不但可以顺利运作，且不需要额外功能对于系统来说也比较安全。
4. 所有来自客户端且想要使用上层程序(也是使用chroot功能的程序)所提供的较高执行权限的vsftpd指令的需求，均被视为不可信任的要求来处理，必需要经过相当程度的身份确认后，方可利用该上层程序的功能。例如chown,Login的要求等等动作。

## 安装

**yum安装**

```bash
yum install vsftpd -y
```

**apt安装**

```sh
apt-get install vsftpd -y
```

## vsftpd包结构

* /etc/vsftpd/vsftpd.conf vsftpd主配置文件(配置方式是参数=设定值)
* /etc/pam.d/vsftpd vsftpd使用PAM模块时的相关配置文件。

## 服务管理

```
# Centos
/etc/init.d/vsftpd {start|stop|restart|try-restart|force-reload|status}
```

## 服务配置

### 1. 服务器环境的配置

配置 /etc/vsftpd/vsftpd.conf

```
##################
# 服务器环境配置 #
##################

# 以后台进程启动(standalone),默认YES。
background=YES
# 监听端口，以standalone模式启动生效。
listen=YES
# vsftpd命令通道连接端口，记得防火墙设置。
listen_port=21
# standalone下，同一时间最大client连接数，0不限制，默认2000。
max_clients=0
# standalone下，同一个IP同一时间允许的最大连接数。
max_per_ip=0
# 用户连接后超时无命令动作强制断线时间(s)
idle_session_timeout=300
# 已建立的连接，在该时间(s)内无法完成数据传输，则被强制断开。
data_connection_timeout=300


# YES则允许FTP主动连接模式，默认NO。
connect_from_port_20=NO
# 主动连接模式，服务器发出连接超时后未响应强制断线时间(s)
connect_timeout=60


# 设置YES则支持ftp被动连接模式(passive mode)，默认YES。
pasv_enable=YES

# 被动连接模式使用的端口范围(默认0使用任何端口)
# 记得限制端口号范围，开启防火墙。
pasv_min_port=0
pasv_max_port=0

# PASV模式下，等待client超时无回应强制断线时间(s)
accept_timeout=60


# 用户连接ftp时，ftp客户端上显示的说明信息。
#ftpd_banner=welcome to ftp !!
# 用户连接ftp时，把文件内容在ftp客户端上显示，如果开启则文件必须存在。
#banner_file=/etc/login_info
# 当用户进入某个目录时，会显示该目录中message_file配置的文件内容。
dirmessage_enable=YES
message_file=.message


# (链接使用子进程管理)每个连接都使用单独process负责,一般不配置.默认不启用(NO)
#one_process_model=NO

# 支持TCP Wrappers(既/etc/hosts.allow|/etc/hosts.deny控制)
tcp_wrappers=YES

# 使用本地时间。（vsftp默认使用GMT时间）
use_localtime=YES

# YES: client就优先使用ASCII格式下载文件,默认NO。
#ascii_download_enable=YES
# YES: client就优先使用ASCII格式上传文件,默认NO。
#ascii_upload_enable=YES

# vsftpd 服务启动用户,默认使用nobody
nopriv_user=nobody

# 必须配置支持PAM模块管理，pam模块名称, 就是/etc/pam.d/vsftpd
pam_service_name=vsftpd


# 启用日志,设置为YES时，客户端上传/下载文件都会被纪录。(默认NO)
xferlog_enable=YES
# 为YES则启用xferlog的wu-ftpd的日志格式记录日志，使用xferlog_file
# 为NO则使用vsftpd的日志格式记录日志(默认NO)，使用vsftpd_log_file
xferlog_std_format=YES
# 使用xferlog的wu-ftpd日志格式的写文件路径。
xferlog_file=/var/log/xferlog
# 使用vsftpd的日志格式记录日志的写文件路径。
vsftpd_log_file=/var/log/vsftpd.log
# 同时启用两种日志格式（vsftp日志格式和xferlog的wu-ftp日志）,默认NO。
#dual_log_enable=YES


# 在上传文件时，创建文件的可执行权限umask，默认0666
file_open_mode=0666

# 限制用户使用ftp的命令(这里禁止删除命令)
cmds_allowed=ABOR,CWD,LIST,MDTM,MKD,NLST,PASS,PASV,PORT,PWD,QUIT,RETR,SIZE,STOR,TYPE,USER,REST,CDUP,HELP,MODE,NOOP,REIN,STAT,STOU,STRU,SYST,FEAT

```

### 2. 匿名账号登录配置

配置 /etc/vsftpd/vsftpd.conf

```
####################
# 匿名账号登录配置 #
####################

# 匿名账号也是虚拟用户，账号名就是anonymous，权限也依赖guest_username配置。
# 设置允许匿名账号(anonymous)登录vsftpd，默认YES
anonymous_enable=NO
# 允许anonymous账号的读取和下载文件的权限，默认YES。
anon_world_readable_only=YES
# 设置anonymous账号对vsftp上的文件写权限，默认NO。
anon_other_write_enable=NO
# 设置anonymous账号在vsftp上创建目录的权限，默认NO。
anon_mkdir_write_enable=NO
# 设置anonymous账号的上传数据权限，默认NO。
anon_upload_enable=NO
# 限制anonymous上传文件的权限(077则anonymous上传的文件权限是-rw—--—---)
anon_umask=077

# 限制anonymous的传输速度，单位为bytes/秒，0则不限制(默认)。
# 如果想让anonymous仅有30KB/s的速度,设定anon_max_rate=30000
anon_max_rate=0

# 设置YES则anonymous登录vsftp时需要输入密码(登录时会检查输入的emai)，默认NO。
#no_anon_password=YES
# anonymous登录vsftp时，会要求输入email地址做密码，启用该项可禁止某些email的登录，默认NO。
#deny_email_enable=YES
# 启用限制登录vsftp的email列表文件，文件内每个email一行，启用email限制后此文件必须存在。
#banned_email_file=/etc/vsftpd/banned_emails
```

### 3. 实体用户登录配置

配置 /etc/vsftpd/vsftpd.conf

```
####################
# 实体用户登录配置 #
####################

# 设置为YES，在/etc/passwd内的账号(或PAM配置的)才能以实体用户的方式登入vsftpd。
# 针对实体用户和虚拟用户，这个值必须开启。
local_enable=YES


# 启用chroot功能，限制登录的账号根目录在自己家目录,(默认NO)
chroot_local_user=YES
# 启用chroot后，chroot_list_enable配置为YES则/etc/vsftpd/chroot_list列表为不被限制的账号；
# 启用chroot后，chroot_list_enable为NO则/etc/vsftpd/chroot_list为需要被chroot的账号（默认）。
chroot_list_enable=YES
# 启用chroot后，chroot_list_file是针对chroot_list_enable设置的账号列表文件，即使没有任何账号，此文件也要存在。
chroot_list_file=/etc/vsftpd/chroot_list


# 是否启用账号登录访问控制，默认NO。
userlist_enable=YES
# 启用userlist时，此选项为YES则userlist_file中的账号不允许登录vsftp服务器,(默认项)。
# 启用userlist时，此选项为NO则userlist_file中的账号为允许登录vsftp服务器。
userlist_deny=YES
userlist_file=/etc/vsftpd/user_list

# YES则允许实体用户和虚拟用户下载数据，默认YES。
download_enable=YES
# YES则允许实体用户和虚拟用户上传数据(包括目录和文件)，默认NO
#write_enable=YES
# 实体用户和虚拟用户创建新目录(755)与文件(644)的权限
local_umask=022

# 实体用户的传输速度限制，单位为bytes/second，0为不限制。
local_max_rate=0
```

### 4. 虚拟用户登录配置

(1)配置 /etc/vsftpd/vsftpd.conf

```
####################
# 虚拟用户登录配置 #
####################

# 该值设置为YES时，任何实体账号都会被当成虚拟用户，权限也依赖虚拟用户的配置(所以默认是NO)。
guest_enable=NO
# guest_enable为YES时生效，指定虚拟用户的身份(虚拟用户的宿主用户)。
guest_username=ftp


# 配置YES，则虚拟用户使用本地用户的权限；配置NO，虚拟用户使用匿名用户的权限，默认NO。
# 启用虚拟用户后，实体账号默认都被当成虚拟用户并且被chroot，开启该项能让实体用户恢复实体用权限，
# 但无论如何，在vsftpd服务器上进行实际操作的用户名任然是guest_username所配置的用户。
# 不过还是建议配置为NO，特殊用户权限可以使用虚拟账号自身配置文件配置。
virtual_use_local_privs=NO

# 虚拟用户的配置文件的主目录(目录自己创建)，该目录下存放虚拟用户的个人配置。
# 每个用户的配置一个文件，配置文件名=虚拟用户名。
user_config_dir=/etc/vsftpd/account_conf

```

(2)配置PAM认证(根据主配置文件中pam_service_name的配置)。

修改 /etc/pam.d/vsftpd 注释掉所有参数，在最后面加入下面两行：

```
auth    required        pam_userdb.so   db=/etc/vsftpd/account
account required        pam_userdb.so   db=/etc/vsftpd/account
```

注意：配置PAM认证数据库后，实体账号也无法直接登录vsftp，也需要在认证数据中添加账号才能登录。

(3)生成用户认证数据库 /etc/vsftpd/account.db

生成数据库使用的工具是Berkeley DB工具，安装包是：

```sh
# yum 安装
yum install db4 db4-utils 
```

生成数据库需要用户密码文本，添加虚拟用户名和密码，奇数行为用户名，偶数行为密码。

/etc/vsftpd/account.txt

```
user1
password1
user2
password2
```

根据用户密码文本最终生成用户认证数据库：

```bash
# 生成vsftp用户名和密码认证数据库
# 每次修改用户和密码信息都必须，用该命令重载一次数据库。
db_load -T -t hash -f /etc/vsftpd/account.txt /etc/vsftpd/account.db
```

(4)虚拟用户个人vsftp权限配置方式(根据主配置文件中user_config_dir的配置)。

在 /etc/vsftpd/account_conf/ 目录下创建虚拟用户配置文件：

每个用户的配置为一个文件，配置文件名为虚拟用户名，配置内容根据情况选择：

```
# 虚拟用户的根目录，需要预先建立并赋予相应权限
local_root=/data/user1

# 配置YES，则虚拟用户使用本地用户的权限；配置NO，虚拟用户使用匿名用户的权限。
virtual_use_local_privs=NO

# 匿名权限，YES则允许读取和下载文件的权限。
anon_world_readable_only=YES
# 匿名权限，YES则允许文件/目录写权限(删除、重命名)。
anon_other_write_enable=NO
# 匿名权限，YES则允许创建目录。
anon_mkdir_write_enable=NO
# 匿名权限，YES则允许上传数据。
anon_upload_enable=NO
# 匿名权限，限制上传文件的权限掩码(drwx------/-rw-------)
anon_umask=077
# 匿名权限，限制传输速度，单位为bytes/秒，0则不限制。
anon_max_rate=0

# 虚拟用户权限，YES则允许实体用户和虚拟用户下载数据。
download_enable=YES
# 虚拟用户权限，YES则允许上传数据(包括目录和文件)
write_enable=YES
# 虚拟用户权限，限制创建新目录(755)与文件(644)的权限
local_umask=022
# 虚拟用户权限，传输速度限制，单位为bytes/second，0为不限制。
local_max_rate=0

# 其他vsftpd中用户相关的配置也可以使用，例如：
idle_session_timeout=0
```

## 针对用户使用命令的权限管理

在vsftpd.conf中存在两个对用户使用ftp命令的限制：

* cmds_allowed
* cmds_deny

cmds_deny优先级高于cmds_allowed，两个配置同一个命令时，以cmds_deny生效。

这两个命令也都可以应用于虚拟用户个人的权限配置文件。

参数说明：  
http://fity.cn/post/317.html  
http://itchentao.blog.51cto.com/5168625/1563813

## 参考资料

- http://blog.csdn.net/wave_1102/article/details/50651433
- http://www.huzs.net/?p=1213#server_real_chroot



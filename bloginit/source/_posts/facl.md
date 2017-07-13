---
title: linux强大的facl功能
description: facl(POSIX Access Control Lists)是针对linux系统文件权限的高级访问控制。
toc: true
categories:
  - linux
tags:
  - facl
date: 2017-06-18 12:06:29
updated: 2017-07-13 16:39:07
---


# facl 

POSIX Access Control Lists

Linux ACL 权限控制：  
ACL即Access Control List 主要的目的是提供传统的owner,group,others的read,write,execute权限之外的具体权限设置，
ACL可以针对单一用户、单一文件或目录来进行r,w,x的权限控制，对于需要特殊权限的使用状况有一定帮助。
如，某一个文件，不让单一的某个用户访问。

## facl support

要使用ACL必须要有文件系统支持才行，目前绝大多数的文件系统都会支持，EXT3文件系统默认启动ACL的。

ext4文件系统：

```
# 查看EXT4文件系统是否支持ACL：
dumpe2fs -h /dev/sda2
# 检查mount options选项： Default mount options:    user_xattr acl

# 如果UNIX支持ACL但是文件系统并不是默认加载此功能，可自己进行添加
mount -o acl /dev/sda1 /export1
mount -o remount,acl /data
```

xfs文件系统：xfs系统已经强制开启了ACL权限了。

## facl soft

**libacl (库)**
- /lib64/libacl.so.1
- /lib64/libacl.so.1.1.0
 
**libacl-devel (库)**
- /lib64/libacl.so
- /usr/include/acl
- /usr/include/acl/libacl.h
- /usr/include/sys/acl.h
- /usr/lib64/libacl.a
- /usr/lib64/libacl.so

**acl (包)**
- /usr/bin/chacl
- /usr/bin/getfacl
- /usr/bin/setfacl

## facl command

- getfacl：取得某个文件/目录的ACL设置项目
- setfacl：设置某个文件/目录的ACL设置项目

### ◆  getfacl

getfacl命令

```
Usage: getfacl [-aceEsRLPtpndvh] file ...
```

### ◆  setfacl

setfacl命令

```
Usage: setfacl [-bkndRLPvh] [{-m|-x} acl_spec] [{-M|-X} acl_file] file ...

OPTION参数:
  -h                 显示帮助信息
  -m acl_spec        设置后续的acl参数，不可与'-x'一起使用
  -M acl_spec_file   根据提供的acl_spec_file文件中的acl设置文件权限
  -x acl_spec        删除后续的acl参数，不可与'-m'一起使用
  -X acl_spec_file   根据提供的acl_spec_file文件中的acl设置文件权限
  -b                 删除所有的acl参数，参数单独使用
  -k                 删除默认的acl参数，参数单独使用
  -R                 递归设置acl参数
  -d                 设置默认的acl参数，只对目录有效，作用使目录下新
                     建的子目录和文件继承父目录的默认acl权限。

  acl_spec_file的内容就是权限内容。

acl_spec格式:
  [d:]u:[username|uid]:perms
  [d:]g:[groupname|gid]:perms
  [d:]o::perms
  [d:]m::perms
  
  perms: 权限内容是‘rwx-’中的3位组合
      ---
      rwx
      r-x
```

setfacl详解

```
# 对特殊用户设置的facls，如果不指定用户则对文件本身生效，同chmod设置。
setfacl -m u:antony:rw /mnt/gluster/data/testfile

# 对用户组设置的facls,如果不指定用户组则对文件本身生效，同chmod设置。
setfacl -m g:usergroup:rw /mnt/gluster/data/testfile

# 针对有效权限设置的facls
# 有效权限（mask）就是acl权限设置的极限值，也就是所设置的acl权限
# 一定是mask的一个子集，如果超出mask范围会将超出的权限去掉.
setfacl -m m::rw /PATH/TO/DIR_OR_FILE

# 设置目录下的新建的文件或子目录的默认ACLs权限
# 针对一个目录为一个用户（组）设置特定权限，但是如果这个目录下在
# 新创建的文件是不具有这些针对这个用户的特定权限的。为了解决这个
# 问题，就需要设置默认 acl 权限，使这个目录下新创建的文件有和目录
# 相同的默认ACL特定权限。因此该设置只能设置目录。
setfacl -m d:u:antony:rw /mnt/gluster/data/dirname
setfacl -d -m u:antony:rw /path/to//dirname

# Removing POSIX ACLs
# 移除已设置的POSIX ACLs权限方式
setfacl -x u:antony /mnt/gluster/data/test-file
setfacl -x g:usergroup:rw /mnt/gluster/data/testfile

# 删除文件权限列表,一下子就全删了，+号都没有了
setfacl -b /path/to/name

# 删除默认的acl参数，会删除默认(defaultt)的权限列表
setfacl -k /path/to/dirname
```

### ◆  查看权限列表

```
[user@linux Desktop]$ ls  -l  file 
-rw-rw-r--      .      1 root group1 0 Nov  7 15:13 file

# 如果此位为 “.”  代表这位没有权限列表
# 如果此位为 “+”  代表有权限列表存在
```

## facl useful

一招搞定开发项目的目录权限：

```bash
chown -R root:www /www #确定目录属组
chmod 2775 /www #目录属组下文件及目录使用共享组权限
setfacl -R -m d:u::rwx /www #配置该目录下创建目录或文件时的默认权限
setfacl -R -m d:g::rwx /www #配置该目录下创建目录或文件时的默认权限
setfacl -R -m d:m::rwx /www #配置该目录下创建目录或文件时的默认权限
setfacl -R -m d:o::rx /www #配置该目录下创建目录或文件时的默认权限

# 这样设置完后，以后给人员权限只要给linux账号添加组就够了：
usermod -a -G Username
```

## reference

http://www.linuxidc.com/Linux/2013-07/88049.htm  
http://wiki.jikexueyuan.com/project/learn-linux-step-by-step/acl-permission-control.html




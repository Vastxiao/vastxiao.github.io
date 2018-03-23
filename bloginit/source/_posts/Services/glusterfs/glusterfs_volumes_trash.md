---
title: glusterfs卷的回收站(Trash)功能.
description: glusterfs的回收站用于保存卷中被删除的文件，可用来备份删除的文件，预防误删除操作。
toc: true
categories:
  - Services
tags:
  - storage
  - tools
  - glusterfs
date: 2018-02-17 13:00:00
updated: 2018-03-22 09:45:00
---

# glusterfs volume trash

glusterfs 卷的回收站功能.

http://docs.gluster.org/en/latest/Administrator%20Guide/Trash/

## 功能特性

* trash功能默认关闭的．
* trash目录默认是.trashcan, 基于挂载卷根目录中．
* trash目录和卷目录共用一个存储空间.
* trash功能允许处理的文件大小限制默认是5MB．
* 启用trash功能后, 删除的文件会被重命名添加时间戳移到trash的相对目录中.

## volume命令选项

```bash
# 启用和停用trash功能:
gluster volume set <VOLNAME> features.trash <on / off>

# 设置trash目录，默认: .trashcan
gluster volume set <VOLNAME> features.trash-dir <name>

# 限制使用trash功能的文件最大大小，默认5MB (单位: KB MB GB)
# 如果删除的文件大小超过这个值,那么该文件删除后不会被放入回收站.
gluster volume set <VOLNAME> features.trash-max-filesize <size>

# 排除不适用trash功能的目录.
gluster volume set <VOLNAME> features.trash-eliminate-path <path1> [ , <path2> , . . . ]

# This command can be used to enable trash for internal operations
# like self-heal and re-balance. By default set to off.
gluster volume set <VOLNAME> features.trash-internal-op <on / off>
```

## 配置

```bash
gluster volume set <VOLNAME> features.trash on
gluster volume set <VOLNAME> features.trash-max-filesize 100GB
```


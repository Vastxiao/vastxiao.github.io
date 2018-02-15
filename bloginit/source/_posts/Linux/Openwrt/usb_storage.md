---
title: Openwrt路由器系统配置支持USB存储设备
description: 这是Openwrt挂在U盘、移动硬盘的教程。
toc: true
categories:
  - Linux
tags:
  - Openwrt
date: 2018-02-15 14:54:00
updated: 2018-02-15 14:54:00
---

# Openwrt路由器系统配置支持USB存储设备

https://wiki.openwrt.org/zh-cn/doc/howto/usb.storage

## 1. OpenWrt设备上获得基本USB支持

https://wiki.openwrt.org/doc/howto/usb.essentials

需要软件包：

* kmod-usb-core
* kmod-usb2

```sh
# Modules for USB 2.0
opkg update
opkg install kmod-usb2
insmod ehci-hcd
# If you see messages like unresolved symbol usb_calc_bus_time try loading usbcore and then try ehci-hcd again:
#opkg update
#insmod usbcore
#insmod ehci-hcd
```

## 2. OpenWrt上配置挂在USB设备环境

https://wiki.openwrt.org/zh-cn/doc/howto/usb.storage#required_packages_for_usb_storage

相关软件包:

* kmod-usb-storage (required) 内核支持usb存储设备需要安装包.
* kmod-fs-<file_system> (required) 根据文件系统安装(examples include kmod-fs-ext4, kmod-fs-msdos, kmod-fs-ntfs, and kmod-fs-xfs).
* kmod-usb-storage-extras (optional) 内核支持更多的设备(SmartMedia card readers).
* block-mount (recommended & required) 挂在设备支持包.
* kmod-scsi-core (Any mass storage is a generic SCSI device)

```sh
# 例子支持挂在usb设备ext4文件系统
opkg update
opkg install kmod-usb-storage block-mount
opkg install kmod-fs-ext4
#mkswap /dev/sda1
#swapon /dev/sda1
mkdir -p /mnt/usb
mount -t ext4 /dev/sda2 /mnt/usb -o rw,sync
```

禁止在未挂载时写入

```sh
# You may create an empty file to indicate that the disk is not plugged in so that you don't put files directly onto NAND by doing
umount /mnt/usb   #make sure the disk isn't mounted before doing this
touch /mnt/usb/USB_DISK_NOT_PRESENT
chmod 555 /mnt/usb 
chmod 444 /mnt/usb/USB_DISK_NOT_PRESENT
```

直接写入/etc/rc.local在启动系统时自动挂载

```
# /etc/rc.local
/bin/mount -t ext4 /dev/sda /mnt/usb -o rw,sync
```


---
title: Shadowsocks 翻墙神器!
description: shadowsocks是个翻墙的神级软件，本章内容包含完整搭建ss服务的内容。
toc: true
categories:
  - Services
tags:
  - ss
  - shadowsocks
date: 2018-03-10 16:45:00
updated: 2018-03-10 16:45:00
---

# shadowsocks (ss)

* 官网：shadowsocks.org
* github: https://github.com/shadowsocks
* wiki: https://github.com/shadowsocks/shadowsocks/wiki

##　服务端搭建(Server)

https://shadowsocks.org/en/download/servers.html

### 安装:

[^1]:

**Ubuntu依赖**

```bash
#安装libsodium支持高级加密(支持chacha20-ietf-poly1305加密)
sudo apt install libsodium-dev

# 另一安装支持chacha20-ietf-poly1305加密:
# (Ubuntu)https://linuxssh.com/ubuntu-anzhuang-chacha20-ietf-poly1305-jiami-shadowsocks/
# Ubuntu17.04和17.10自带Shadowsocks-libev，可直接安装。
#sudo apt install shadowsocks-libev
```

**Centos依赖**

```bash
# 安装libsodium支持高级加密(支持chacha20-ietf-poly1305加密)注意安装shadowsocks3.0的版本(从github安装)
# yum源:
#   wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo # epel(RHEL 7)
#   wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-6.repo # epel(RHEL 6)
yum install libsodium -y  # for Yum-base install


# 以下是另一安装加密支持的方式: (支持chacha20-ietf-poly1305加密)
# (Centos)https://linuxssh.com/cntos7-anzhuang-chacha20-ietf-poly1305-shadowsocks/
# YUM源: https://copr.fedorainfracloud.org/coprs/librehat/shadowsocks/
#cd /etc/yum.repos.d/
##curl -O https://copr.fedorainfracloud.org/coprs/librehat/shadowsocks/repo/epel-6/librehat-shadowsocks-epel-6.repo
#curl -O https://copr.fedorainfracloud.org/coprs/librehat/shadowsocks/repo/epel-7/librehat-shadowsocks-epel-7.repo
#yum install -y shadowsocks-libev
```

**shadowsocks安装**

```bash
# 依赖Python 2.6 or 2.7 +
# https://www.python.org/downloads/
# PyPI安装:
pip install shadowsocks

# 或
# GitHub安装:
git clone https://github.com/shadowsocks/shadowsocks.git
cd shadowsocks
git checkout master
python setup.py build
python setup.py install
```

### 启动服务:

**ssserver服务配置项说明**

* server: your hostname or server IP (IPv4/IPv6).
* server_port: server port number.
* local_port: local port number.
* password: a password used to encrypt transfer.
* timeout: connections timeout in seconds.
* method: encryption method. [more...](https://shadowsocks.org/en/spec/Stream-Ciphers.html)
    - chacha20-ietf-poly1305  (shadowsocks 3.0.0+)
    - aes-256-gcm  (shadowsocks 3.0.0+)
    - aes-256-cfb

**启动ssserver服务**

```bash
# 直接使用命令选项启动服务:
nohup ssserver -s 0.0.0.0 -p 1111 -k "z7T0XRpassword" -t 600 -m chacha20-ietf-poly1305 </dev/null &>/tmp/ssserver.log &

# To run in the background
ssserver -p 443 -k password -m aes-256-cfb --user nobody -d start

# 或着启动服务指定配置文件:
nohub ssserver -c /path/to/ssserver.json </dev/null &>/tmp/ssserver.log &
```

ssserver.json:

```
{
    "server": "0.0.0.0",
    "server_port": 1111,
    "password":"z7T0XRpassword",
    "timeout": 60,
    "method":"chacha20-ietf-poly1305",
    "fast_open": false,
    "user": "nobody",
    "PID_FILE": "/var/run/shadowsocks_ssserver.pid",
    "LOG_FILE": "/var/run/shadowsocks_ssserver.log"
}
```

### 服务器参数优化

**Step 1, increase the maximum number of open file descriptors**

编辑/etc/security/limits.conf文件

```
echo '* soft nofile 51200' >> /etc/security/limits.conf
echo '* hard nofile 51200' >> /etc/security/limits.conf

```

启动ssserver前先执行``ulimit -n 51200``

**Step 2, Tune the kernel parameters**

编辑内核配置文件/etc/sysctl.conf

```
fs.file-max = 51200

net.core.rmem_max = 67108864
net.core.wmem_max = 67108864
net.core.netdev_max_backlog = 250000
net.core.somaxconn = 4096

net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 0
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.ip_local_port_range = 10000 65000
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_max_tw_buckets = 5000
net.ipv4.tcp_fastopen = 3
net.ipv4.tcp_mem = 25600 51200 102400
net.ipv4.tcp_rmem = 4096 87380 67108864
net.ipv4.tcp_wmem = 4096 65536 67108864
net.ipv4.tcp_mtu_probing = 1
net.ipv4.tcp_congestion_control = hybla
```

执行``sysctl -p``重新加载内核参数.

**启用BBR**

https://linuxssh.com/centos-qiyong-bbr/

```bash
# 内核版本必须大于4.9
uname -sr

echo “net.core.default_qdisc = fq” >> /etc/sysctl.conf
echo “net.ipv4.tcp_congestion_control = bbr” >> /etc/sysctl.conf
sysctl -p
```

## 客户端搭建(Client)

https://shadowsocks.org/en/download/clients.html

### (Linux|Windows|Mac OS X)命令行使用:

**sslocal客户端工具安装**

[^1]安装方式跟服务端搭建方式一样.

**客户端启动命令**

```bash
# 直接使用sslocal命令选项启动:
sslocal -s 48.48.48.48 -p 1111 -b 192.168.1.209 -l 1080 -k 'z7T0XRpassword' -m chacha20-ietf-poly1305 -t 600 --user nobody --pid-file /var/run/shadowsocks.pid --log-file /var/log/shadowsocks.log -d start

# 或着启动时指定配置文件:
sslocal -c /path/to/sslocal.json
```

sslocal.json

```
{
    "server": "48.38.87.48",
    "server_port": 1111,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"z7T0XRpassword",
    "timeout": 60,
    "method":"chacha20-ietf-poly1305",
    "fast_open": false,
    "user": "nobody",
    "PID_FILE": "/var/run/shadowsocks_sslocal.pid",
    "LOG_FILE": "/var/run/shadowsocks_sslocal.log"
}
```

### OpenWRT

```sh
opkg install shadowsocks-libev
opkg install shadowsocks-libev-polarssl
```

### Android

支持
* shadowsocks-android: [GitHub](https://github.com/shadowsocks/shadowsocks-android/releases)

### iOS

支持

### Windows GUI Client

* shadowsocks-win: [GitHub](https://github.com/shadowsocks/shadowsocks-windows/releases)
* Shadowsocks-Qt5: [GitHub](https://github.com/shadowsocks/shadowsocks-qt5/releases)

## 使用socks5请求代理客户端

### curl

```bash
curl 'https://www.google.com/' --socks5-hostname 127.0.0.1:1080
```

### 代理插件 SwitchyOmega (for Chromium & Firefox)

https://github.com/FelisCatus/SwitchyOmega/releases

### 通过proxychains工具在终端命令行使用代理

**安装proxychains**

```bash
# On Debian/Ubuntu:
sudo apt install proxychains
```

**配置proxychains**

```
strict_chain
proxy_dns 
remote_dns_subnet 224
tcp_read_time_out 15000
tcp_connect_time_out 8000
localnet 127.0.0.0/255.0.0.0
quiet_mode

[ProxyList]
socks5  127.0.0.1 1080
```

**使用proxychains代理**
```bash
# Run command with proxychains. Examples:
proxychains4 curl https://www.twitter.com/
proxychains4 git push origin master

# Or just proxify bash:
proxychains4 bash
curl https://www.twitter.com/
git push origin master
```


---
title: nginx配置ssl支持https协议
description: nginx需要配置ssl来支持https协议，本文是关于nginx配置ssl的方法。
toc: true
categories:
  - Services
tags:
  - openssl
  - CA
  - SSL/TLS
  - nginx
  - https
date: 2017-10-08 14:38:00
updated: 2017-10-08 14:30:00
---

# nginx为https协议配置ssl

http://nginx.org/en/docs/http/configuring_https_servers.html#chains

**[ssl证书文件](https://vastxiao.github.io/article/2017/09/03/Linux/openSSl_for_CA_and_Certificate/)**

配置ssl需要先获得两个配对的文件：服务器ssl密钥文件(server.key)和CA颁发的证书(server.crt)。

## ssl配置

nginx关于ssl的配置结构：

nginx.conf

```
worker_processes auto;

http {
    ssl_session_cache   shared:SSL:10m;
    ssl_session_timeout 10m;

    server {
        listen              443 ssl;
        server_name         www.example.com;
        keepalive_timeout   70;

        ssl_certificate     www.example.com.crt;
        ssl_certificate_key www.example.com.key;
        ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers         HIGH:!aNULL:!MD5;

        ...
    }

    ...
}
```

### server{} 配置示例

**最简单的配置**

```
server {
    listen               443 ssl;
    ssl_certificate      /path/to/nginx_ssl_file_path/server.crt;
    ssl_certificate_key  /path/to/nginx_ssl_file_path/server.key;

    root                 /www/domain_wwwroot;
}
```

**HTTPS Config**

```
server {
    listen              443 ssl;
    server_name         www.example.com;
    ssl_certificate     www.example.com.crt;
    ssl_certificate_key www.example.com.key;
    ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers         HIGH:!aNULL:!MD5;
    ...
}

server {
    listen 443 ssl;
    ssl_certificate /etc/ssl_keys/domain/server.crt;
    ssl_certificate_key /etc/ssl_keys/domain/server.key;
    #ssl_protocols SSLv2 SSLv3 TLSv1 TLSv1.1 TLSv1.2;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    #ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    ssl_session_timeout 5m;

    root /www/domian_wwwroot;
}
```

**A single HTTP/HTTPS server**

```
server {
    listen              80;
    listen              443 ssl;
    server_name         www.example.com;
    ssl_certificate     www.example.com.crt;
    ssl_certificate_key www.example.com.key;
    ...
}
```

**An SSL certificate with several names**

```
ssl_certificate     common.crt;
ssl_certificate_key common.key;

server {
    listen          443 ssl;
    server_name     www.example.com;
    ...
}

server {
    listen          443 ssl;
    server_name     www.example.org;
    ...
}
```

[nginx 同一个IP上配置多个HTTPS主机](http://www.ttlsa.com/web/multiple-https-host-nginx-with-a-ip-configuration/)

## nginx的ssl相关模块

* ngx_http_ssl_module  http://nginx.org/en/docs/http/ngx_http_ssl_module.html
* ngx_mail_ssl_module  http://nginx.org/en/docs/mail/ngx_mail_ssl_module.html
* ngx_stream_ssl_module  http://nginx.org/en/docs/stream/ngx_stream_ssl_module.html
* ngx_stream_ssl_preread_module  http://nginx.org/en/docs/stream/ngx_stream_ssl_preread_module.html


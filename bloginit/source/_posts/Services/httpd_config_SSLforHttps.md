---
title: apache(httpd)配置ssl支持https协议
description: 本文是关于apache-httpd配置ssl的方法。
toc: true
categories:
  - Services
tags:
  - openssl
  - CA
  - SSL/TLS
  - apache
  - httpd
  - https
date: 2017-10-10 21:25:00
updated: 2017-10-10 21:25:00
---

# Apache httpd 配置https

* apache要提供https服务需要在httpd中启用ssl。
* httpd启用ssl功能需要加载mod_ssl模块。

## (1)httpd安装mod_ssl模块

### 直接编译mod_ssl为httpd的静态模块

编译httpd时在编译选项中指定ssl选项：

```bash
# 需要openssl-devel依赖(Centos)
#yum install openssl-devel -y

# 指定--enable-ssl --with-ssl 选项
# 在 httpd 源码目录中 configure 时，指定 --enable-ssl --with-ssl 选项
./configure --prefix=/usr/local/apache2 --enable-static-support --disable-status --enable-rewrite --enable-so --enable-ssl --with-ssl
make
make install
```

### 动态编译mod_ssl到httpd

```bash
# 在httpd 源码目录中：
cd modules/ssl
/usr/local/apache2/bin/apxs -i -a -D HAVE_OPENSSL=1 -I/usr/include/openssl/ -L/usr/lib/openssl/ -c *.c -lcrypto -lssl -ldl

# 编译成功后还需要在apache的httpd.conf中加载mod_ssl模块才能使用：
sed -i 's@#LoadModule ssl_module         modules/mod_ssl.so@LoadModule ssl_module         modules/mod_ssl.so@' httpd.conf
```

### 查看ssl_module是否正常：

```bash
bin/apachectl -l
# 在信息中可以看到mod_ssl.c
```

## (2)配置ssl启用https服务

**[ssl证书文件](https://vastxiao.github.io/article/2017/09/03/Linux/openSSl_for_CA_and_Certificate/)**

配置ssl需要先获得两个配对的文件：服务器ssl密钥文件(server.key)和CA颁发的证书(server.crt)。

**1. 在httpd.conf配置文件中引入ssl配置文件,增加支持ssl：**

```
# (去掉行首的注释)
Include conf/extra/httpd-ssl.conf
```

**2. 根据实际需求配置conf/extra/httpd-ssl.conf：**

### 可以直接修改文件中的默认的VirtualHost直接配置https站点。

```
# 如果配置报错：
# [warn] _default_ VirtualHost overlap on port 443, the first has precedence
# 需要添加这个配置：
NameVirtualHost *:443


####### apache2.2默认配置 #######
<VirtualHost _default_:443>
DocumentRoot "/usr/local/apache2/htdocs"
ServerName www.example.com:443
ServerAdmin you@example.com
ErrorLog "/usr/local/apache2/logs/error_log"
TransferLog "/usr/local/apache2/logs/access_log"
SSLEngine on
SLCipherSuite ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP:+eNULL
SSLCertificateFile "/usr/local/apache2/conf/server.crt"
SSLCertificateKeyFile "/usr/local/apache2/conf/server.key"
<FilesMatch "\.(cgi|shtml|phtml|php)$">
    SSLOptions +StdEnvVars
</FilesMatch>
<Directory "/usr/local/apache2/cgi-bin">
    SSLOptions +StdEnvVars
</Directory>
BrowserMatch ".*MSIE.*" \
         nokeepalive ssl-unclean-shutdown \
         downgrade-1.0 force-response-1.0
CustomLog "/usr/local/apache2/logs/ssl_request_log" \
          "%t %h %{SSL_PROTOCOL}x %{SSL_CIPHER}x \"%r\" %b"
</VirtualHost>


####### apache2.4默认配置 #######
<VirtualHost _default_:443>
DocumentRoot "/usr/local/apache4/htdocs"
ServerName www.example.com:443
ServerAdmin you@example.com
ErrorLog "/usr/local/apache4/logs/error_log"
TransferLog "/usr/local/apache4/logs/access_log"
SSLEngine on
SSLCertificateFile "/usr/local/apache4/conf/server.crt"
<FilesMatch "\.(cgi|shtml|phtml|php)$">
    SSLOptions +StdEnvVars
</FilesMatch>
<Directory "/usr/local/apache4/cgi-bin">
    SSLOptions +StdEnvVars
</Directory>
BrowserMatch "MSIE [2-5]" \
         nokeepalive ssl-unclean-shutdown \
         downgrade-1.0 force-response-1.0
CustomLog "/usr/local/apache4/logs/ssl_request_log" \
          "%t %h %{SSL_PROTOCOL}x %{SSL_CIPHER}x \"%r\" %b"
</VirtualHost>
```

### 或者先注释文件的VirtualHost配置，后自行配置在任何文件中。

先根据需要注释掉httpd-ssl.conf默认的VirtualHost配置：

```bash
cd conf/extra/httpd-ssl.conf
cp -a httpd-ssl.conf httpd-ssl.conf.default
line_n=(sed -n '/^<VirtualHost _default_:443>$/=' httpd-ssl.conf)
sed -i "${line?err},$s/^/#/g" httpd-ssl.conf
```

自己配置VirtualHost启动https web：

```
# 如果配置报错：
# [warn] _default_ VirtualHost overlap on port 443, the first has precedence
# 需要添加这个配置：
NameVirtualHost *:443


# 最简单的https配置：
<VirtualHost *:443>
        DocumentRoot /www/domain/wwwroot
        ServerName www.doamin.com
        SSLEngine on
        SSLCertificateFile /etc/httpd/conf/server.crt
        SSLCertificateKeyFile /etc/httpd/conf/server.key
</VirtualHost>


# 一般的https服务配置：
<VirtualHost *:443>
    SSLEngine on
    SSLProtocol all -SSLv2
    SSLCipherSuite HIGH:MEDIUM:!aNULL:!MD5
    SSLCertificateFile "/path/to/www.example.com/domain_server.crt"
    SSLCertificateKeyFile "/path/to/www.example.com/domain_server.key"
    <FilesMatch "\.(cgi|shtml|phtml|php)$">
        SSLOptions +StdEnvVars
    </FilesMatch>
    BrowserMatch "MSIE [2-5]" \
        nokeepalive ssl-unclean-shutdown \
        downgrade-1.0 force-response-1.0

    ServerAdmin webmaster@domain.com
    ServerName www.example.com
    DocumentRoot "/web/www.example.com/wwwroot"
    ErrorLog "/var/logs/www.example.com-error.log"
    CustomLog "|/usr/local/apache2/bin/rotatelogs /var/logs/www.example.com-access.log_%Y.%m.%d.%H.%M.%S 86400 480" combined
    <Directory "/web/www.example.com/wwwroot">
        SSLOptions +StdEnvVars
        Options FollowSymLinks
        AllowOverride All
        Order allow,deny
        Allow from all
    </Directory>
</VirtualHost> 
```

### 在一个httpd服务中配置多个https网站

配置https关于涉及到多个域名、泛域名解析、通配符SSL证书、单服务器/多服务器、IP、端口等方面的情况。

* 老式的SSL证书是一个证书一个站点一个IP的一一对应，但后来有了改进；
* 可以配置为一台服务器多个IP，分别对应不同的站点、不同的证书；
* 还可以配置为一台服务器一个IP，多个端口号对应不同的站点、不同的证书；
* 后来出现SNI（Server Name Indication服务器名称指示）技术，让https与http一样实现一台服务器多个虚拟站点，每个站点都可以对应不同的证书，无需多个IP、无需多个端口（全部都用https标准的端口号443），多个域名、泛域名都支持。
* 对SSL证书提供商，根据自己的需要及预算选择，如果自己的站点多，最好是选择支持多域名、通配符的证书，例如StartCom (link is external)的EV、OV、IV认证支持的证书（DV认证不支持通配符）；
* 如果需要设置多个SSL站点，在Apache 2.2以上版本中是开启SSL模块后是直接支持SNI的，添加NameVirtualHost *:443和SSLStrictSNIVHostCheck off两句后，就可以像http虚拟站点一样设置多个https虚拟站点；
* 多个https虚拟站点可以分别指向多个不同的证书文件，其中第一个默认https站点是在后续https站点配置找不到的时候自动使用的默认配置；
* https虚拟站点与http虚拟站点配置一样，可以使用ServerAlias来将多个子域名指向同一个目录、采用相同的SSL证书；

```
Listen 443
#Listen 8081
NameVirtualHost *:443
SSLStrictSNIVHostCheck off

<VirtualHost _default_:443>
DocumentRoot "/usr/local/apache/htdocs/example.com"
ServerName example.com
ServerAlias subdomain.example.com
ServerAdmin you@example.com
SSLEngine on
SSLProtocol all -SSLv2
SSLCipherSuite HIGH:MEDIUM:!aNULL:!MD5
SSLCertificateFile "/usr/local/apache/conf/server.crt"
SSLCertificateKeyFile "/usr/local/apache/conf/server.key"
SSLCertificateChainFile "/usr/local/apache/conf/1_root_bundle.crt"
<FilesMatch "\.(cgi|shtml|phtml|php)$">
    SSLOptions +StdEnvVars
</FilesMatch>
<Directory "/usr/local/apache/htdocs/example.com">
    AllowOverride All
    SSLOptions +StdEnvVars
</Directory>
</VirtualHost>

<VirtualHost *:443>
DocumentRoot "/usr/local/apache/htdocs/example2.com"
ServerName example2.com
ServerAlias subdomain.example2.com
SSLEngine on
SSLProtocol all -SSLv2
SSLCipherSuite HIGH:MEDIUM:!aNULL:!MD5
SSLCertificateFile "/usr/local/apache/conf/server2.crt"
SSLCertificateKeyFile "/usr/local/apache/conf/server2.key"
SSLCertificateChainFile "/usr/local/apache/conf/1_root_bundle2.crt"
<FilesMatch "\.(cgi|shtml|phtml|php)$">
    SSLOptions +StdEnvVars
</FilesMatch>
<Directory "/usr/local/apache/htdocs/example2.com">
    AllowOverride All
    SSLOptions +StdEnvVars
</Directory>
</VirtualHost>
```

[apache mod_gnutls实现多HTTPS虚拟主机](http://www.ttlsa.com/apache/multi-https-virtual-host-apache-mod_gnutls/)

## (3)http协议重定向(可选)

可以让用户HTTP访问自动重定向为HTTPS。

### 可以直接在全局生效配置

```
# 如http.conf/httpd-ssl.conf文件中加入301永久重定向配置：
RewriteEngine onRewriteCond %{SERVER_PORT} !^443$RewriteRule ^/?(.*)$ https://%{SERVER_NAME}/$1 [L,R]
```

### 在VirtualHost生效配置

```
<VirtualHost *:80>
    ServerAdmin webadmin@domain.com
    ServerName www.domain.com

    RewriteEngine On
    RewriteRule ^/(.*)$ https://www.domain.com/$1 [R=301,L]
</VirtualHost>
```

# 参考

* http://lookingdream.blog.51cto.com/5177800/1919718
* http://blog.csdn.net/ownfire/article/details/7686746 
* https://jamesqi.com/%E5%8D%9A%E5%AE%A2/https%E5%A4%9A%E7%BD%91%E7%AB%991%E4%B8%AAIP%E5%A4%9A%E4%B8%AASSL%E8%AF%81%E4%B9%A6%E7%9A%84Apache%E8%AE%BE%E7%BD%AE%E5%8A%9E%E6%B3%95


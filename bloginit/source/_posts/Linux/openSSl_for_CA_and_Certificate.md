---
title: 基于OpenSSL的CA服务和SSL/TLS证书颁发
description: 本文是关于OpenSSL的CA服务和SSL/TLS证书颁发机制的服务器环境搭建和证书颁发过程。
toc: true
categories:
  - Linux
tags:
  - openssl
  - CA
  - SSL/TLS
  - service
date: 2017-09-03 20:14:32
updated: 2017-10-08 14:30:00
---

# 基于OpenSSL的CA服务和SSL/TLS证书颁发

## OpenSSL, SSL/TLS, CA Certificate

**OpenSSL**

openssl是一个开源程序的套件、这个套件有三个部分组成：

* libcryto，这是一个具有通用功能的加密库，里面实现了众多的加密库
* libssl，这个是实现ssl机制的，它是用于实现TLS/SSL的功能
* openssl，是个多功能linux命令行工具，它可以实现加密解密，还可以当CA来用，可以管理证书。

**SSL/TLS**

SSL/TLS原理详解: http://seanlook.com/2015/01/07/tls-ssl/

**CA Certificate**

CA和数字证书: http://seanlook.com/2015/01/15/openssl-certificate-encryption/

## Linux上基于OpenSSL的CA证书服务系统

### CA证书认证服务器(证书认证中心)

默认情况和CentOS和Ubuntu上都已安装好openssl。

CentOS 6.x上有关ssl证书默认的目录结构：

```
/etc/ssl/
     └── certs -> /etc/pki/tls/certs
/etc/pki/
     ├── CA/               # 本服务器上搭建的CA认证中心/目录。
     │   ├── certs/
     │   ├── crl/          # 吊销的证书。
     │   ├── newcerts/     # 存放CA签署（颁发）过的数字证书（证书备份目录）。
     │   └── private/      # 用于存放CA的私钥。
     ├── ca-trust/         # 存放系统信任的CA机构证书。
     └── tls/
         ├── cert.pem -> certs/ca-bundle.crt
         ├── certs/        # 系统的证书存放目录，可以放置系统自己的证书和内置证书。
         │   └── ca-bundle.crt    # Mozilla root CA 生成的X.509 CA公钥证书。
         ├── openssl.cnf   # openssl的CA主配置文件。
         └── private/      # 系统的证书私钥目录，存放系统使用的证书私钥。
```

Ubuntu的CA位于/etc/ssl/demoCA/

### 创建CA证书认证服务(Centos)

**(1)修改openssl的CA的配置文件**

CA要给别人颁发证书，首先自己得有一个作为根证书，
要在一切工作之前修改好CA的配置文件、序列号、索引等等。

/etc/pki/tls/openssl.cnf

```
# 对于自建的非权威CA认证中心，这个文件配置大部分配置使用配置比较简单。
# 这里只有配置文件的部分内容。

[ ca ]
default_ca      = CA_default            # The default ca section

# ca证书/目录相关配置
[ CA_default ]
dir             = /etc/pki/CA           # Where everything is kept
certs           = $dir/certs            # Where the issued certs are kept
crl_dir         = $dir/crl              # Where the issued crl are kept
database        = $dir/index.txt        # database index file.
#unique_subject = no                    # Set to 'no' to allow creation of
                                        # several ctificates with same subject.
new_certs_dir   = $dir/newcerts         # default place for new certs.
certificate     = $dir/cacert.pem       # The CA certificate
serial          = $dir/serial           # The current serial number
crlnumber       = $dir/crlnumber        # the current crl number
                                        # must be commented out to leave a V1 CRL
crl             = $dir/crl.pem          # The current CRL
private_key     = $dir/private/cakey.pem# The private key
RANDFILE        = $dir/private/.rand    # private random number file
x509_extensions = usr_cert              # The extentions to add to the cert
name_opt        = ca_default            # Subject Name options
cert_opt        = ca_default            # Certificate field options

# 颁发的证书默认有效时间配置
default_days    = 5365                  # how long to certify for
default_crl_days= 30                    # how long before next CRL
default_md      = default               # use public key default MD
preserve        = no                    # keep passed DN ordering

# A few difference way of specifying how similar the request should look
# For type CA, the listed attributes must be the same, and the optional
# and supplied fields are just that :-)
policy          = policy_match

# For the CA policy
[ policy_match ]
countryName             = match
stateOrProvinceName     = match
organizationName        = match
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional
# For the 'anything' policy
# At this point in time, you must list all acceptable 'object'
# types.
[ policy_anything ]
countryName             = optional
stateOrProvinceName     = optional
localityName            = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

# 使用openssl生成证书时的一些证书信息默认值。
# 这个文件基本上配置这个就可以了，方便生成证书时的信息输入。
[ req_distinguished_name ]
countryName                     = Country Name (2 letter code)
countryName_default             = XX
countryName_min                 = 2
countryName_max                 = 2
stateOrProvinceName             = State or Province Name (full name)
#stateOrProvinceName_default    = Default Province
localityName                    = Locality Name (eg, city)
localityName_default    = Default City
0.organizationName              = Organization Name (eg, company)
0.organizationName_default      = Default Company Ltd
# we can do this but it is not needed normally :-)
#1.organizationName             = Second Organization Name (eg, company)
#1.organizationName_default     = World Wide Web Pty Ltd
organizationalUnitName          = Organizational Unit Name (eg, section)
#organizationalUnitName_default =
commonName                      = Common Name (eg, your name or your server\'s hostname)
commonName_max                  = 64
emailAddress                    = Email Address
emailAddress_max                = 64
```

**(2)生成CA根密钥**

```bash
(umask 077;openssl genrsa -out /etc/pki/CA/private/cakey.pem 2048)
```

**(3)生成根证书**

使用openssl req命令生成自签证书：

```bash
(umask 077;openssl req -new -x509 -key /etc/pki/CA/private/cakey.pem -out /etc/pki/CA/cacert.pem)
```

1. 会提示输入一些内容（之前修改的openssl.cnf会在这里呈现），最好记住能与后面保持一致。
2. 注意输入的内容，Common Name选项，需要填写CA服务器对应的IP或域名。
3. 生成的这个cacert.pem证书文件，可以导入到客户端(用户)系统CA信任证书中，或客户端浏览器信任证书中。

**(4)初始化CA的/目录文件**

```bash
touch /etc/pki/CA/newcerts
touch /etc/pki/CA/index.txt
touch /etc/pki/CA/serial
echo "01" > /etc/pki/CA/serial
```

_到此一个CA认证服务中心就创建完成了。_

### 应用服务端(服务器)生成证书签署请求文件

 应用服务端使用HTTPS或基于ssl/TLS应用需要到CA签署证书文件。

**(1)应用服务端(服务器)生成ssl密钥**

```bash
openssl genrsa -out server.key 2048
```
_生成的key文件为应用服务端(服务器)私有的，需要注意保存。_

**(2)应用服务端(服务器)生成证书签署请求文件**

```bash
openssl req -new -key server.key -out server.csr
```

1. 同样会提示输入一些内容。除了Commone Name一定要是你要授予证书的服务器域名或主机名，challenge password不填。
2. 注意输入的内容，Common Name选项，需要填写需要授予证书的服务器的域名或IP地址，challenge password不填。
3. 生成的csr文件只对CA签署证书有用，提供此文件CA就能根据信息完成证书签署。

### CA根据应用服务端(服务器)的请求签署证书并颁发证书文件

```bash
# 生成的crt文件就是应用服务端(服务器)的证书文件，要将文件提供给应用服务端(服务器)。
openssl ca -in server.csr -ourt server.crt

# 指定证书有效天数：
openssl ca -in server.csr -ourt server.crt -days 400
```

**注：**

应用服务端(服务器)最后使用部署ssl服务时用到的文件是两个：

* 应用服务端(服务器)生成ssl密钥key文件(server.key)。
* CA签署后颁发的证书crt文件(server.crt)。

应用服务器拿这两个文件部署服务。

# 资料参考

* http://seanlook.com/2015/01/18/openssl-self-sign-ca/
* [申请免费的SSL证书](https://jamesqi.com/%E5%8D%9A%E5%AE%A2/%E7%94%B3%E8%AF%B7%E5%85%8D%E8%B4%B9%E7%9A%84SSL%E8%AF%81%E4%B9%A6%EF%BC%8C%E5%BC%80%E9%80%9Ahttps%E7%BD%91%E7%AB%99)
* [关于SNI: 实现多域名虚拟主机的SSL/TLS认证](http://www.ttlsa.com/web/sni-multi-domain-virtual-host-ssl-tls-authentication/)


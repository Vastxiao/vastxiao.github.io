---
title: 关于linux上postfix的maildrop目录中存在大量小文件问题的最好解决办法
description: 解决linux邮件系统postfix中存在的大量maildrop文件问题。
toc: true
categories:
  - Linux
tags:
  - QA
  - linux
date: 2017-08-21 21:50:20
updated: 2017-08-21 21:50:20
---

# maildrop目录中存在大量文件的原因及解决方法

在/var/spool/postfix/maildrop/ 中发现了大量的文件, 分析原因：

由于linux在执行cron时，会将cron执行脚本中的output和warning信息以邮件的形式发送给cron所有者，
而如果服务器中关闭了postfix将导致邮件发送不成功，全部小文件堆积在了maildrop目录下面，
如果sendmail或者postfix正常运行，则会在/var/mail目录下也会堆积大量的邮件。

__解决方法：__

```
修改“/etc/crontab”文件，将‘MAILTO=root’替换成‘MAILTO=""
修改之后需要重启crond服务才生效/etc/init.d/crond restart

也可以在crontab（crontab -e）中最前面直接加入MAILTO=""
```

删除文件提示文件太多，解决办法(这个方式删除较慢，但占用系统资源少，不影响线上机器性能):

```sh
/bin/find /var/spool/postfix/maildrop -type f -exec rm -vf {} \;
```

# 参考资料

* http://blog.chinaunix.net/uid-26364035-id-3163574.html  
* http://www.shangxueba.com/jingyan/121368.html


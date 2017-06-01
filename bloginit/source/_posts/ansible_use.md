---
title: ansible配置和使用
description: 自动化工具ansible的基本配置和用法。
toc: true
categories:
  - linux
tags:
  - tools
  - ansible
date: 2017-06-1 22:46:09
updated: 2017-06-1 22:46:09
---

# 配置和使用ansible

## 配置文件

### 应用程序的主配置文件

Ansible的一些的设置可以通过配置文件完成.

用户可以修改一下配置文件来修改设置,他们的被读取的顺序如下:

* ANSIBLE_CONFIG (一个环境变量)
* ansible.cfg (位于当前目录中)
* .ansible.cfg (位于家目录中)
* /etc/ansible/ansible.cfg (默认全局配置文件)

配置形式(ini格式)：

```
[defaults]
host_key_checking = False
[ssh_connection]
scp_if_ssh = smart
```

配置参考：http://www.ansible.com.cn/docs/intro_configuration.html

### Host Inventory(定义管控主机)

参考：http://www.ansible.com.cn/docs/intro_inventory.html

Inventory文件，用于配置节点信息，使用ansible时，
指定Patterns，根据Inventory配置来操作远程机器。

#### 单独使用Inventory配置文件

**默认配置文件：**
/etc/ansible/hosts

```
# 未分组
mail.example.com
badwolf.example.com:5309

[dbservers]
one.example.com
two.example.com

[webservers]
www[01:50].example.com

# 以下方式单独设置主机变量，这些变量定义后可在playbooks中使用
[targets]
localhost              ansible_connection=local
other1.example.com     ansible_connection=ssh        ansible_ssh_user=mpdehaan
other2.example.com     ansible_connection=ssh        ansible_ssh_user=mdehaan
host1 http_port=80 maxRequestsPerChild=808

# 也可以定义属于整个组的变量
[databases]
db-[a:f].example.com
[databases:vars]
ntp_server=ntp.atlanta.example.com
proxy=proxy.atlanta.example.com

# 可以把一个组作为另一个组的子成员,以及分配变量给整个组使用.
# 这些变量可以给ansible-playbook使用,但不能给ansible使用
[atlanta]
host1
host2
[raleigh]
host2
host3
[southeast:children]
atlanta
raleigh
[southeast:vars]
some_server=foo.southeast.example.com
halon_system_timeout=30
self_destruct_countdown=60
escape_pods=2
[usa:children]
southeast
northeast
```

**可以手动指定Inventory配置文件：**

```
ansible -i /PATH/TO/HOSTS_File HOST-Patterns -m MODULE_Name -a "ARGS..."

ansible -i /tmp/myhosts.file all -m ping
```

#### 分文件定义Host和Group变量来配置Inventory

在inventory主文件中保存所有的变量并不是最佳的方式.
还可以保存在独立的文件中,这些独立文件与inventory文件保持关联.
不同于inventory文件(INI格式),这些独立文件的格式为YAML.

对主机的配置存放在host_vars/目录下。  
对组的配置存放在group_vars/目录下。

```
/etc/ansible/hosts
/etc/ansible/group_vars/raleigh
/etc/ansible/group_vars/webservers
/etc/ansible/host_vars/foosball
```

group_vars/和host_vars/目录可放在inventory目录下,或是playbook目录下.
如果两个目录下都存在,那么playbook目录下的配置会覆盖inventory目录的配置.

#### Patterns语法

Patterns用法参考：http://www.ansible.com.cn/docs/intro_patterns.html

```
# 支持通配，支持域名
all
*
one.example.com
one.example.com:two.example.com
192.168.1.50
192.168.1.*

# group组
webservers
webservers[0]
webservers[0-25]

# 多个组，相当于或集
webservers:dbservers
# 排除其他组！，非集
webservers:!phoenix
# 主机必须同时存在组，并集
webservers:&staging
webservers:dbservers:&staging:!phoenix

# 正则
~(web|db).*\.example\.com
```

## ansible使用

Ansible提供两种方式去完成任务,一是ad-hoc命令,一是写Ansible playbook.
前者可以解决一些简单的任务, 后者解决较复杂的任务.

* 使用ad-hoc，既使用/usr/bin/ansible这个命令的意思。
* 使用playbook，既使用/usr/bin/ansible-playbook这个命令的意思。

比如说因为圣诞节要来了,想要把所有实验室的电源关闭,我们只需要执行一行命令就可以达成这个任务,而不需要写playbook来做这个任务.
至于说做配置管理或部署这种事,还是要借助 playbook 来完成,即使用 ‘/usr/bin/ansible-playbook’ 这个命令.

使用ansible推荐：
* 通讯方式使用ssh，sftp，scp，及基于ssh的分通讯方式
* 使用ssh启用启用公钥认证方式
* 使用ssh-agent管理私钥

### 使用Ad-Hoc管理简单任务

参考：http://www.ansible.com.cn/docs/intro_adhoc.html

```
# 最常使用命令
ansible [-i host-file] <host-pattern> [-f forks] [-m module_name] [-a args]
  -i host-file     指定host-file替代/etc/ansible/hosts
  host-pattern     指定Patterns参数，匹配需要执行操作的host
  -f forks         指明每批管控forks数量的主机，默认为5个主机一批次
  -m module_name   使用何种模块管理操作，所有的操作都需要通过模块来指定，默认command
  -a args          指明模块专用参数。args一般为key=value格式；而command模块的参数非为kv格式，而是直接给出要执行的命令即可。
```

### 使用Playbook管理复杂任务

参考：http://www.ansible.com.cn/docs/playbooks.html

对于需反复执行的、较为复杂的任务，可以通过定义Playbook来搞定。
Playbook是Ansible真正强大的地方，它允许使用变量、条件、循环、以及模板，
也能通过角色及包含指令来重用既有内容。

该例子在远端机器上创建一个新的用户：

user.yml
```
---  
- name: create user  
hosts: vps  
user: root  
gather\_facts: false
 
vars:  
- user: "toy"
 
tasks:  
- name: create {{ user }} on vps  
user: name="{{ user }}"
```

首先，我们给Playbook指定了一个名称；接着，通过hosts让该Playbook仅作用于vps组；
user指定以root帐号执行，Ansible也支持普通用户以sudo方式执行；
gather\_facts的作用是搜集远端机器的相关信息，稍后可通过变量形式在Playbook中使用；
vars定义变量，也可单独放在文件中；tasks指定要执行的任务。

要执行Playbook，可以敲入：

```sh
ansible-playbook user.yml
```



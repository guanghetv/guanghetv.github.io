---
layout: post
title: "搭建Netatalk Server"
category: tech
author: diggzhang

---

## 简介

**Netatalk**是AppleTalk的开源实现方案，我们的服务器目前针对windows用户提供samba文件共享服务，针对mac用户提供afp（AppleShare file server):

* Windows用户使用smb://10.8.3.X
* Apple用户使用afp://10.8.3.X 


## NetaTalk Download
Ubuntu似乎已经提供了[Netatalk的包](https://launchpad.net/ubuntu/+source/netatalk).但是我们的生产环境是Centos7,Netatalk 3.1.7的rpm包均为Failed状态，需要手工编译。首先下载编译包：

* [netatalk-3.1.7-0.1.fc22.src.rpm](http://www003.upp.so-net.ne.jp/hat/files/netatalk-3.1.7-0.1.fc22.src.rpm)

## Build
下载基础构建工具

```
	# yum install rpm-build gcc make

```

安装下载好的src RPM

```
	$ rpm -ivh netatalk-X.X.X-XXX.XXX.src.rpm

```

安装后会提示，没有hat用户以及用户组，不会影响后续工作

```
	warning: group hat does not exist - using root
	warning: user hat does not exist - using root
```

安装完成后会在home目录下创建一个**rpmbuild**文件夹，切到指定目录执行rpmbuild

```
	$ cd ~/rpmbuild/SPECS/
	$ rpmbuild -bb netatalk.spec
```

rpmbuild时会提示缺乏各种依赖关系，需要一一对应下载这些关联包

```
	$ sudo yum install -y dbus-devel dbus-glib-devel docbook-style-xsl flex libacl-devel libattr-devel libdb-devel libevent-devel libgcrypt-devel krb5-devel mysql-devel openldap-devel openssl-devel pam-devel quota-devel systemtap-sdt-devel tcp_wrappers-devel libtdb-devel tracker-devel

```

## Install

 编译完成后，构建好的包位置在 *~/rpmbuild/RPMS/XXX*
 
 ``` 
 	$ cd ~/rpmbuild/RPMS/XXX/
 ```
 
 初次安装使用“-ivh”
 
 ```
 	# rpm -ivh netatalk-X.X.X-XXX.XXX.XXX.rpm
 ```
 
 如果已经安装过，使用"-Uvh"
 
 ```
  # rpm -Uvh netatalk-X.X.X-XXX.XXX.XXX.rpm

 ```
 至此，安装完成 :D
 
 
## Check
检查Netatalk是否被正确安装

```
$ /usr/sbin/afpd -V
afpd 3.1.7 - Apple Filing Protocol (AFP) daemon of Netatalk

This program is free software; you can redistribute it and/or modify it under
the terms of the GNU General Public License as published by the Free Software
Foundation; either version 2 of the License, or (at your option) any later
version. Please see the file COPYING for further information and details.

afpd has been compiled with support for these features:

AFP versions: 2.2 3.0 3.1 3.2 3.3 3.4 
CNID backends: dbd last tdb mysql 
Zeroconf support: Avahi
TCP wrappers support: Yes
Quota support: Yes
Admin group support: Yes
Valid shell checks: Yes
cracklib support: Yes
EA support: ad | sys
ACL support: Yes
LDAP support: Yes
D-Bus support: Yes
Spotlight support: Yes
DTrace probes: Yes

afp.conf: /etc/netatalk//afp.conf
extmap.conf: /etc/netatalk//extmap.conf
state directory: /var/netatalk/
afp_signature.conf: /var/netatalk/afp_signature.conf
afp_voluuid.conf: /var/netatalk/afp_voluuid.conf
UAM search path: /usr/lib64/netatalk//
Server messages path: /var/netatalk/msg/

```
## Setting Up
Netatalk的配置文件在*/etc/netatalk/afp.conf* 这里贴出一段可用配置，用户share登入后可以访问到自己的家目录

```
[Global]
  ; Global server settings
  log level = default:warn
  log file = /var/log/afpd.log
  ;hosts allow = 10.8.3.0/24

[Homes]
  basedir regex = /home

[share]
  path = /home/share
```

## Starting
```
	# systemctl enable netatalk
	# systemctl start netatalk
```
之后在Finder菜单栏中，前往->连接服务器->输入 afp://XX.XX.XX.XX 填写用户名密码就可以访问到了。

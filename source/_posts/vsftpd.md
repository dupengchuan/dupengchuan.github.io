layout: post
title: FTP服务器安装以及配置
date: 2016-05-27 18:41:18
tags: Linux
---
今天安装了一下FTP服务器，记录一下安装过程，以及遇到的问题

OS： CentOS6.5

vsftpd Version： 2.2.2

# 安装vsftpd #

    yum install vsftpd

<!--more-->

## 配置 ##

vaftpd的配置文件：/etc/vsftpd/vsftpd.conf

关掉匿名登录开启本地用户登录

    anonimous_enable=NO

    local_enable=YES

## 设置开机启动 ##

- 使用可交互的界面设置（空格选中）

      ntsysv

- 命令设置

      chkconfig vsftpd on   

## 启动vsftpd ##

    service vsftpd start

发现不能访问，可以ping通，说明不是网络的问题，telnet ip 21。telnet不上,应该是防火墙的原因

# 配置防火墙 #

allow all ftp incoming connections

    iptables -A INPUT -p tcp --dport 21 -m state --state ESTABLISHED -j ACCEPT

    iptables -A OUTPUT -p tcp --sport 21 -m state --state NEW,ESTABLISHED -j ACCEPT

Enable active ftp transfers

    iptables -A INPUT -p tcp --dport 20 -m state --state ESTABLISHED,RELATED -j ACCEPT

    iptables -A OUTPUT -p tcp --sport 20 -m state --state ESTABLISHED -j ACCEPT

Enable passive ftp transfers

    iptables -A INPUT -p tcp --sport 1024:65535 --dport 1024:65535 -m state --state ESTABLISHED -j ACCEPT

    iptables -A OUTPUT -p tcp --sport 1024:65535 --dport 1024:65535 -m state --state ESTABLISHED,RELATED -j ACCEPT

主动模式中，使用端口20发送数据，所以是对20端口配置

被动模式的端口号是动态申请的，所以指定的是一个范围

防火墙配置好之后记得保存规则

    service iptables save

重启服务，发现还是不能登录，报错如下：

    500 OOPS: cannot change directory:/home/*******

    500 OOPS: child died

一个权限的问题，google一下，发现是SELinux做了约束，需要修改相应配置

# 修改SELinux #

方法一、关闭SELinux

SELinux的启动配置文件是： /etc/selinux/config

    SELINUX=disabled

方法二、开启某些功能

网上有的人这么做

    setsebool -P ftpd_disable_trans on

但是我这里找不到 ftpd_disable_trans，可能是系统版本的问题，经过尝试，终于用下面命令搞定

    setsebool -P ftp_home_dir on

---
一些关于SELinux的命令

    setsebool xxx on/off　　＃设置状态

    getsetools -a　　＃结合grep可以看到有关ftp的约束

    sestatus　　＃查看selinux是否开启

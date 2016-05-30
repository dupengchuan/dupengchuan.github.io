layout: post
title: SSH远程登录
date: 2016-05-30 12:22:52
tags: Shell
---
SSH为Secure Shell的缩写，是位于传输层与应用程序之间的安全传输协议，能够保证信道的安全性。SSH在*uix上的一个重要应用就是远程登录。

SSH提供了两种验证方式。
1. 基于口令的安全验证，知道账号和口令，输入口令可以登录到远程主机，但是每次登陆都需要输入口令，无法避免“中间人”攻击。
2. 基于密钥的安全验证，客户机需要有自己的一对密钥，把公钥放在需要访问的服务服务器上。通过使用ssh-agent和ssh-add可以做到免口令登陆。

<!--more-->

# 基于口令的安全验证 #

当发送一个登陆命令时

    ssh user@host

1. 查询$HOME/.ssh/known_host里面是否有host对应的公钥
2. 如果找到host对应的公钥，不会有是否连接远程主机的提示，客户端用公钥加密口令发功给远程主机，远程主机解密，如果口令正确则登陆成功
3. 如果没有找到host对应的公钥，则向host请求公钥，之后登陆界面会显示远程主机的指纹，是否连接
4. 如果yes，则执行步骤2，并将远程主机公钥存入$HOME/.ssh/known_host文件中，供下次使用。

流程图如下:

![sorry](http://img2.ph.126.net/uWUDtGxI3xhg_EE_W3cpVg==/6631698885372855341.png)

# 基于密钥的安全验证 #

验证过程如下：

1. local host将自己的公钥发送给remote host。
2. remote host查询自己的authorized_keys文件，如果存在该公钥则给local host 发送一个chanllenge(用local host的公钥加密一个随机数)。
3. local host使用自己的私钥加密这个随机数发送给remote
4. remote检查随机数，确定验证公钥的合法性，给local host反馈结果。

流程图如下：

![sorry](http://img0.ph.126.net/nfQy6oBEaoTDnhcFBO8qZQ==/1993687260144251892.png)


# 基于密钥验证登录实践 #

步骤很简单

1. 生成本地公私钥对
2. 本地公钥添加到remot的$HOME/.ssh/authorized_keys文件中

## 本地生成密钥对 ##

    ssh-keygen -t rsa

也可以为自己的私钥设置口令

    ssh-keygen -t rsa -P "xxxx"

自己的私钥设置口令之后每次登陆都需要输入自己的口令，使用ssh-agent和ssh-add可以做到把口令的放入代理不用自己每次输入。

## 添加公钥到remote ##

方法1

    $ssh-copy-id user@host

tips:

远程配置文件/etc/ssd/sshd_config去掉以下注释，然后重启ssh服务

    #RSAAuthentication yes
    #PublickeyAuthentication yes
    #AuthorizedKeyFile .ssh/authorized_keys

方法2

    ssh user@host 'mkdir -p .ssh && cat >> .ssh/authorized_keys' < ~/.ssh/id_rsa.pub

方法3

  scp id_rsa.pub user@host:.ssh/
  cat id_rsa.pub >> authorized_keys


## 设置免口令登陆 ##

     ssh-add $HOME/.ssh/id_rsa

 输入自己的口令，之后登陆就交给代理处理，不需要输入口令

遇到的问题：

Could not open a connection to your authentication agent

通过下面命令解决了

    ssh-agent bash

不用之后一定要清除，否则别人用你的电脑一样可以免口令登陆

    ssh-agent -k

查看私钥指纹

    ssh-add -l

---
权限问题可能会造成不是自己想要的结果，比如id_rsa文件group和other如果有了写权限，基于密钥的验证就会降级为基于口令,设置.ssh目录的最小权限

    chmod 700 .ssh
    chmod 600 .ssh/id_rsa

SSH总的流程如下：

![sorry](http://img1.ph.126.net/2wPg2IDX5YSLWin_EGsRJw==/6631745064861226711.jpg)

参考文章：

[SSH原理与运用（一）：远程登录](http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html)

[使用ssh公钥密钥自动登陆linux服务器](http://7056824.blog.51cto.com/69854/403669/)

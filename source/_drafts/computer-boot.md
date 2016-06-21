layout: draft
title: 计算机启动过程详解
date: 2016-05-31 14:22:27
tags: Arch
---
一直对一个问题很感兴趣，那就是一台计算机究竟是怎么启动起来的呢？从按下电源键到操作系统启动，这之间发生了什么？

计算机是人类发明出来的工具，它的启动遵循着很多认为的规定，这些规定就像和计算机商量好的协议，当按下开机键的时候，计算机按要求首先执行BIOS固件里面的程序，当完成开机自检之后，从RAM地址0x7f00位置去读主引导记录，[为什么不是0xffff?](http://www.ruanyifeng.com/blog/2015/09/0x7c00.html)，因为这一切都是套路。

计算机的启动可以分两个阶段

1. BIOS开机自检（POST）
2. 加载内核镜像，并执行内核程序

<!--more-->

# 主引导记录 #

计算机启动的时候，[主引导记录](https://zh.wikipedia.org/wiki/%E4%B8%BB%E5%BC%95%E5%AF%BC%E8%AE%B0%E5%BD%95)被存入内存地址0x7c00，它是磁盘的第一个扇区，大小为512B,由三部分组成：

1. 引导程序区
2. 分区表
3. 结束标志

如下图所示：

![](http://img1.ph.126.net/zjlrB_4XscPzzpsk8Yzdrg==/2003820359305388956.png)

分区表由64个字节组成，每个分区由16个字节表示，所以最多有4个分区

分区表项内容以及含义：

|字节|内容以及含义|
|-|-|
|0|引导标志。80H表示活动分区，00表示非活动分区|
|1、2、3|本分区起始位置，通过（磁头号，扇区号，柱面号）三维坐标表示|
|5|分区类型符|
|5、6、7|本分区的结束位置，表示同起始位置|
|8、9、A、B|本分区之前已经用了的扇区数|
|C、D、E、F|本分区的总扇区数|

---
参考资料：

[为什么主引导记录的内存地址是0x7C00？](http://www.ruanyifeng.com/blog/2015/09/0x7c00.html)

[计算机是如何启动的？](http://www.ruanyifeng.com/blog/2013/02/booting.html)

[Linux引导过程内幕](https://www.ibm.com/developerworks/cn/linux/l-linuxboot/)

[硬盘概念：磁盘、磁头、扇区、柱面、簇](http://blog.csdn.net/xiaominthere/article/details/19756551)

[grub源码分析](http://jiaoshuaihit.blog.163.com/blog/static/260615942007611111056944/)

[Linux文件系统的实现](http://www.cnblogs.com/vamei/p/3506566.html)

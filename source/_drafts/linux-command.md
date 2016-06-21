layout: draft
title: shell-command
date: 2016-05-28 21:21:50
tags:
---
|命令|功能|Demo|Descrip|
|-|-|-|-|-|
| time | 查看脚本执行时间 |time 解释器 ./xxx|time不能执行程序，放在执行命令之前|


## 进程操作 ##

|命令|功能|Demo|Descrip|
|-|-|-|-|-|
|pidof|查看进程pid| pidof init|根据名字查看pid|
|pstree|查看进程树|pstree -p|显示pid|
|&|进程后台运行|ping localhost &|-|

|字段|说明|
|-|-|
|pid|进程id|
|ppid|父进程id|
|pgid|进程组|
|comm|进程名|
|cmd|进程名带进程参数|
|sid|session id|
|tty|终端|

进程内存映像： /proc/pid/maps

# 流操作 #
|命令|功能|Demo|Decrip|
|-|-|
|>|重定向标准输出|-|-|
|>>|重定向标准输出(追加模式)|-|-|
|2>|重定向标准错误|-|-|
|>&|重定向标准输出和标准错误|-|-|
|<|重定向标准输入|cat < txt|将cat的标准输入流重定向到txt文件|
|sort|对流排序|sort -nr|对流按照数字排序逆序输出|
|wc|对流统计|wc -cmwl|统计字节，字符，word以及行数|


# 文件系统 #

|命令|功能|Demo|Decrip|
|-|-|
|blkid|查看分区信息|blkid dev/sda1|查看UUID,文件系统类型|
|df|查看分区资源使用情况|df -i|inode使用情况|
|stat|查看文件状态|stat xxx | 查看文件某文件inode信息|
|dumpe2fs|查看分区扇区分配情况|dumpe2fs -h /dev/sda1|查看sda1超级块内容|
|fidsk|分区表操作|fdisk -l /dev/sda|查看sda的分区表|

# 其他 #

|命令|功能|Demo|Decrip|
|-|-|
|lspci|查看pci设备|-|-|

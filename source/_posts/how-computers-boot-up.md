layout: post
title: 计算机是如何启动的？
date: 2016-06-02 13:53:30
tags: Arch
---
>本文来自[Gustavo Duarte的博客](http://duartes.org/gustavo/blog/)，本人出于兴趣翻译过来，原文地址：[How Computer Boot up](http://duartes.org/gustavo/blog/post/how-computers-boot-up/)

前面的一篇博文讲述了Intel计算机的[主板芯片结构以及内存映射](http://dupengchuan.me/2016/05/31/motherboard-chipsets-and-the-memory-map/)，这些属于为计算机启动的初始化阶段，计算机启动是一个复杂多阶段并且有趣的过程。下面是一个流程图：

<!--more-->
![bootProcess](http://img2.ph.126.net/_N-Mf1KiaxbqiX8V5WCyfg==/6631579038605486672.png)

当你按下开机键的时候，一切变开始运转了。主板一旦被供电，它就初始化自己的固件（芯片组和其他附属组件）并启动CPU。如果此时硬件失败（比如CPU故障或者不存在），那么你的系统很有可能除了那旋转的风扇什么反应都没有。一些主板可能会发出蜂鸣告警CPU故障或者缺失，但据我的经验，风扇故障是最常见的情况。甚至有时候USB或者其它也设备能够引起这种情况发生，这时候拔下一些不必要的设备是一个解决办法。你可以通过逐步移除的方法来定位故障设备。

如果一切顺利，CPU开始运行。在多处理器或者多核的系统，一个CPU核心被动态的选择成为引导处理器（BSP），然后运行所有的BIOS和kernel初始化代码。其余的处理器称为应用处理器（AP），这时候它们依旧停留在原来的状态直到后来被内核激活。Intel的CPU已经发展了那么多年，但它们是完全向后兼容，所以现代的CPU和1978年的[intel 8086](http://en.wikipedia.org/wiki/Intel_8086)启动还是很相似，上电之后它们做哪些工作是很已经确定的。在上电后的早期状态，处理器在处在[实模式](http://en.wikipedia.org/wiki/Real_mode)（禁用内存[分页](http://en.wikipedia.org/wiki/Paging)）。这个就像早期的MS-DOS系统，可以寻址1MB内存空间，并且任何代码都可以存放在该空间的任何位置，这里没有内存保护和特权的概念。

CPU的大部分[寄存器](http://en.wikipedia.org/wiki/Processor_register)都有已经定义好的明确的功能，比如指令指针（EIP）寄存器，里面存储了CPU正在执行的指令的内存地址。Intel CPU使用基址寻址访问向量程序。即使上电时只有1MB的的内存可以被访问，一个隐藏的基址（本质上是一个偏移量）被加到EIP,因此第一条指令在0xFFFFFFF0(4G内存的最后16个字节，远远高于1M空间)。这个神奇的地址被称为[复位向量](http://en.wikipedia.org/wiki/Reset_vector)，是现代化CPU的一个标准。

主板确保在复位向量的跳转指令可跳转到存储器上BIOS程序的入口点。这个跳转指令也隐含清除了上电的时候产生的基址。所有的这些存储单元都有CPU需要的内容。这多亏了芯片（包括BIOS）的[内存映射](http://duartes.org/gustavo/blog/post/motherboard-chipsets-memory-map)机制。此时RAM模块在这些区域之间也存在没有被使用的内存。内存区域相关的例子如下图所示

![bootMemoryRegions](http://img1.ph.126.net/nUyi_jsfkqNY5i1PwMwA9g==/4939322891419790476.png)

CPU跳转到BIOS程序并执行，初始化一些硬件。之后BIOS开始进行[上电自检](http://en.wikipedia.org/wiki/Power_on_self_test)（POST），这个过程中检测各种计算机组件。如果缺少显卡会引发BIOS程序停止，并且发出蜂鸣警告，这样你就知道哪里出了问题（由于显卡出了问题，所以不可能从屏幕获得信息）。如果显卡正常运行，我们可能感觉计算机好像是正常的：制造商的商标打印了，内存也开始检测。其他的自检失败，像缺少键盘将会导致BIOS停止并将错误信息显示在屏幕上。POST包括硬件检测和初始化，POST之后会整理出所有资源-中断、内存范围、I/O端口（PCI设备）。现代的BIOS根据进行[高级配置和启动选项](http://en.wikipedia.org/wiki/ACPI)，建立一个启动表，里面保存了计算机上的设备，这个表稍后将会被内核用到。

POST之后，BIOS准备启动一个操作系统，首先要找到操作系统在哪里：硬盘，CD-ROM，软盘等等，这里，用户可以配置BIOS的选择启动设备的顺序。BIOS如果没有合适的启动设备，就会给出信息： “Non-System Disk Error”。一个坏掉的硬盘可能会导致这个问题。但愿不要这样，希望BIOS找到一个正常工作的硬盘顺利启动。

BIOS读取硬盘的第一个[扇区](http://en.wikipedia.org/wiki/Disk_sector)的512个字节。这个被称作[主引导记录](http://en.wikipedia.org/wiki/Master_boot_record)，通常它包括两部分：操作系统相关的引导程序和磁盘分区表。BIOS并不关心它们有什么用，BIOS只是仅仅将这个512字节加载到内存的0x7c00的位置，然后跳转到MBR中的引导程序继续执行。

![](http://img0.ph.126.net/c5QoVMKGUiOOubfIdSLraA==/6631565844465954720.png)

MBR中的引导程序可以是Windows的MBR loader,或者是Linux loader比如LILO或者GRUB,或者甚至有可能是一个病毒。相比较引导程序，分区表则是标准化的格式：它总共长64个字节，16个字节表示一个分区（所以在一个磁盘上可以有不同分区也可以有不同的系统）。通常，Microsoft MBR代码会查看分区表，找到那个被标记为活动状态的分区，加载该分区上引导扇区，并运行上面的代码。引导扇区是一个分区的第一个扇区，注意和磁盘的第一个扇区的区别不一样。如果分区表上有错误，你会看到：“无效的分区表”或者“找不到操作系统”。这些消息不是来自BIOS，而是来自MBR上的引导程序。至于消息的内容与特定的MBR引导程序有关。

随着时代的发展，引导程序已经变得越来越复杂和灵活。Linux引导程序LILO和GRUB可以引导多种操作系统、文件系统而且可以被配置。它们的MBR代码已经不是必须引导上面所说的活动分区。功能上的流程如下：

MBR包括boot loader的第一阶段，GRUB就在这个阶段。

由于MBR空间有限，MBR没有足够的空间加载从磁盘加载包含其他引导代码的扇区。这个扇区有可能是这个分区的引导扇区，但是也可以被硬编码到MBR代码区。

MBR代码加上那些在第二步加载代码，然后读取一个包含bootloader第二阶段代码的文件。GRUB中是grub/stage2,Windows中是c:\NTLDR。如果步骤2失败，Windows系统你会得到像：“NTLDR is missing”。阶段2的代码读取一个启动配置文件（linux中是grub.conf，windows中是boot.ini）。然后展示用户选择启动系统的界面或者是直接启动系统（仅仅存在一个启动项）

此时，bootloader代买需要启动一个内核。它必须知道文件系统是什么然后才可以去分区读取内核。在Linux中，这意味着去读取名字差不多是“vmlinuz-2.6.22-14-server”的文件，加载进内存并且跳转到内核启动代码。在WIndows2003中，一些内核启动代码和内核镜像是分开的。实际上嵌入到了NYLDR文件中。进行几个初始化之后，NTDLR才加载镜像c:\Windows\System32\ntoskrnl.exe。然后就想GRUB做的一样，跳转到内核的程序执行。

这里有一个值得一提的问题（）。当前的Linux 内核镜像即使经过压缩，也不能放进RAM最开始地那640K地址空间(实模式下可访问)。我的Ubuntu内核压缩之后是1.7M。但是bootloader通过BIOS读取磁盘数据必须运行在实模式中，因此内核显然是获取不到的。解决办法就是引入了[虚拟模式](http://en.wikipedia.org/wiki/Unreal_mode)。这个并不是一个真正的处理器模式（我希望Intel的工程师被允许在这方面有兴趣），但是是一项技术，程序为了能够通过BIOS例程访问超过1MB以上的地址空间在实模式和保护模式之间切换。如果你读过GRUB源码，你会看到这些转换（看stage2中的real_to_prot和prot_to_real）。当BIOS将内核加载到内存中，最后处理器又会切换回实模式。

正如图1中展示的，我们现在处在从“引导加载程序”到“早期的内核初始化”阶段。接下来的事情会变得麻烦，内核解压，动态初始化。下一篇文章是Linux内核初始化的一个介绍，我会给出源码的连接。当然Windows我不能这样做。但是我会指出值得注意的地方。

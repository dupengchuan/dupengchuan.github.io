layout: post
title: 主板芯片和内存映射原理
tags: Arch
date: 2016-05-31 16:37:10
---


>这篇博文翻译自[Gustavo Duarte](http://duartes.org/gustavo/blog/)博客上的一篇博文，笔者出于兴趣，同时也是学习，翻译过来，原文： [Motherboard Chipsets and the Memory Map](http://duartes.org/gustavo/blog/post/motherboard-chipsets-memory-map/)

为了阐明现在操作系统内核如何工作，我打算写一些关于计算机内部实现原理的文章。希望对那些对这方面有兴趣但是却没有经验的电脑爱好者和程序员们有所帮助。计算机的内部实现是我的一个爱好，因此我们主要关注点在Linux、Windows操作系统以及Intel处理器。这篇博文主要讲述了基于Intel处理器的主板布局，CPU怎么样访问内存以及系统内存映射。

<!--more-->

首先让我们看一下现代基于Intel处理器的计算机架构是什么样子的。下面这幅图主要展示了主板上的主要组件：

![](http://img2.ph.126.net/syfL7tvt2FaivsIy765tJg==/4819414551340637687.png)

现代主板由南桥芯片组和北桥芯片组构成

看到这幅图，记住最关键一点，那就是CPU不知道他连接的是什么设备。CPU通过引脚连接外面的世界，但是它不关心连接的是什么。虽然可能是一台计算机的主板，但是也有可能是烤面包机，网络路由器，CPU实验板等等。CPU有三种方式和外面通信：内存地址、IO地址和中断。我们现在只讨论主板和内存。

CPU通往外面的大门是连接到北桥的前端总线，CPU通过前端总线读写内存。它使用一些引脚传送它想要读或者写的物理内存地址，使用另外一些引脚传送或者接收数据。Intel Core2 QX6600有33根传送内存地址的引脚（地址空间2^33个内存地址）和64根发送和接收数据的引脚（数据宽度是64bit或者8字节）。这允许CPU能够寻址的内存空间大小为64GB（2^33 * 8 byte）尽管大部分的芯片只使用了8G的内存。

我们曾经认为程序不停地读取和写入的只有RAM,事实上大部分内存请求也是经过北桥芯片到了RAM模块，但并不是全部。内存物理地址也可以映射到各种各样的主板上的设备上（这种通信被称为[内存映射I/O](http://en.wikipedia.org/wiki/Memory-mapped_IO)）。这些设备有显卡、大部分PCI设备（比图，扫描仪或者SCSI卡）以及存储BIOS程序的[闪存](http://en.wikipedia.org/wiki/Flash_memory)。

当北桥芯片接收到一个内存请求，北桥芯片根据内存地址映射决定最终请求的设备是RAM还是显卡。对于每一个内存地址段，通过内存映射可以知道属于哪个设备。地址空间的大部分地址映射到了RAM,但是当如果没有内存映射告诉芯片哪个设备需要处理这些请求，那些不在RAM的地址空间就造成了内存地址空洞（640KB-1MB这部分地址是设备使用的但是如果没有设备使用就相当于这段内存没有用）。当内存地址为显卡和PCI设备预留了地址空间的时候，将会引起更大的空洞。这也就是[为什么32位的操作系统在使用4GRAM的时候会出问题](http://support.microsoft.com/kb/929605)。下面的这幅图展示了一个4G地址空间上的一个内存映射。


![memoryLayout](http://img2.ph.126.net/TMKxjy78Awc82dLBKfijVA==/6631513067907784718.png)

Intel系统前4G内存模型

实际的地址范围取决于具体的主板和实际的设备，但是大部分的Core 2系统和上面很类似。所有的褐色段被映射到RAM之外的设备。这些地址可以被主板总线直接使用。而在CPU内部（比如，运行的程序需要写）这些地址是逻辑地址，它们在被总线访问之前必须先被CPU转成物理地址。

逻辑地址转换为物理地址的过程比较复杂，依赖于CPU运行模式（实模式、32位保护模式和64位保护模式）。不考虑内存转换机制，CPU的模式决定了可以访问多大物理地址空间。比如，如果CPU运行在32位保护模式，那么它的寻址空间是4GB(但是开启[Physical Address Extension](http://en.wikipedia.org/wiki/Physical_address_extension)是个例外，这里暂不讨论)。由于最前面的大约1GB内存映射给主板设备，CPU能够实际使用的只有1~3GB的RAM空间（有时候会更少-我的Vista系统只有2.4G可用）。如果CPU运行在[实模式](http://en.wikipedia.org/wiki/Real_mode)（系统启动早期使用的模式），那么它能够寻址1GB的RAM空间。另外，64位保护模式可以寻址64GB内存空间（虽然很少有芯片支持）。在64位模式，系统在访问那些被主板设备占据的内存的时候,有可能使用超过总RAM的物理地址空间。这就是所谓的可回收内存，在芯片的支持下完成。

下一篇我们将讲述系统启动过程，从按开机键到bootloader跳转到内核，将控制权交给内核。如果你想要了解更多关于这方面的内容，我强烈推荐你看[Intel手册](http://www.intel.cn/content/www/cn/zh/processors/architectures-software-developer-manuals.html)。我是从大的方向把握，但Intel手册写上面的非常详细和准确。这里列举一些。

- [Intel G35 参考手册](http://download.intel.com/design/chipsets/datashts/31760701.pdf)描述了一些支持Core 2的代表性芯片。是本篇博文的主要来源。

- [Intel Core 2 Quad-Core-Q600 参考手册](http://download.intel.com/design/processor/datashts/31559205.pdf)是一个处理器手册。它介绍了处理器上面的每个引脚（实际上并没有太大的实际意义，如果你对它们进行分类，发现也没有多少）。很神秘，也很让人着迷。

- [Intel 软件开发手册](http://www.intel.com/products/processor/manuals/index.htm)都很不错。通俗全面地阐明了有关架构的各种事情。卷1和卷3A有一些很不错的内容（不要被卷这个字吓到，这个卷很小，而且你可以有选择地看）

- [Pádraig Brady](http://www.pixelbeat.org/)建议我链接到Ulrich Drepper一篇[关于内存的的文章](https://www.akkadia.org/drepper/cpumemory.pdf)，写的非常的好。我一直在等待有一篇（当然越多越好）关于内存的博文可以链接这篇文章，

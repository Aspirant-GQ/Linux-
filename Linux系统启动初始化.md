@[TOC]

Linux系统启动初始化的主要流程是：

1.上电BIOS自检

2.启动Boot Loader（GRUB）

3.加载内核

4.启动第一个进程

5.配置环境

# 一.BIOS 加载启动引导程序



>  **BIOS**（英文：**B**asic **I**nput/**O**utput **S**ystem），即**基本输入输出系统**，亦称为ROM BIOS、System BIOS、PC BIOS，是在通电引导阶段运行硬件初始化，以及为[操作系统](https://zh.wikipedia.org/wiki/作業系統)提供运行时服务的[固件](https://zh.wikipedia.org/wiki/韌體)。BIOS最早随着[CP/M](https://zh.wikipedia.org/wiki/CP/M)操作系统的推出在1975年出现。BIOS预安装在[个人电脑](https://zh.wikipedia.org/wiki/個人電腦)的[主板](https://zh.wikipedia.org/wiki/主機板)上，是[个人电脑](https://zh.wikipedia.org/wiki/个人电脑)启动时加载的第一个软件。
>
>  现在，BIOS的作用是初始化和测试[硬件](https://zh.wikipedia.org/wiki/硬體)组件，以及从大容量存储设备（如硬盘）加载[引导程序](https://zh.wikipedia.org/wiki/啟動程式)，并由[引导程序](https://zh.wikipedia.org/wiki/啟動程式)加载操作系统。BIOS还为[DOS](https://zh.wikipedia.org/wiki/DOS)操作系统提供键盘、显示及其他[I/O](https://zh.wikipedia.org/wiki/I/O)设备的[硬件抽象层](https://zh.wikipedia.org/wiki/硬體抽象層)。
>
>  许多BIOS程序都只能在特定电脑型号或特定主板型号上运行。早年，BIOS存储于[ROM](https://zh.wikipedia.org/wiki/ROM)芯片上；现在的BIOS多存储于[闪存](https://zh.wikipedia.org/wiki/快閃記憶體)芯片上，这方便了BIOS的更新。



计算机上电之后，CPU就可以执行程序了，但是此时内存中并没有程序让CPU执行，因为内存是RAM掉电丢失。而且操作系统也没有装上，这个时候就要装系统了。ROM中就固化了一些计算机刚上电要执行的初始化程序，也就是BIOS（Basic Input and Output System，基本输入输出系统）  。简单地理解 BIOS，它就是固化在主板上一个 ROM（只读存储器）芯片上的程序，==主要保存计算机的基本输入/输出信息、系统设置信息、开机自检程和系统自启动程序，用来为 计算机提供最底层和最直接的硬件设置与控制。==



BIOS 的初始化主要完成以下 3 项工作：

1. 第一次检查计算机硬件和外围设备（第二次自检由内核完后，后续会讲），例如 CPU、内存、风扇灯。当 BIOS 一启动，就会做一个==自我检测的工作==，整个自检过程也被称为 POST（Power On Self Test）自检。
2. 如果自检没有问题，==BIOS 开始对硬件进行初始化==，并规定当前可启动设备的先后顺序，选择由那个设备来开机。
3. 选择好开启设备后，==就会从该设备的 MBR（主引导扇区）中读取 Boot Loader（启动引导程序）并执行。启动引导程序用于引导操作系统启动，Linux 系统中默认使用的启动引导程序是 GRUB。==



**BIOS从MBR中读取启动引导程序，加载至RAM中，就可以执行启动引导程序了。**



# 二.MBR 主引导扇区

==MBR是用来存储启动引导程序的！！！==

 MBR 也就是主引导扇区 ，位于硬盘的 0 磁道、0 柱面、1 扇区中，主要记录了启动引导程序和磁盘的分区表 ，如图是MBR的结构：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200928205605352.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNzQzNzYy,size_16,color_FFFFFF,t_70#pic_center)

MBR大小是一个扇区512Byte， 其中 446 Byte 安装了启动引导程序，其后 64 Byte 描述分区表，最后的 2 Byte 是结束标记 。

==BIOS从MBR中读取启动引导程序，将启动引导程序加载至RAM中，然后BIOS将控制权交给启动引导程序。==

所以，虽然启动引导程序是在MBR中的，但是实际上是由BIOS从MBR中将启动引导程序加载至RAM中运行的！

> 注意：这里的446Byte中存放的只是启动引导程序的一个镜像文件，后面还要通过这个镜像文件来加载出完整的启动引导程序！



# 三.GRUB引导内核

Linux下是通过一个工具Grub2（ Grand Unified Bootloader Version 2 ），这个工具就是专门引导系统启动的。

> **GNU GRUB**（简称“GRUB”）是一个来自[GNU项目](https://zh.wikipedia.org/wiki/GNU計劃)的[启动引导程序](https://zh.wikipedia.org/wiki/启动引导程序)。GRUB是[多启动规范](https://zh.wikipedia.org/w/index.php?title=多启动规范&action=edit&redlink=1)的实现，它允许用户可以在计算机内同时拥有多个操作系统，并在计算机启动时选择希望运行的操作系统。GRUB可用于选择操作系统分区上的不同[内核](https://zh.wikipedia.org/wiki/内核)，也可用于向这些内核传递启动参数。 





## 3.1运行 boot.img

> BootLoader，是启动引导程序，==启动引导程序的主要任务就是加载操作系统内核==， 每种操作系统的文件格式不同，因此，每种操作系统的启动引导程序也不一样。不同的操作系统只有使用自己的启动引导程序才能加载自己的内核。这里使用GRUB2作为启动引导程序。 

上面说BIOS将启动引导程序加载到RAM中执行，但是==实际上启动引导程序的大小比512Byte要大，所以MBR中的只是一个镜像文件：`boot.img`。我们还得通过MBR中的`boot.img`找到完整的启动引导程序，从而正式启动内核。==

所以，`boot.img`的任务并不是启动内核，而是加载其他镜像文件来完成内核的启动，也可以理解为只有446Byte大小的`boot.img`不足以启动内核，只能召唤出更加强大厉害的其他镜像文件来启动内核。



## 3.2加载 core.img

这个强大的后援就是：`core.img`， core.img 由 lzma_decompress.img、diskboot.img、kernel.img 和一系列的模块组成，功能比较丰富，能做很多事情。 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200928205650643.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNzQzNzYy,size_16,color_FFFFFF,t_70#pic_center)



 boot.img 先加载的是 core.img 的第一个扇区。如果从硬盘启动的话，这个扇区里面是 diskboot.img，对应的代码是 diskboot.S。boot.img 将控制权交给 diskboot.img 后，diskboot.img 的任务就是将 core.img 的其他部分加载进来，先是解压缩程序 lzma_decompress.img，再往下是 kernel.img，最后是各个模块 module 对应的映像。这里需要注意，它不是 Linux 的内核，而是 grub 的内核。lzma_decompress.img 对应的代码是 startup_raw.S，本来 kernel.img 是压缩过的，现在执行的时候，需要解压缩。 

 在这之前，我们所有遇到过的程序都非常非常小，完全可以在实模式下运行，但是随着我们加载的东西越来越大，实模式这 1M 的地址空间实在放不下了，==所以在真正的解压缩之前，lzma_decompress.img 做了一个重要的决定，就是调用 real_to_prot，切换到保护模式==，这样就能在更大的寻址空间里面，加载更多的东西。 



## 3.3切换到保护模式

切换到保护模式需要做以下几点：

* 启用分段，在内存中建立段描述符表（和中断向量表差不多），原来的段寄存器指向现在的段描述符
* 启动分页，现在内存大了，需要分页进行管理
* 打开Gate A20，使用第21根地址控制线

## 3.4kernel.img 引导内核

这里kernel.img不是Linux的内核，而是GRUB的内核。

 kernel.img 对应的代码是 startup.S 以及一堆 c 文件，在 startup.S 中会调用 grub_main，这是 grub kernel 的主函数，主函数中 grub_load_config() 开始解析，我们上面写的那个 grub.conf 文件里的配置信息。 

当启动了操作系统后，就要开始调用 grub_menu_execute_entry() ，开始解析并执行你选择的那一项。 



 例如里面的 linux16 命令，表示装载指定的内核文件，并传递内核启动参数。于是 grub_cmd_linux() 函数会被调用，它会首先读取 Linux 内核镜像头部的一些数据结构，放到内存中的数据结构来，进行检查。如果检查通过，则会读取整个 Linux 内核镜像到内存。 

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200928205904181.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNzQzNzYy,size_16,color_FFFFFF,t_70#pic_center)




# 四.内核初始化

上面已经引导了操作系统，也就是引导了内核，接下来就是内核的初始化了。

内核启动的入口函数从`start_kernel()`开始， 在`init/main.c ` 文件中，start_kernel 相当于内核的 main 函数。打开这个函数，你会发现，里面是各种各样初始化函数 XXXX_init .

内核初始化做了以下这些事：

* 创建==0号进程， 这是唯一一个没有通过 fork 或者 kernel_thread 产生的进程==，是进程列表的第一个。 
* 初始化中断， 对应的函数是 trap_init()  ，里面设置了很多中断门（Interrupt Gate），用于处理各种中断。其中有一个 ==set_system_intr_gate(IA32_SYSCALL_VECTOR, entry_INT80_32)，这是系统调用的中断门==。系统调用也是通过发送中断的方式进行的。 
* 初始化内存管理模块， mm_init() 
* 初始化调度模块， sched_init() 
* 初始化基于内存的文件系统rootfs， vfs_caches_init() 
* 初始化其他， rest_init() 
* 创建用==户态第一个进程==：1号进程，用 kernel_thread(kernel_init, NULL, CLONE_FS) 创建1号进程 
* 创建内核态第一个进程：2号进程， kernel_thread(kthreadd, NULL, CLONE_FS | CLONE_FILES) 



这样看来，内核初始化靠的是`start_kernel()`函数，初始化主要做三件事：
1.创建样板进程（也就是0号进程），以及各个模块的初始化

2.创建用户态的进程

3.创建内核态的进程



用户态访问核心资源时，通过中断来请求，就是系统调用的统一中断，流程如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200928210154781.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNzQzNzYy,size_16,color_FFFFFF,t_70#pic_center)




用户态和内核态进程执行的框图如下：
![img](https://img-blog.csdnimg.cn/img_convert/21afbb454497c03fda7a9bffa57d49e4.png)

# 五.系统调用

从上面得知，一般程序运行在用户态，如果要想使用核心资源，就要进入内核态，就要通过系统调用。

此外，linux的一些驱动程序都是写在内核中的，当上层应用想要调用驱动的接口时，也要通过系统调用来实现。



但是，直接操作系统调用比较繁琐，所以一般使用glibc库对系统调用进行封装。

64位的系统调用如图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200928210225662.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNzQzNzYy,size_16,color_FFFFFF,t_70#pic_center)



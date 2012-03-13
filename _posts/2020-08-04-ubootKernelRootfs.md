---
layout: post
title: 嵌入式系统移植 学习记录
tags: 嵌入式
---

嵌入式系统移植

uboot/LinuxKernel/rootfs学习记录

<!--more-->

| 修订时间   | 修订内容                                | 修订人 |
| :--------- | :-------------------------------------- | :----- |
| 2020-08-04 | 搭建文章框架；<br>补充uboot命令行模式； | JD     |
| 2020-09-01 | 完善uboot部分：<br>源码分析；           | JD     |
| 2020-09-02 | 完善uboot部分：<br>uboot代码流程；      | JD     |


# uboot

> U-boot(Universal Boot Loader) is an open-source, primary boot loader used in embedded devices to package the instructions to boot the device's operating system kernel. --[Wiki](https://en.wikipedia.org/wiki/Das_U-Boot)


基于高通ipq4019 平台学习 uboot。


我们常用的PC都是基于X86机器架构，通常PC的BIOS程序存放在Norflash中，OS存放在外部存储器磁盘中，RAM和CPU掉电后是不工作的。


上电后，Norflash中的BIOS程序先运行，BIOS主要是负责将RAM内存/GPIO/声卡/网卡等外设硬件的初始化并检查，然后将OS复制到RAM中，以完成对系统的启动。


在嵌入式设备中，uboot也是被用来解决PC下类似的问题。所以，uboot具备以下特点：
- 上电后直接启动；
- 管理SOC和板级外设（串口、LCD控制器等）；
- 能够引导内核并传参；
- 具备完整系统部署的功能；
- uboot是一个裸机程序，并且是单线程；
- uboot的出口是启动内核，若没有启动内核，则uboot将会被一直执行；
- uboot实现了一个shell界面；


## uboot 源码文件框架

通过Linux 下 `tree -L 1 -d `命令 ，我们可以获取uboot的文件框架通常如下所示：

```s
uboot-1.0
├── api     与硬件无关的API函数
├── arch    与架构体系有关的代码
├── board   不同板子的定制代码
├── common  通用代码
├── disk    磁盘分区相关代码
├── doc
├── drivers 驱动代码
├── dts     设备树
├── examples示例代码
├── fs      文件系统
├── include 头文件
├── lib     库文件
├── nand_spl
├── net     网络相关代码
├── onenand_ipl
├── post    上电自检程序
├── spl
├── test    测试代码
└── tools
```

基于高通ipq4019 我们需要着重关注以下几个重要的文件。

```s
uboot-1.0
├── arch
│   ├── cpu
│   │   ├── armv7
│   │   │   ├── ipq
│   ├── include
│   │   └── asm
│   │       ├── arch-ipq40xx
│   │       │   └── ess
│   └── lib

qca\src\uboot-1.0\arch\arm\cpu\
-rw-rw-r--  1 jiangdi jiangdi 1973 Sep  2 10:22 u-boot.lds #生成uboot.img 可执行文件的 链接脚本

qca/src/uboot-1.0/arch/arm/cpu/armv7

-rw-rw-r--  1 jiangdi jiangdi 10638 Sep  2 10:22 cache_v7.c
-rw-rw-r--  1 jiangdi jiangdi  1654 Sep  2 10:22 config.mk
-rw-rw-r--  1 jiangdi jiangdi  2388 Sep  2 10:22 cpu.c
-rw-rw-r--  1 jiangdi jiangdi  1525 Sep  2 10:22 Makefile
-rw-rw-r--  1 jiangdi jiangdi 13762 Sep  2 10:22 start.S #uboot的入口
-rw-rw-r--  1 jiangdi jiangdi  2279 Sep  2 10:22 syslib.c

qca/src/uboot-1.0/arch/arm/cpu/armv7/ipq
-rw-rw-r-- 1 jiangdi jiangdi   263 Sep  2 10:22 asm-offsets.c
-rw-rw-r-- 1 jiangdi jiangdi 14690 Sep  2 10:22 clock.c
-rw-rw-r-- 1 jiangdi jiangdi 16324 Sep  2 10:22 cmd_bootipq.c
-rw-rw-r-- 1 jiangdi jiangdi  3155 Sep  2 10:22 cmd_dumpipq_data.c
-rw-rw-r-- 1 jiangdi jiangdi  2817 Sep  2 10:22 gpio.c
-rw-rw-r-- 1 jiangdi jiangdi   716 Sep  2 10:22 Makefile
-rw-rw-r-- 1 jiangdi jiangdi  6820 Sep  2 10:22 scm.c
-rw-rw-r-- 1 jiangdi jiangdi 11438 Sep  2 10:22 smem.c
-rw-rw-r-- 1 jiangdi jiangdi  3632 Sep  2 10:22 timer.c


qca/src/uboot-1.0/arch/arm/include/asm/arch-ipq40xx
drwxrwxr-x 2 jiangdi jiangdi 4096 Sep  2 10:22 ess
-rw-rw-r-- 1 jiangdi jiangdi 2157 Sep  2 10:22 iomap.h
-rw-rw-r-- 1 jiangdi jiangdi 1190 Sep  2 10:22 scm.h
-rw-rw-r-- 1 jiangdi jiangdi 4143 Sep  2 10:22 smem.h
-rw-rw-r-- 1 jiangdi jiangdi 2005 Sep  2 10:22 timer.h

qca/src/uboot-1.0/arch/arm/lib
-rw-rw-r-- 1 jiangdi jiangdi  1549 Sep  2 10:22 _ashldi3.S
-rw-rw-r-- 1 jiangdi jiangdi  1549 Sep  2 10:22 _ashrdi3.S
-rw-rw-r-- 1 jiangdi jiangdi 17900 Sep  2 10:22 board.c # uboot的第二阶段
-rw-rw-r-- 1 jiangdi jiangdi 12232 Sep  2 10:22 bootm.c
-rw-rw-r-- 1 jiangdi jiangdi  1976 Sep  2 10:22 cache.c
-rw-rw-r-- 1 jiangdi jiangdi  5396 Sep  2 10:22 cache-cp15.c
-rw-rw-r-- 1 jiangdi jiangdi  3185 Sep  2 10:22 cache-pl310.c
-rw-rw-r-- 1 jiangdi jiangdi  1007 Sep  2 10:22 div0.c
-rw-rw-r-- 1 jiangdi jiangdi  3127 Sep  2 10:22 _divsi3.S
-rw-rw-r-- 1 jiangdi jiangdi   729 Sep  2 10:22 eabi_compat.c
-rw-rw-r-- 1 jiangdi jiangdi  4697 Sep  2 10:22 interrupts.c
-rw-rw-r-- 1 jiangdi jiangdi  1549 Sep  2 10:22 _lshrdi3.S
-rw-rw-r-- 1 jiangdi jiangdi  2303 Sep  2 10:22 Makefile
-rw-rw-r-- 1 jiangdi jiangdi  4928 Sep  2 10:22 memcpy.S
-rw-rw-r-- 1 jiangdi jiangdi  2437 Sep  2 10:22 memset.S
-rw-rw-r-- 1 jiangdi jiangdi  2316 Sep  2 10:22 _modsi3.S
-rw-rw-r-- 1 jiangdi jiangdi  1496 Sep  2 10:22 reset.c
-rw-rw-r-- 1 jiangdi jiangdi  2431 Sep  2 10:22 _udivsi3.S
-rw-rw-r-- 1 jiangdi jiangdi  2762 Sep  2 10:22 _umodsi3.S


uboot-1.0
├── board
│   ├── qcom
│   │   ├── common
│   │   ├── ipq40xx_cdp


qca\src\uboot-1.0\board\qcom\ipq40xx_cdp
-rw-rw-r-- 1 jiangdi jiangdi 38081 Sep  2 10:22 ipq40xx_board_param.h
-rw-rw-r-- 1 jiangdi jiangdi 41179 Sep  2 10:22 ipq40xx_cdp.c
-rw-rw-r-- 1 jiangdi jiangdi  4160 Sep  2 10:22 ipq40xx_cdp.h
-rw-rw-r-- 1 jiangdi jiangdi   667 Sep  2 10:22 Makefile

qca/src/uboot-1.0/board/qcom/common
-rw-rw-r-- 1 jiangdi jiangdi 10100 Sep  2 10:22 athrs17_phy.c
-rw-rw-r-- 1 jiangdi jiangdi 25625 Sep  2 10:22 athrs17_phy.h
-rw-rw-r-- 1 jiangdi jiangdi  1291 Sep  2 10:22 ipq806x_phy.h
-rw-rw-r-- 1 jiangdi jiangdi 13361 Sep  2 10:22 qca8511.c
-rw-rw-r-- 1 jiangdi jiangdi  5850 Sep  2 10:22 qca8511.h
-rw-rw-r-- 1 jiangdi jiangdi   768 Sep  2 10:22 qca_common.h
```

## uboot 启动流程分析

uboot 启动通常是两阶段的启动过程。

第一阶段通常使用汇编语言，依据CPU体系结构初始化，并调用第二阶段的代码；

- 硬件初始化；
  - 关闭WATCHDOG；
  - 关中断；
  - 设置CPU速率、时钟频率及终端控制寄存器；
  - RAM初始化；
- 初始化堆栈；
- 跳转到第二阶段代码（C语言）的入口点；

uboot的第一阶段在`start.S`中处理。

第二阶段通常使用C语言实现更为复杂的功能；
- 初始化外设；
  - 至少初始化一个串口方便交互；
- 检测内存映射；
- 设置内核启动参数，调用kernel；

uboot的第二阶段通常再`board.c`中

## uboot 源码分析

由于整个uboot源码十分庞大，所以我们只需要分析部分重要部分的代码即可。

### 预备知识

学习uboot源码，需要对以下几个概念做到有些了解；

#### 连接器lds文件

链接器脚本文件，用来描述链接器如何链接生成一个目标执行文件。

在uboot中，我们将汇编文件(start.s)生成目标文件（PS:汇编指令转为机器指令）后，需要再将目标文件链接生成可执行文件，uboot的链接过程需要lds文件传递给ld链接器用来生成可执行文件。

- [链接器*.lds文件简介](https://blog.csdn.net/rikeyone/article/details/104267605?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.edu_weight&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.edu_weight)
- [Linker Script File](http://bravegnu.org/gnu-eprog/lds.html)
- [GNU Linker Script（.lds文件）的学习](https://blog.csdn.net/eleanoryss/article/details/72142892)

#### 可执行文件ELF
编译器编译源代码后生成的文件称作目标文件，目标文件经过编译器链接之后可以得到**可执行文件**

- [计算机那些事(4)——ELF文件结构](http://chuquan.me/2018/05/21/elf-introduce/)
- [GCC编译器原理（二）------编译原理一：目标文件](https://www.cnblogs.com/kele-dad/p/9478344.html)
- [计算机那些事(3)——程序构建及编译原理](http://chuquan.me/2018/05/12/compiler-principle/)


### 重要代码
1. uboot.lds：主要是负责告诉链接器Linker如何在可执行文件中配置代码段，数据段和bss段等等。
2. start.S：主要是负责uboot运行中的第一阶段；
3. board.c：

参考文档:
- [u-boot.lds链接文件详解](https://www.jianshu.com/p/ec39403db315)
- [uboot-的start.S详细注解及分析](http://blog.chinaunix.net/uid-22891435-id-380150.html)




### 顶层Makefile分析


## uboot 命令行模式
进入uboot的命令行模式后，一般常用的命令如下所示：
```s
#查看当前uboot所支持的命令
help
#查看某个命令的用法
helo bootz
#常用的信息查询命令（板子信息，输出环境变量，输出uboot版本号）
bdinfo
printenv
version
#修改环境变量
setenv
setenv bootdelay 5
setenv bootargs 'console = ttymxc0,115200 root = /dev/mmcblklp2 rootwait rw'
#删除环境变量
setenv bootargs
#保存修稿后的环境变量
saveenv
# 网络操作命令
ping
# BOOT 操作命令
# bootz命令用于自动zIamge文件
bootz
# bootm用于启动uImage镜像文件
bootm
# boot命令读取环境变量bootcmd来启动Linux系统
boot
# 其他常用命令
# 重启命令
reset
# 用于运行我们自定义的环境变量
run
# 内存读写测试命令
mtest
```
## uboot spi驱动
基于spi驱动 SSD1309 OLED 显示；
```c

```


### 参考资料
- [SPI Memories - Linux Foundation Events](https://events19.linuxfoundation.org/wp-content/uploads/2017/12/raynal-spi-memories.pdf)
- [U-boot Splashscreen through SPI](https://stackoverflow.com/questions/62529964/u-boot-splashscreen-through-spi)
- [U-Boot Drivers](https://www.chromium.org/chromium-os/firmware-porting-guide/u-boot-drivers)
- [Driver Model In Uboot](https://www.denx.de/wiki/pub/U-Boot/MiniSummitELCE2014/dm-u-boot.pdf)


## uboot 参考资料
- [UBoot 源码分析(1)——快刀斩乱麻认识UBoot](https://www.pianshen.com/article/9373217635/)
- [uboot源码简要分析](https://www.cnblogs.com/80scd/p/5872425.html)
- [嵌入式linux开发uboot启动过程源码分析(一)](https://www.cnblogs.com/cyyljw/p/10998066.html)
- [uboot学习前传](https://blog.51cto.com/whylinux/1898776)
- [UBOOT从零开始的学习](https://blog.csdn.net/dhauwd/category_9274153.html)
- [UBOOT源码分析（详细）](https://blog.csdn.net/davidlinux/article/details/45725305)
- [uboot启动流程](https://zhuanlan.zhihu.com/p/65853967)
- [u-boot](https://blog.csdn.net/itxiebo/category_6140192.html)
- [uboot启动第一阶段](https://mengchaobbbigrui.github.io/2019/04/14/uboot%E5%90%AF%E5%8A%A8%E7%AC%AC%E4%B8%80%E9%98%B6%E6%AE%B5/)
- [uboot启动第二阶段](https://mengchaobbbigrui.github.io/2019/04/15/uboot%E5%90%AF%E5%8A%A8%E7%AC%AC%E4%BA%8C%E9%98%B6%E6%AE%B5/)
- [uboot启动内核、命令体系、环境变量](https://mengchaobbbigrui.github.io/2019/04/16/uboot%E5%90%AF%E5%8A%A8%E5%86%85%E6%A0%B8-%E5%91%BD%E4%BB%A4%E4%BD%93%E7%B3%BB-%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F/)
- [u-boot](https://blog.csdn.net/u010979030/category_1810283.html)
- [uboot代码解析](https://dycc.github.io/2016/03/17/uboot.html)






# Linux Kernel Image

# rootfs


# 系统移植步骤


# 参考资料

- [Linux源代码阅读——内核引导](http://home.ustc.edu.cn/~boj/courses/linux_kernel/1_boot.html)
- [Kernel Notes](http://home.ustc.edu.cn/~ljm0910/kernel.html)

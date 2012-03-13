---
layout: post
title: 操作系统 启示录
tags: 操作系统
---
操作系统学习笔记 基于CMU 213, MIT 6.828 和清华大学OS课程

![MIT](https://www.ledgerinsights.com/wp-content/uploads/2019/10/MIT-810x476.png)

<!--more-->

# MIT 6.828 Operate System Engineering
## 部署环境
基础环境：
我的开发环境是Unbuntu 20.04 LTS。输入如下`cat /proc/version`命令查看内核版本：
```
Linux version 5.4.0-26-generic (buildd@lcy01-amd64-029) (gcc version 9.3.0 (Ubuntu 9.3.0-10ubuntu2)) #30-Ubuntu SMP Mon Apr 20 16:58:30 UTC 2020
```
### 1.编译工具链确定
```s
objdump -i
# The second line should say elf32-i386.
gcc -m32 -print-libgcc-file-name
# /usr/lib/gcc/x86_64-linux-gnu/9/libgcc.a
```
如上显示类似答案，则说明编译工具链没有问题。

### 2.安装QEMU
```s
git clone https://github.com/mit-pdos/6.828-qemu.git qemu

sudo apt-get install libsdl1.2-dev
sudo apt-get install libtool-bin
sudo apt-get install libglib2.0-dev
sudo apt-get install libz-dev
sudo apt-get install libpixman-1-dev

# 我是使用官网提供的Unbuntu桌面版 还需要下载以下的环境
sudo apt-get install python
sudo apt-get install g++
sudo apt-get install make


sudo ./configure --disable-kvm --disable-werror --target-list="i386-softmmu x86_64-softmmu"

sudo make 

sudo make install
```

编译过程中遇到的问题及解决方法：
- 问题：`qga/commands-posix.o: in function 'dev_major_minor' ` 
   - 解决方法：在qga/commands-posix.c 中添加头文件`#include <sys/sysmacros.h>`即可解决。 具体问题解决 [参考链接](https://github.com/Ebiroll/qemu_esp32/issues/12)

参考如下所示
1. [MIT 6.828 环境配置](https://zhuanlan.zhihu.com/p/58143429)
2. [MIT 6.828 2017版本环境配置](https://zhuanlan.zhihu.com/p/36889298)
3. [MIT-6.828-JOS-环境搭建](https://www.cnblogs.com/gatsby123/p/9746193.html)


### 3. Unbuntu Tips
#### 修改root用户密码
```s
# 在已经登陆好已创建的用户命令行下，执行如下命令即可修改root的密码
sudo passwd
```
#### 修改Unbuntu 默认编辑器
```s
sudo update-alternatives --config editor
#There are 4 choices for the alternative editor (providing /usr/bin/editor).
#
#  Selection    Path                Priority   Status
#------------------------------------------------------------
#* 0            /bin/nano            40        auto mode
#  1            /bin/ed             -100       manual mode
#  2            /bin/nano            40        manual mode
#  3            /usr/bin/vim.basic   30        manual mode
#  4            /usr/bin/vim.tiny    15        manual mode
#
#Press <enter> to keep the current choice[*], or type selection number: 4
```

## Lab1 Booting a PC
Lab1 可以分成如下三个部分来学习：
- 熟悉X86的汇编语言、QEMU X86仿真器和PC上电后的boot引导过程
- 第二部分了解JOS的boot loader
- 深入了解JOS的内核

### Part I 汇编、QEMU和boot引导过程
#### X86 汇编
这一部分的材料是希望我们通过资料页参考页提供的一些材料，能够对汇编代码有一定的了解。

JOS 采用的是X86结构的机器，汇编指令是AT&T语法，使用的都是Inter架构的指令。

我对Lab里推荐了几个阅读和学习汇编的材料整理如下所示：
- [PC Assembly Language Book](https://pdos.csail.mit.edu/6.828/2018/readings/pcasm-book.pdf):这本书写的比较全一些；
- [Brennan's Guide to Inline Assembly](http://www.delorie.com/djgpp/doc/brennan/brennan_att_inline_djgpp.html)：由于上一本书使用的是NASM语法，但是JOS使用的是AT&T的汇编语法，所以可以通过阅读这个材料来弄明白两个材料的不同；同时这个阅读材料也简要摘取出了JOS将要用到的一些汇编指令；
- [Intel 80386 Reference Programmer's Manual](https://pdos.csail.mit.edu/6.828/2018/readings/i386/toc.htm)：这个材料能够使用一些简明的方式为告诉我们在这门课里X86处理器的一些细节；
- [IA-32 interl Archtecture Software Developer's Manuals](https://software.intel.com/content/www/us/en/develop/articles/intel-sdm.html)：是一份很全的资料，几乎涵盖了最近的处理器的所有细节。一般学有余力再去看看。

在JOS下常见汇编命令简述：
```asm

```

#### QEMU
> Instead of developing the operating system on a real, physical personal computer (PC), we use a program that faithfully emulates a complete PC: the code you write for the emulator will boot on a real PC too.  ----QEMU

```s
#启动QEMU Emulator
make qemu # or  make qemu-nox
```
#### 内存物理地址

![The PC's Physical Address Space](https://yangbolong.github.io/images/lab2.mm.png) 

从0x00000000到0x000A0000的640Kb 称为 “Low Memory”，这一部分区域通常时RAM。

从0x000A0000到0x000FFFFF可以分为三个部分：
- 0x000A0000(640KB) -- 0x000C0000(768KB) VGA 显示Buffer
- 0x000C0000(768KB) -- 0x000F0000(960KB) 16 Bit ROM拓展
- 0x000F0000(960KB) -- 0x00100000(1MB) BIOS (早期BISO存储在ROM中，但是今天的BIOS存在flash中)

第一台PC是基于16Bit的8088处理器，有效寻址能力做多只有1MB的物理地址空间。后来Inter打破`one megabyte barrier`，发展了基于80286/80386的处理器最多可以支持16Mb到4Gb的空间寻址。同时为了向前的兼容新，保留了早期1MB的存储空间，因此现代的PC在物理内存0x000A000到0x00100000上有一个漏洞，将RAM划分位 “low memory”/“conventional memory” 和“extended memory”。

JOS只使用256MB的物理地址空间，所以我们的PC只支持32位空间寻址。

#### BISO（ROM）
```s
#In one terminal windows
make qemu-gdb
# or
make qemu-nox-gdb
#In another terminal windows
make gdb
#Then Thw windos show as follow:
The target architecture is assumed to be i8086
--Type <RET> for more, q to quit, c to continue without paging--
[f000:fff0]    0xffff0:	ljmp   $0xf000,$0xe05b
0x0000fff0 in ?? ()
+ symbol-file obj/kern/kernel
(gdb) 
```
第一行`[f000:fff0]    0xffff0:	ljmp   $0xf000,$0xe05b`表示如下意思：
- IBM PCs 在上电启动后，一般从物理地址0x000ffff0开始执行(IBM早期PC使用的就是8088处理器)
- PC初始硬件地址为，CS = 0xf000 ， IP = 0xfff0 .
- 这是一条跳转指令，跳转到CS = 0xf000, IP = 0xe05b .

这样的设计是为了确保在机器上电或者重启后，BIDO总是能优先得到机器的控制权。因为在上电后，RAM里并没有任何软件指令。BIOS启动后，设置中断描述符表、初始化各种各样的设备。然后将控制权交给 `bootloader`。

实模式下地址转换：
> physical address = 16 * segment(CS) + offset(IP);
>   16 * 0xf000 + 0xfff0
> = 0xf0000 + 0xfff0
> = 0xffff0

一些从Exercise2 材料中看完的收获：

> 计算机上电后，首先通过BIOS完成上电自检(POST)同时初始化一些IO基础服务,上电自检包括内存检查，BIOS ROM检查以及一些IO设备芯片的检查等。完成初始化后，BIOS进入BOOTSTRAP（用来引导boot的）从floppy disk/hard disk/cd-rom/network导入操作系统，同时将机器的控制权转给操作系统。-------------[THE BIOS, POST and the Boot Strap Loader](http://web.archive.org/web/20040318114524/http://members.iweb.net.au/~pstorr/pcbook/book1/post.htm)

### Part II bootloader
预备知识：

硬盘由若干个以包含512字节扇区（sector）组成，一个扇区时硬盘传输的最小单位，每一次对硬盘的读写操作都必须以扇区为单位对其边界；


bootloader存在硬盘的第一个扇区，因此这个扇区也被叫做`boot sector`。从另一方面也说明bootloader的大小不能超过512字节。


对于MIT6.828, bootloader 主要包含一个汇编文件`boot/boot.S`和C源文件`boot/main.c`。

bootloadter的两个作用：
- 将实模式(real mode)切换至保护模式(protected mode)
  - 因为只有在保护模式下，软件可以寻址访问内存中超出1MB的地址空间。
- 读取内核并载入内存中，将机器的控制权转交给内核
  - 从hard disk中读取内核通过IDE disk device registers的特殊X86 IO指令

bootloader部分源码解读:
```
#待补充
```
Exercise 3:
1. 在bootloader里，是哪条指令16位切换到32位的模式？
   - `ljmp    $PROT_MODE_CSEG, $protcseg`
2. boot loader的最后一条指令是什么？kernel要载入的第一条指令又是什么？
   - boot loader 的最后一条指令是 `0x7d81:	call   *0x10018`
   - kernel 载入的第一条指令是 `movw   $0x1234,0x472`
3. 内核的第一条指令在哪里？
   - 位于`0x10000c:	movw   $0x1234,0x472`
4. 为了获取完整的操作系统镜像，boot loader怎么知道应该读取多少个sector？从哪里可以获取这个信息？
   - 从ELF程序头部表可以获取需要读取的sector数量，即如下指令

```
ph = (struct Proghdr *) ((uint8_t *) ELFHDR + ELFHDR->e_phoff);
eph = ph + ELFHDR->e_phnum;
```


> The boot loader uses the ELF program headers to decide how to load the sections. The program headers specify which parts of the ELF object to load into memory and the destination address each should occupy.


参考如下所示：
- [Lab 1 Exercise 3](https://www.cnblogs.com/fatsheep9146/p/5115086.html)
- [x86 Registers](https://www.eecg.utoronto.ca/~amza/www.mindsec.com/files/x86regs.html)
- [XT, AT and PS/2	 I/O port addresses](http://bochs.sourceforge.net/techspec/PORTS.LST)
- [MIT6.828操作系统工程Lab1-Booting a PC实验报告](https://blog.codedragon.tech/2017/12/09/MIT6-828%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E5%B7%A5%E7%A8%8BLab1-Booting-a-PC%E5%AE%9E%E9%AA%8C%E6%8A%A5%E5%91%8A/)

#### 内核载入
内核载入主要是由`boot/main.c`负责执行，在理解bootloader中得载入内核代码得过程，需要我们对ELF文件具有一定得了解。

>当我们编译和链接C语言源码时，编译器将C语言源码(*.c)翻译成目标文件(*.o)。（目标文件里面是由汇编指令构成。）接着链接器将所有编译好的目标文件链接成可执行文件。可执行文件通常都是以二进制得ELF（Executable and Linkable Format）文件格式呈现。

Exercise 4:
1. 阅读C相关文档 K&R C语言 5.1（Pointers and Addresses）~5.5(Character Pointers and Functions)；
2. 理解`pointers.c`；
3. 重点学习指针的知识；

这一段得练习警告很有意思哈哈
>Warning: Unless you are already thoroughly versed in C, do not skip or even skim this reading exercise.

参考如下：
- [Pointers and Addresses](https://pdos.csail.mit.edu/6.828/2018/labs/lab1/pointers.c)
- [A tutorial by Ted Jensen](https://pdos.csail.mit.edu/6.828/2018/readings/pointers.pdf)



在一个标准的ELF文件组成中：先是一个固定长度的ELF头部(ELF Header)，然后跟着可变长度的程序头部(Program Header)，在程序头部中分别列出需要载入的程序块(Program sections)。ELF的头文件在 `inc/elf.h`中。在这门课的学习中，我们只对以下三项程序块加以关注。
- .text  程序的执行指令
- .rodata   只读数据，比如常量字符串等；
- .data  包括了程序的初始化数据。比如对某全局变量初始化5，则存储在这一程序块。


参考资料如下：
- [the ELF specification ](https://pdos.csail.mit.edu/6.828/2018/readings/elf.pdf)
- [Executable and Linkable Format -ELF](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format)



> Tips:当链接器分配内存地址时，它会在.data段后的存储未初始化数据的.bss段保留一些内存空间。C对未初始化的数据均设置为0，所以在.bss段中没有存储任何数据，只是保留了每个数据的地址与size；


输入`objdump -h obj/kern/kernel`可以查看程序头部的所有Program sections。
```s
jiangdi@ubuntu:~/MIT6.828_Labs$ objdump -h obj/kern/kernel

obj/kern/kernel:     file format elf32-i386

Sections:
Idx Name          Size      VMA       LMA       File off  Algn
  0 .text         00001acd  f0100000  00100000  00001000  2**4
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
  1 .rodata       000006bc  f0101ae0  00101ae0  00002ae0  2**5
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  2 .stab         00004291  f010219c  0010219c  0000319c  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  3 .stabstr      0000197f  f010642d  0010642d  0000742d  2**0
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  4 .data         00009300  f0108000  00108000  00009000  2**12
                  CONTENTS, ALLOC, LOAD, DATA
  5 .got          00000008  f0111300  00111300  00012300  2**2
                  CONTENTS, ALLOC, LOAD, DATA
  6 .got.plt      0000000c  f0111308  00111308  00012308  2**2
                  CONTENTS, ALLOC, LOAD, DATA
  7 .data.rel.local 00001000  f0112000  00112000  00013000  2**12
                  CONTENTS, ALLOC, LOAD, DATA
  8 .data.rel.ro.local 00000044  f0113000  00113000  00014000  2**2
                  CONTENTS, ALLOC, LOAD, DATA
  9 .bss          00000648  f0113060  00113060  00014060  2**5
                  CONTENTS, ALLOC, LOAD, DATA
 10 .comment      00000024  00000000  00000000  000146a8  2**0
                  CONTENTS, READONLY

```

`VMA` link address is the memory address from which the section expects to execute.


`LMA` load address is the memory address at which that section should be loaded into memory.

通常情况下，`VMA` 和 `LMA` 地址是相同的，比如下面的例子
```s
jiangdi@ubuntu:~/MIT6.828_Labs$ objdump -h obj/boot/boot.out

obj/boot/boot.out:     file format elf32-i386

Sections:
Idx Name          Size      VMA       LMA       File off  Algn
  0 .text         0000019c  00007c00  00007c00  00000074  2**2
                  CONTENTS, ALLOC, LOAD, CODE
  1 .eh_frame     0000009c  00007d9c  00007d9c  00000210  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  2 .stab         00000870  00000000  00000000  000002ac  2**2
                  CONTENTS, READONLY, DEBUGGING
  3 .stabstr      00000940  00000000  00000000  00000b1c  2**0
                  CONTENTS, READONLY, DEBUGGING
  4 .comment      00000024  00000000  00000000  0000145c  2**0
                  CONTENTS, READONLY

```

bootloader 使用ELF program headers 决定如何载入这些  program sections。program headers 规定 ELF 里的哪一个部分载入内存，应该占多带内存等等。使用如下命令可以查看program headers。`objdump -x obj/kern/kernel`
```
jiangdi@ubuntu:~/MIT6.828_Labs$ objdump -x obj/kern/kernel

obj/kern/kernel:     file format elf32-i386
obj/kern/kernel
architecture: i386, flags 0x00000112:
EXEC_P, HAS_SYMS, D_PAGED
start address 0x0010000c

Program Header:
    LOAD off    0x00001000 vaddr 0xf0100000 paddr 0x00100000 align 2**12
         filesz 0x00007dac memsz 0x00007dac flags r-x
    LOAD off    0x00009000 vaddr 0xf0108000 paddr 0x00108000 align 2**12
         filesz 0x0000b6a8 memsz 0x0000b6a8 flags rw-
   STACK off    0x00000000 vaddr 0x00000000 paddr 0x00000000 align 2**4
         filesz 0x00000000 memsz 0x00000000 flags rwx

Sections:
Idx Name          Size      VMA       LMA       File off  Algn
  0 .text         00001acd  f0100000  00100000  00001000  2**4
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
  1 .rodata       000006bc  f0101ae0  00101ae0  00002ae0  2**5
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  2 .stab         00004291  f010219c  0010219c  0000319c  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  3 .stabstr      0000197f  f010642d  0010642d  0000742d  2**0
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  4 .data         00009300  f0108000  00108000  00009000  2**12
                  CONTENTS, ALLOC, LOAD, DATA
  5 .got          00000008  f0111300  00111300  00012300  2**2
                  CONTENTS, ALLOC, LOAD, DATA
  6 .got.plt      0000000c  f0111308  00111308  00012308  2**2
                  CONTENTS, ALLOC, LOAD, DATA
  7 .data.rel.local 00001000  f0112000  00112000  00013000  2**12
                  CONTENTS, ALLOC, LOAD, DATA
  8 .data.rel.ro.local 00000044  f0113000  00113000  00014000  2**2
                  CONTENTS, ALLOC, LOAD, DATA
  9 .bss          00000648  f0113060  00113060  00014060  2**5
                  CONTENTS, ALLOC, LOAD, DATA
 10 .comment      00000024  00000000  00000000  000146a8  2**0
                  CONTENTS, READONLY
SYMBOL TABLE:
f0100000 l    d  .text	00000000 .text
f0101ae0 l    d  .rodata	00000000 .rodata
f010219c l    d  .stab	00000000 .stab
f010642d l    d  .stabstr	00000000 .stabstr
f0108000 l    d  .data	00000000 .data
f0111300 l    d  .got	00000000 .got
f0111308 l    d  .got.plt	00000000 .got.plt
f0112000 l    d  .data.rel.local	00000000 .data.rel.local
f0113000 l    d  .data.rel.ro.local	00000000 .data.rel.ro.local
f0113060 l    d  .bss	00000000 .bss
00000000 l    d  .comment	00000000 .comment
00000000 l    df *ABS*	00000000 obj/kern/entry.o
f010002f l       .text	00000000 relocated
f010003e l       .text	00000000 spin
00000000 l    df *ABS*	00000000 entrypgdir.c
00000000 l    df *ABS*	00000000 init.c
00000000 l    df *ABS*	00000000 console.c
f01001d0 l     F .text	0000001e serial_proc_data
f01001ee l     F .text	00000062 cons_intr
f0113080 l     O .bss	00000208 cons
f0100250 l     F .text	0000012e kbd_proc_data
f0113060 l     O .bss	00000004 shift.1385
f0101ca0 l     O .rodata	00000100 shiftcode
f0101ba0 l     O .rodata	00000100 togglecode
f0113000 l     O .data.rel.ro.local	00000010 charcode
f010037e l     F .text	00000203 cons_putc
f0113288 l     O .bss	00000002 crt_pos
f0113290 l     O .bss	00000004 addr_6845
f011328c l     O .bss	00000004 crt_buf
f0113294 l     O .bss	00000001 serial_exists
f0111200 l     O .data	00000100 normalmap
f0111100 l     O .data	00000100 shiftmap
f0111000 l     O .data	00000100 ctlmap
00000000 l    df *ABS*	00000000 monitor.c
f0113010 l     O .data.rel.ro.local	00000018 commands
00000000 l    df *ABS*	00000000 printf.c
f0100a2c l     F .text	00000026 putch
00000000 l    df *ABS*	00000000 kdebug.c
f0100aa5 l     F .text	000000f5 stab_binsearch
00000000 l    df *ABS*	00000000 printfmt.c
f0100dac l     F .text	000000be printnum
f0100e6a l     F .text	00000021 sprintputch
f0113028 l     O .data.rel.ro.local	0000001c error_string
f01012fe l       .text	00000000 .L20
f0100fa2 l       .text	00000000 .L36
f01012eb l       .text	00000000 .L35
f0100f5e l       .text	00000000 .L34
f0100f27 l       .text	00000000 .L66
f0100f8a l       .text	00000000 .L33
f0100f30 l       .text	00000000 .L32
f0100f39 l       .text	00000000 .L31
f0100fc5 l       .text	00000000 .L30
f010112f l       .text	00000000 .L29
f0100fe1 l       .text	00000000 .L28
f0100fb9 l       .text	00000000 .L27
f010120a l       .text	00000000 .L26
f010122a l       .text	00000000 .L25
f010103a l       .text	00000000 .L24
f01011b8 l       .text	00000000 .L23
f0101296 l       .text	00000000 .L21
00000000 l    df *ABS*	00000000 readline.c
f01132a0 l     O .bss	00000400 buf
00000000 l    df *ABS*	00000000 string.c
00000000 l    df *ABS*	00000000 
f0111308 l     O .got.plt	00000000 _GLOBAL_OFFSET_TABLE_
f0100da8 g     F .text	00000000 .hidden __x86.get_pc_thunk.cx
f010000c g       .text	00000000 entry
f01014f6 g     F .text	00000026 strcpy
f01005ac g     F .text	00000021 kbd_intr
f01008b1 g     F .text	0000000a mon_backtrace
f010010e g     F .text	0000006e _panic
f0100784 g     F .text	00000000 .hidden __x86.get_pc_thunk.si
f01000aa g     F .text	00000064 i386_init
f01016ac g     F .text	00000066 memmove
f010138c g     F .text	0000001e snprintf
f0100eac g     F .text	0000047d vprintfmt
f01005cd g     F .text	0000005a cons_getc
f0100a8d g     F .text	00000018 cprintf
f0101712 g     F .text	0000001a memcpy
f01013aa g     F .text	00000109 readline
f0110000 g     O .data	00001000 entry_pgtable
f0100040 g     F .text	0000006a test_backtrace
f0101329 g     F .text	00000063 vsnprintf
f0113060 g       .bss	00000000 edata
f0100627 g     F .text	00000126 cons_init
f0100780 g     F .text	00000000 .hidden __x86.get_pc_thunk.ax
f010642c g       .stab	00000000 __STAB_END__
f010642d g       .stabstr	00000000 __STABSTR_BEGIN__
f0101980 g     F .text	0000014d .hidden __umoddi3
f0100581 g     F .text	0000002b serial_intr
f0101870 g     F .text	0000010a .hidden __udivdi3
f0100776 g     F .text	0000000a iscons
f010178a g     F .text	000000de strtol
f01014cf g     F .text	00000027 strnlen
f010151c g     F .text	00000029 strcat
f01136a4 g     O .bss	00000004 panicstr
f01136a0 g       .bss	00000000 end
f010017c g     F .text	00000050 _warn
f0101640 g     F .text	00000020 strfind
f0101acd g       .text	00000000 etext
0010000c g       .text	00000000 _start
f0101576 g     F .text	0000003f strlcpy
f01015df g     F .text	0000003c strncmp
f0101545 g     F .text	00000031 strncpy
f01001cc g     F .text	00000000 .hidden __x86.get_pc_thunk.bx
f010172c g     F .text	0000003d memcmp
f010074d g     F .text	00000014 cputchar
f0101660 g     F .text	0000004c memset
f0100761 g     F .text	00000015 getchar
f0100e8b g     F .text	00000021 printfmt
f0107dab g       .stabstr	00000000 __STABSTR_END__
f01015b5 g     F .text	0000002a strcmp
f0100b9a g     F .text	0000020e debuginfo_eip
f0100a52 g     F .text	0000003b vcprintf
f0110000 g       .data	00000000 bootstacktop
f0112000 g     O .data.rel.local	00001000 entry_pgdir
f0108000 g       .data	00000000 bootstack
f010219c g       .stab	00000000 __STAB_BEGIN__
f01014b3 g     F .text	0000001c strlen
f010161b g     F .text	00000025 strchr
f01007dc g     F .text	000000d5 mon_kerninfo
f01008bb g     F .text	00000171 monitor
f0101769 g     F .text	00000021 memfind
f0100788 g     F .text	00000054 mon_help
``` 

前面我们有提到，在BIOS准备载入 bootloader是， 它是在内存地址的 `0x7c00`载入 boot sector，并随后将CPU的控制权转交给它。通过设置链接器`boot/Makefrag`的 `-Ttext 0x7c00`即可实现。
Exercise 5:
1. GDB追踪 BIOS 是如何载入 bootloader的 ？
2. 修改上述`boot/Makefrag`的地址，看看会发生什么 ？

随意修改`-Ttext 0x7c00`为 `-Ttext 0x9c00`后，发现代码始终在0x7c2d这个位置循环跳跃，无法跑下去。
```
   0x7c22:	pushf  
   0x7c23:	mov    %cr0,%eax
   0x7c26:	or     $0x1,%eax
   0x7c2a:	mov    %eax,%cr0
=> 0x7c2d:	ljmp   $0x8,$0x9c32
   0x7c32:	mov    $0xd88e0010,%eax
   0x7c38:	mov    %ax,%es
   0x7c3a:	mov    %ax,%fs
   0x7c3c:	mov    %ax,%gs
   0x7c3e:	mov    %ax,%ss
```

在ELF文件中，我们还需要关注一个`e_entry`的信息。指出程序在内存中从哪里开始执行。
```s
jiangdi@ubuntu:~/MIT6.828_Labs$ objdump -f obj/kern/kernel

obj/kern/kernel:     file format elf32-i386
architecture: i386, flags 0x00000112:
EXEC_P, HAS_SYMS, D_PAGED
start address 0x0010000c
```

通过以上的学习，我们可以掌握最简单的ELF文件(/boot/main.c)，它告诉main.c如何从ELF文件的每个sections中将内核读取到内存中，并跳到entry_point节点。


Exercise 6:
- 熟悉GDB命令的使用 `x/Nx ADDR`；
- 在地址0x00100000设置断点；

### Part III kernel of JOS
这一部分开始了解JOS 内核的一些细节，并尝试写一些代码。 和bootloader类似，内核的启动后也是执行一些汇编指令代码。完成这些汇编指令后，再跑到C语言里去。

#### 虚拟内存映射
操作系统内核通常从运行再虚拟地址的高地址区0xf0100000，这样可以将低地址区域给用户程序使用。具体细节会在下一个lab介绍。

事实上很多机器的物理内存地址没有高达0xf0100000，所以我们需要使用内存管理区将虚拟地址映射到物理地址。

再`kern/entry.S`代码中，直到`CR0_PG`被配置，内存地址寻址从物理寻址切换我虚拟地址。

Exercise 7:
- 追踪代码到执行汇编指令`movl %eax , %cr0`，看看0x00100000和0xf0100000。
- 看看建立映射后的第一条指令是什么？
- 试着将`kern/entry.S`文件中去除`movl %eax %cr0`，看看会发生什么？

```s
(gdb) x/10 0x00100000
   0x100000:	add    %al,(%bx,%si)
   0x100002:	add    %al,(%bx,%si)
   0x100004:	add    %al,(%bx,%si)
   0x100006:	add    %al,(%bx,%si)
   0x100008:	add    %al,(%bx,%si)
   0x10000a:	add    %al,(%bx,%si)
   0x10000c:	add    %al,(%bx,%si)
   0x10000e:	add    %al,(%bx,%si)
   0x100010:	add    %al,(%bx,%si)
   0x100012:	add    %al,(%bx,%si)
(gdb) x/10 0xf0100000
   0xf0100000 <_start-268435468>:	add    %al,(%bx,%si)
   0xf0100002 <_start-268435466>:	add    %al,(%bx,%si)
   0xf0100004 <_start-268435464>:	add    %al,(%bx,%si)
   0xf0100006 <_start-268435462>:	add    %al,(%bx,%si)
   0xf0100008 <_start-268435460>:	add    %al,(%bx,%si)
   0xf010000a <_start-268435458>:	add    %al,(%bx,%si)
   0xf010000c <entry>:	add    %al,(%bx,%si)
   0xf010000e <entry+2>:	add    %al,(%bx,%si)
   0xf0100010 <entry+4>:	add    %al,(%bx,%si)
   0xf0100012 <entry+6>:	add    %al,(%bx,%si)
```

## Refer
- [课程主页](https://pdos.csail.mit.edu/6.828/2018/schedule.html)
- [推荐一门课：6.828](http://zyearn.github.io/blog/2016/02/24/recommmend-6-dot-828/)
- [MIT-大专栏](https://www.dazhuanlan.com/2019/09/25/5d8b4b4a3f2b9/)

# CMU 213 CSAPP
**Computer Systems: A Programmer's Perspective**

## Refer
- [CSAPP HomePage](http://csapp.cs.cmu.edu/3e/labs.html)
- [CS:APP Blog](http://csappbook.blogspot.com/)

# 操作系统-清华大学

## Refer
- [操作系统-学堂在线课程主页](https://next.xuetangx.com/course/THU08091000267/1516699)
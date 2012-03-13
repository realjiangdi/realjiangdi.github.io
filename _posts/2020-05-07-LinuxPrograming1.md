---
layout: post
title: Linux系统编程
tag: Linux

---

通过阅读《Linux系统编程手册》，结合一些工作上的经验，完成对Linux系统编程知识体系的梳理。

<!--more-->

# 0. 修订记录

| 时间 | 修订内容 | 修订人 |
| :------ |:--- | :--- |
| 2020-05-08 | 完成初步框架搭建； | JiangDi |
| 2020-05-08 | 补充基本概念； | JiangDi |

# 1. 摘要

# 2. 基本概念
1. 内核：内核指管理和分配计算机资源的核心层软件。
2. shell：交互程序，读取用户输入的命令，并执行以响应“输入的命令”。也被成为“命令解释器”。此外，对shell脚本解释并执行也是其用途之一。
3. 文件I/O模型：应用程序发起的IO请求（oepn，read，write，close）等，可以用于所有文件类型或者设备文件，内核会转化IO请求为相应文件系统的操作或者设备驱动程序的操作。
  >One of the distinguishing features of the I/O model on UNIX systems is the concept of universality of I/O. This means that the same system calls (open(), read(),write(), close(), and so on) are used to perform I/O on all types of files, includingdevices. (The kernel translates the application’s I/O requests into appropriate file-system or device-driver operations that perform I/O on the target file or device.)Thus, a program employing these system calls will work on any type of file. --[Linux/UNIX系统编程手册](https://book.douban.com/subject/25809330/)
4. 文件描述符：I/O系统使用文件描述符来指代打开的文件。 
 >由shell启动的进程会默认继承3个已经打开的文件描述符。 
 0-输入； 
 1-输出； 
 2-错误输出；
5. 进程：进程是正在允许的程序实例。 
 >逻辑上一个进程同城可以分为以下4个部分：
 >- 文本：程序的指令
 >- 数据：程序使用的静态变量
 >- 堆：程序可以从该区域动态分配的额外内存
 >- 栈：随函数调用、返回而增减的一片内存，主要用于局部变量和函数调用分配内存
6. 静态库与共享库：库的意思是指写好的函数，提供给我们直接调用使用。静态库在编译时，会链接一份副本到可执行文件中去。动态库则写一条记录，只有在程序调用时才会载入。
7. `/proc`文件系统：Linux下一种虚拟的文件系统，提供了一种查看和改变内核系统属性的方式。

# 3. 系统编程基础
## 3.1 处理系统函数调用错误
```c
#include <stdio.h>
void perror(const char *msg);
```

```c
#include <string.h>
char *strerr(int errnum);
```
## 3.2 常用函数及头文件
```c
#ifndef TLPI_HDR_H
#define TLPI_HDR_H

#include <sys/types.h>
#include <stdio.h>
#include <stdlib.h>

#include <unistd.h> /* many system calls*/
#include <error.h>
#include <string.h>
#include "get_num.h"
#include "error_function.h"

typedef enum {FALSE, TRUE} Boolean;

#endif
```
# 3.3 移植
1. 数据结构和机器位数的问题
  
# 4. 通用I/O模型
## 4.1 文件描述符
> a file descriptor, a (usuallysmall) nonnegative integer. File descriptors are used to refer to all types of openfiles, including pipes, FIFOs, sockets, terminals, devices, and regular files. Eachprocess has its own set of file descriptors.


## 4.2 open
>fd = open(pathname,flags,mode) opens the file identified by pathname, returninga file descriptor used to refer to the open file in subsequent calls. If the filedoesn’t exist, open() may create it, depending on the settings of the flags bit-mask argument. The flags argument also specifies whether the file is to beopened for reading, writing, or both. The mode argument specifies the permis-sions to be placed on the file if it is created by this call. If the open() call is notbeing used to create a file, this argument is ignored and can be omitted.

```c
#include <sys/stat.h>
#include <fcntl.h>
int open(const char *pathname ,int flags,...);

//some usage as follow
/* open existing file for read-only */
fd = open("start.up",O_RDONLY);

/* open new or existing file and writing, truncating to zero bytes,file permmission read+write for owner.nothing for other */
fd = open("myfile", O_RDWR|O_CREAT|O_TRUNC,S_IRUSER|S_IWUSER);

/* open new or existing file and writing , write should always append to end of file*/
fd = open("otherfile", O_WRONLY|O_CREAT|O_APPEND,S_IRUESR|S_IWUSER);

```

some explaination of modes as follow:
1. O_CREAT: If the file doesn’t already exist, it is created as a new, empty file.
2. O_EXCL: This flag is used in conjunction with O_CREAT to indicate that if the file already exists, it should not be opened; 
3. O_TRUNC: If the file already exists and is a regular file, then truncate it to zero length, destroying any existing data.
4. O_APPEND
5. O_RDONLY
6. O_WRONLY
7. O_RDWR


## 4.3 close
>status = close( fd) is called after all I/O has been completed, in order to release the file descriptor fd and its associated kernel resources.

```
#include <unistd.h>
int close(int fd);
```
## 4.4 write
> numwritten = write( fd, buffer, count) writes up to count bytes from buffer to theopen file referred to by fd. The write() call returns the number of bytes actuallywritten, which may be less than count.

```c
#include <unistd.h>
ssize_t write(int fd,void* buffer,size_t count);
```
## 4.5 read
>numread = read( fd, buffer, count) reads at most count bytes from the open filereferred to by fd and stores them in buffer. The read() call returns the number ofbytes actually read. If no further bytes could be read (i.e., end-of-file wasencountered), read() returns 0.

注意：read 函数读取的字符串后没有放置`terminating null byte`, 但是printf却要求一个终止字节，所以如下代码在输出时，输出字符串后会跟着一个莫名奇妙的符号。
```c
#define MAX_READ 10

char buffer[MAX_READ];
int count;
int fd;
fd=open("test.txt",O_RDONLY);
if(fd == -1)
    return 1;
    
if((count=read(fd,buffer,MAX_READ)) == -1)
    return 2;
       
  printf("The input data was %s \n",buffer);
  return 0;
}
```
输出结果
```shell
[jiangdi@jiangdi code_test]$ cat test.txt 
1234567890123456789
[jiangdi@jiangdi code_test]$ ./a.out 
The input data was 1234567890@ 
```
如果需要在读取的字符串后放置一个终止字节，我们需要显性的在读取的字符串最后以为放置终止字节。代码需要修改为如下所示：
```c
#define MAX_READ 10

char buffer[MAX_READ+1];
int count;
int fd;
fd=open("test.txt",O_RDONLY);
if(fd == -1)
    return 1;
    
if((count=read(fd,buffer,MAX_READ)) == -1)
    return 2;
  buffer[count]='\0';     
  printf("The input data was %s \n",buffer);
  return 0;
}
```
输出结果如下所示
```shell
[jiangdi@jiangdi code_test]$ cat test.txt 
1234567890123456789
[jiangdi@jiangdi code_test]$ ./a.out 
The input data was 1234567890 
```
## 4.6 lseek
```c
#include <unistd.h>
off_t lseek(int fd,off_t offset,int whence);
```
whence optopms as follow:
- SEEK_SET
  set offset bytes from the beginning of file.
- SEEK_CUR
  adjust offset bytes relative to current file offset.
- SEEK_END
  
some usage as follow :
```c
lseek(fd,0,SEEK_SET);
lseek(fd,0,SEEK_END);
lseek(fd,-1,SEEK_END);
lseek(fd,-10,SEEK_CUR);
lseek(fd,10000,SEEK_END);
```

### 4.6.1 File holes 
> The space in between the previous end of the file and the newly written bytes isreferred  to  as  a  file  hole. 

## 4.7 Exercise
### 4.7.1 mytee
```c
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>

#ifndef BUFSIZE
#define BUFSIZE 1024
#endif

void errExit(char *buf);
int main(int argc , char** argv){

    int out,rcount,wcount;
    char buff[BUFSIZE]={0};
    if(argc == 1 || !strncmp("-h",argv[1],sizeof("-h")) || !strncmp("--help",argv[1],sizeof("--help"))){
        printf("%s file \n",argv[0]);
        return 0;
    }    
    if(!strncmp("-a",argv[1],sizeof("-a"))){
        out=open(argv[2],O_WRONLY|O_CREAT|O_APPEND,S_IRUSR|S_IWUSR);
        printf("append");
    }
    else
         out=open(argv[1],O_RDWR|O_CREAT|O_TRUNC,S_IRUSR|S_IWUSR);
    while((rcount=read(0,buff,BUFSIZE)) > 0)
        if((wcount=write(out,buff,rcount))!=rcount){
            errExit("write");
            return 1;
        }
    if(rcount == -1){
        errExit("read");
        return 1;
    }
    close(out);
    return 0;
}
void errExit(char *buf){
    write(2,buf,sizeof(buf));
}
```
### 4.7.2 my cp
```c
#include <stdio.h>

```

# 5. 进程
简单阐述程序和进程的区别
>a process is an abstract entity, defined by the kernel, to which system resources are allocated in order to execute a program.

进程号
```c
#include <unistd.h>
pid_t getpid(void);
pid_t getppid(void);
```
## 5.1 Memory Layout of a Process
In gernal, the memory allocated to each process is composed of a number of parts, usually referred to as segments.These segments are as follow:
- text segment  
  This segment is made read-only so that a process doesn't accidentally modify its own instructions via a bad pointer value.
- initialized data segment  
  Contains global and static variables that are explicitly initialized.
- uninitialized data segment  
  Contains global and static variables that are not explicitly initialized.Before starting program, the system initialized all memory in the segment to 0. This segemtn also is called bss segment.
- stack  
  a dynamically growing and shrinking segment containing stack frames. Any stack frame is allocated for each currently called function.
- heap  
  a dynamically memory allocated or free at run time. such like malloc() or free()

## 5.2 虚拟内存技术
> The aim of this technique is to make efficient use of both the CPU and RAM (physical memory) by exploiting a property that is typical of most programs: locality of reference. 

虚拟内存技术的提出是为了提高CPU和内存的使用效率。因为大部分程序在运行过程中存在“局部访问(locality of reference)”的特性。也就是说，实际运行中的程序，只有部分程序实际存在于RAM中（resident set）。程序未使用的部分则存在于 Swap Area 中。

Kernel通常创建一张*page table*记录每个进程的内存地址实现虚拟存储技术。
![](/screenshots/virturalMemory.jpg)

虚拟内存技术的优点：
1. 进程彼此之间在内存上相互独立；
2. 进程彼此之间可以共享内存；
   - 执行同一文件的多个进程；
   - 进程间通信；
3. 封装底层内存细节，简化开发难度；
4. 提高程序运行速度；
5. 实现内存page的读写管理；

参考资料：
* [操作系统 虚拟内存技术](https://zhuanlan.zhihu.com/p/53004596)
* [虚拟内存的那点事儿](https://sylvanassun.github.io/2017/10/29/2017-10-29-virtual_memory/)

## 5.3 栈
> The stack grows and shrinks linearly as functions are called and return.

## 5.4 命令行参数
```c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[]){
    int i;
    for(i=0;i<argc;i++){
        printf("%s\t",argv[i]);
    }
    printf("\n");
    return 0;
}
```
## 5.5 环境变量
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

extern char **environ;

int main(int argc,char *argv[]){
    char **ep;
    for(ep=environ;*ep != NULL;ep++){
        puts(*ep);

    }
    return 0;
}
```
此外：
```c
#include <stdlib.h>
char *getenv(const char *name);
char *putenv(char *string);
int setenv(const char *name,const char *value,int overwrite);
int unsetenv(const char *name);
```

# 6. 内存分配
进程可以通过堆来给动态数据结构分配内存。堆的大小通过调整*program break*的位置来决定。

>传统UNIX系统提供了两个操作*program break*的system call.
```c
#inlcud <unistd.h>
int brk(void *end_data_segment);
//系统调用brk会将program break设置为参数end_data_segment所指定的位置。
void * sbrk(intptr_t increment);
//Retrurns previois program break on seccess or -1 on error
```

## 6.1 malloc() and free()
1. malloc
```c
#include <stdlib.h>
void *malloc(size_t size);
```
malloc 返回类型为void* ，因此可以将其复制给任意类型的C指针。malloc()返回内存块通常采用字节对齐方式。

2. free
```c
#include <stdlib.h>
void free(void *ptr);
```
通常,free操作没有改变program break的位置，而是将这块内存增加到空闲列表中。供后续的malloc（）使用。原因如下：
* 通常被释放的内存位于堆的中间，而非堆的底部，所以改变program break的位置不可行。
* 减小了调用sbrk()的次数，降低系统的开销。

![](/screenshots/malloc_return.jpg)
malloc分配内存实现原理是：分配内存前，先去查空闲内存块表，如果有和请求大小相当的内存，则直接返回。并在空闲内存表上标记使用。此外，malloc分配内存块时，会额外分配几个字节来存放记录这块内存大小的整数值。通常是存在内存块的起始处，函数实际返还的内存地址则在这一记录长度字节之后。

![](/screenshots/free_ptr.png)
使用free释放内存块后，free会使用内存块本身的空间来存放链表指针。

## 6.2 malloc 调试工具
## 6.3 对上分配内存的其他方法
### 6.3.1 calloc() 和 reallloc（）
### 6.3.2 分配对齐的内存
### 6.3.3 在堆栈上分配内存
# 7. 系统和进程信息
## 7.1 /proc文件系统
>The /proc  file system exposes a range of kernel information to application programs.

部分子目录用途：

| 目录 | 目录中文件表达的信心 |
| --- | --- |
| /proc  |  各种系统信息  |
| /proc/net | 有关网络和套接字的信息  |
| /proc/sys/fs  | 文件系统相关设置  |
| /proc/sys/kernel  | 各种常规的内核设置  |
| /proc/sys/net | 网络和套接字的设置 |
| /proc/sys/vm  | 内存管理设置 |
| /proc/sysvipc | System V IPC 对象信息 |


# 8. 文件IO缓冲
## 8.1 内核缓冲
### 8.1.x 控制内核缓冲
## 8.2 stdio缓冲
# 9. 

# 参考资料
1. [CS 306 - Linux/UNIX Programming Web Resources](https://gist.github.com/fffaraz/2b86cf1f1a1f9564c51b)
2. [Linux/UNIX系统编程手册](https://book.douban.com/subject/25809330/)
3. [Unix/Linux编程实践教程](https://book.douban.com/subject/1219329/)
4. [C++静态库与动态库](https://www.cnblogs.com/skynet/p/3372855.html)

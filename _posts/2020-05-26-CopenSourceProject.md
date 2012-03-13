---
layout: post
title: C语言开源项目学习笔记
tag: C语言
key: SouProject20200526
---

他山之石，可以攻玉。

学习和阅读一些优秀、轻量的C语言开源项目，提高自身代码水平，规范编程习惯。
<!--more-->

# cJson
学习记录基于Milo Yip的《从零开始的JSON库教程》为主，部分网络资源为辅。
## 什么是Json？
Json是一种数据格式，可用于前后端交换数据。
## 编译json库教程代码
基于教程提供的git源码库，可以fork后直接clone到本地，执行如下步骤编译进行练习和测试：
```s
cd json-tutorial/tutorial01
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=Debug ..
make
```
### CMake
> CMake is an open-source, cross-platform family of tools designed to build, test and package software. --[CMake](https://cmake.org/)

通常使用CMake需要配置CMakeList.txt即可，我们教程的CmakeList.txt如下所示：
```s
#cmake 最低版本要求
cmake_minimum_required (VERSION 2.6)
#工程名称
project (leptjson_test C)
#设置C编译选项
if (CMAKE_C_COMPILER_ID MATCHES "GNU|Clang")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -ansi -pedantic -Wall")
endif()

add_library(leptjson leptjson.c)
#编译源码生成目标
add_executable(leptjson_test test.c)
#添加链接库
target_link_libraries(leptjson_test leptjson)
```
## 单元测试
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
static int main_ret=0;
static int test_count=0;
static int test_pass=0;

#define EXPECT_EQ_BASE(equality , expext, actual,format) \
do {\
    test_count ++；\
    if(equality)\
        test_pass++;\
    else{\
        fprintf(stderr,"%s:%d:expect: "format"actual: " format"\n",__FILE__, __LINE__, expect, actual);\
        main_ret=1;
    }\
}while(0)

#define EXPECT_EQ_INT(expect, actual) EXPECT_EQ_BASE((expect) == (actual),expect,actual,"%d")
```
通常单元测试除了测试正常的结果外，还应加入一些不合语法的测试。
## 宏的编写技巧
```C
#define M() do{ a(); b();}while(0)

if(cond)
    M();
else
    c();
```
## Linux下内存泄漏检测方法
valgrind 工具
使用方法
```
valgrind --leak-check=full  ./leptjson_test
```

## Unicode
> 跨平台字符编码标准

### 字符转16进制
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
int main(int argc,char**argv){
	int i=0;
	char *hex="1ab4";
	char *phex;
	unsigned int u=0;
	unsigned int *pu;
	char ch;
	phex=hex;
	pu=&u;
	
	for(i=0;i<4;i++){
		ch=*phex++;
		*pu <<=4;
		printf("%0x\t",*pu);
		if(ch >= '0' && ch <= '9') *pu|=ch - '0';
		else if (ch >= 'A' && ch <= 'F' ) *pu |= ch-('A'-10);
		else if (ch >= 'a' && ch <= 'f') *pu |= ch -('a' -10);
		else return 1;
		printf("%0x\n",*pu);
	}
	printf("%0x\n",*pu);
	return 0;
	
}
```
## 堆栈与动态数组
压栈
```c
static void* lept_context_push(lept_context* c, size_t size) {
    void* ret;
    assert(size > 0);
    if (c->top + size >= c->size) {
        if (c->size == 0)
            c->size = LEPT_PARSE_STACK_INIT_SIZE;
        while (c->top + size >= c->size)
            c->size += c->size >> 1;  /* c->size * 1.5 */
        c->stack = (char*)realloc(c->stack, c->size);
    }
    ret = c->stack + c->top;
    c->top += size;
    return ret;
}
```

出栈
```c
static void* lept_context_pop(lept_context* c, size_t size) {
    assert(c->top >= size);
    return c->stack + (c->top -= size);
}
```

将值存储到堆栈
```c
#define PUTC(c, ch)         do { *(char*)lept_context_push(c, sizeof(char)) = (ch); } while(0)
#define PUTS(c, s, len)     memcpy(lept_context_push(c, len), s, len)
```

## 参考资料
- [我的JSON学习项目代码](https://github.com/Jiangdiii/json-tutorial)
- [Wiki-pedia JSON](https://en.wikipedia.org/wiki/JSON)
- [从零开始的 JSON 库教程](https://zhuanlan.zhihu.com/p/22457315)
- [cJSON download](https://sourceforge.net/projects/cjson/)
- [CMake 从入门到应用](https://aiden-dong.github.io/2019/07/20/CMake%E6%95%99%E7%A8%8B%E4%B9%8BCMake%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E5%BA%94%E7%94%A8/)

# 简单数据库的实现（基于C）


## 参考资料 
- [How Does a Database Work?](https://cstack.github.io/db_tutorial/)

# TinyHTTP
> tinyhttpd 是一个简易的 http 服务器，支持CGI。代码量少，非常容易阅读，十分适合网络编程初学者学习的项目。 麻雀虽小，五脏俱全。在tinyhttpd中可以学到 linux 上进程的创建，管道的使用。linux 下 socket 编程基本方法和http 协议的最基本结构。 --[tinyhttpd 阅读与分析](https://jacktang816.github.io/post/tinyhttpdread/)


由于这个项目代码只有不到500行，而且比较简单。网上也有很多对TinyHttp作解读的教程或者博文，所以这里不再做重复性赘述。

测试时，确保本机安装perl-cgi。
```s
sudo yum -y install perl-CGI
[jiangdi@jiangdi Tinyhttpd]$ sudo yum -y install perl-CGI
Package perl-CGI-3.63-4.el7.noarch already installed and latest version
Nothing to do
```

## 程序流程图
![程序运行流程](https://jacktang816.github.io/img/tinyhttpd/tinyhttpd-work-flow.png)
## HTTP 报文格式
![HTTP报文格式](https://img-blog.csdn.net/20150526201634674)
## 参考资料
- [Tiny HTTPd's tiny homepage](http://tinyhttpd.sourceforge.net/)
- [TinyHttpd](https://github.com/EZLippi/Tinyhttpd)
- [TinyHttpd 源码阅读](https://blog.csdn.net/kongshuai19900505/article/details/79270821?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.nonecase)
- [tinyhttpd 阅读与分析](https://jacktang816.github.io/post/tinyhttpdread/)
- [TinyHTTPd--超轻量型Http Server源码分析](https://blog.csdn.net/wenqian1991/article/details/46011357)
- [如何写一个Web服务器](http://zyearn.github.io/blog/2015/05/16/how-to-write-a-server/)

# 引用
1. [最值得关注的10个C开源项目](https://zhuanlan.zhihu.com/p/30661549)
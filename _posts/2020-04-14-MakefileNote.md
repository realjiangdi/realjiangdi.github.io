---
layout: post
title: Makefile Fast快速笔记
tags: 效率
key: makefnote
---
Makefile 是一种自动化编译工具。在Makefile文件中编写好编译命令后，输入`make`即可执行编译。Makefile可以帮助我们简化对**大型项目工程**的编译工作。通常用于Linux、Unix等平台环境下。能否熟练掌握Makefile，也能从侧面反映我们是否具备大型软件项目的能力。

![Title Photo](https://img.fotocommunity.com/national-geographic-winner-egypt-2010-acee709f-0cdd-4d9d-ade2-9f7843ec80b7.jpg?height=1080 "girfa")

这篇快速笔记主要分为三个部分：首先阐述部分**易忘**的概念、其次结合用例解释部分用法、最后补充几个工作中的高级用法。

<!--more-->

>Makefile is a program building tool which runs on Unix, Linux, and their flavors. It aids in simplifying building program executables that may need various modules. 

# 修订记录

| 时间 | 修订内容 | 修订人 |
| :------ |:--- | :--- |
| 2020-04-14 | 完成初步框架搭建； | JiangDi |
| 2020-04-15 | 补充基本概念； | JiangDi |
| 2020-04-16 | 补充基本概念； | JiangDi |

# 1. 引子
一个简单的Makefile文件
```s
say_hello:
	@echo "Hello World" 
```
输入`make`执行：效果如下所示：
```s
$make
$Hello World
```

# 2. 基础概念
## 2.0 Useful Command Line Options
```s
-n #which tells make to display the commands it would execute for a particular target without actually executing them.

```
## 2.1 Basic Makefile Syntax 基础模板
模板：
```s
target: prereq1 prereq2
    command
```
通常目标（target）在`:`前面，`:`后面是生成目标（target）所需要的依赖（prereq）。

## 2.2 Explicit Rules 隐含规则

## 2.3 Wildcards 通配符
```makefile
SRCS := ${wildcards *.c}
BINS := $(SRCS:%.c=%)# Substitution Reference
```
## 2.4 Variables 变量
变量通常用于存储常量字符串（用于命令行中）或者环境变量。
```makefile
# 变量赋值
## 简单字符串
CC := gcc
MKDIR := mkdir -p
## 变量
SRC := *.c

# 变量调用
$(variable-name)
${variable-name}
```

变量类型可以分为递归变量和拓展变量：
1. 拓展变量
   通常可以使用`:=`赋值。
2. 递归变量



**Automatic Variables（自动变量）:** 
```s
$@ #The filename representing the target.
# 通常代表target，常用。
$% #The filename element of an archive member specification.
$< # 通常代表第一个依赖，常用。
$? #The names of all pre that are newer than target.
# 通常代表所有比目标更新的依赖。
$^ #The filenames of all pre.
# 通常代表所有依赖
$+
$*
```
Usage Examples:
```makefile
count_words:count_words.o counter.o lexer.o -lfl
    ${CC} $^ -O $@
count_words.o: count_cords.c
    ${CC} -c $<
counter.o: counter.c
    ${CC} -c %<
```
**VPATH 与 vpath 变量**

给变量`VPATH`加值后，makefile会自动搜索target或者prerequisites。`VPATH`的设置解决了文件搜索的问题。如果想要更精确的搜索路径下的某种类型（后缀）文件，可以使用`vpath`。

```makefile
VPATH = src

vpath %.c src
vpath %.h include
```
## 2.5 模式匹配规则
示例：
```makefile
%:%.sh
    cat $< > $@
    chmod a+x $@

%:%.c
    ${CC} $^ ${LOADLIBES} ${LDLIBS} -o $@

#static pattern
${BINS}:%.o:%.c
    ${CC} -c ${CFLAGS} $< -o $@

```
## 2.6 库的使用和创建（待完善）
## 2.7 条件判断
```makefile
ifdef COMSPEC
    PATH := ;
else
    PATH := :
endif

ifeq (a,a)
endid
```

## 2.8 函数创建
在Makefile中创建函数的示例：
```makefile
define hello_world
    @echo "Hello World"
    @echo "Hello ${1}"
    @echo "Bye ${1}"
endef
$(call hello_world jiangdi,didi)
```
执行函数使用`call`调用
```
$(call <name_of_function> param1,param2,param3)
```
## 2.9 shell命令调用（待完善）

## 2.10 动态库和静态库的链接



# 3. 用例

# 4. 高级用法
## 4.1 大型项目构建（待完善）

### 4.1.1 Make嵌套执行

## 4.2 可兼容Makefile（待完善）

# 5. 其他(待完善)

# 0. 引用
1. [What is a Makefile and how does it work?](https://opensource.com/article/18/8/what-how-makefile)
2. [GNU MAKE](https://www.gnu.org/software/make/manual/make.pdf)
3. [Unix Makefile Tutorial](https://www.tutorialspoint.com/makefile/index.htm)
4. [跟我一起写makefile](https://wangpengcheng.github.io/2019/07/06/write_makefile_with_me/)
5. [makefile中的条件判断ifeq、ifneq、ifdef](https://blog.csdn.net/nyist327/article/details/42552743)
6. [Define your own function in a Makefile](https://coderwall.com/p/cezf6g/define-your-own-function-in-a-makefile)
7. [Linux学习笔记——例说makefile 索引博文](https://blog.csdn.net/xukai871105/article/details/37083675)
8. [Linux下Makefile中动态链接库和静态链接库的生成与调用](https://blog.csdn.net/u011964923/article/details/73297443)
9. [makefile 中的 指定库和头文件的路径](https://blog.csdn.net/sunxiaopengsun/article/details/79012249)
10. [Linux学习笔记——例说makefile 头文件查找路径](https://blog.csdn.net/xukai871105/article/details/36476793)
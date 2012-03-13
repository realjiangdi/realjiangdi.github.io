---
layout: post
title: 嵌入式系统高级C语言编程
tags: C语言
key: embededcnote
---
这篇博文是基于凌明老师的**嵌入式系统高级C语言**做的课程笔记。同时结合自身工作经验将一些C语言的用法实例化，加深自己对C语言的理解。亦方便自己复习。同时摘有K&R版本C语言的部分笔记

<!--more-->

# 修订记录

| 时间 | 修订内容 | 修订人 |
| :------ |:--- | :--- |
| 2020-04-14 | 完成初步框架搭建； | JiangDi |
| 2020-05-25 | 补充K&R版本C语言笔记（指针和数组）； | JiangDi |


# 0. 简介

# 1. 用例

# K&R C Bible 笔记
## Pointers and Arrays
> A pointer is a variable that contains the address of a variable.

some complicated Declarations:
```c
char **argv;
//pointer to pointer to char
int (*daytab)[10];
// pointer to array[10] of int
int *daytab[10];
//array[10] of pointer to int
void *comp;
//function returning pointer to void
void (*comp)();
//pointer to function return void
```
## Structure
> A structure is a collection of one or more variables, possible of different types, grouped together under a single name for convenient handling.

Pointers to structure are so frequently used that an alternative notation is provided as a shorthand. If p is a pointer to a Structure, then 

p->member

```c
struct {
    int len;
    char *str;
}*p;
++p->len;
//increments len , not p, because the implied parenthesization is ++(p->len)
(p++)->len;
//increments p afterward
```


# 引用
1. [嵌入式系统高级C语言编程](https://www.youtube.com/watch?v=col5_Hesz5E&list=PLE-H-rR7yAFuh2d0uH4W5BTI4tury3LFi)
2. [C语言K&R_CBible](http://mef-lab.com/osnove-2016/C-Programming-Ebook.pdf)
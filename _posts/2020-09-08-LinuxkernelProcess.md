---
layout: post
title: Linux 内核线程
tags: Linux
key: LinuxKernelProcess0908
---

Linux 内核线程 学习Note

<!--more-->

| 修订时间   | 修订内容                                   | 修订人 |
| :--------- | :----------------------------------------- | :----- |
| 2020-09-04 | 搭建文章框架；<br>内核线程创建/销毁/实例； | JD     |

***

# 内核线程

Linux 内核线程只运行在内核态。

## 内核线程的创建

内核线程的创建可以通过以下两个函数接口实现：
- `kthead_create`
  - 创建一个新的内核线程；
  - 需要使用`wake_up_process`启动内核线程；
- `kthead_run`
  - 创建一个新的内核线程后，立即启动；

## 内核线程的销毁

线程销毁需要调用`kthread_stop`函数；

## 内核线程与用户态通信


## 参考链接
1. [linux内核线程的创建与销毁](https://blog.csdn.net/lijzheng/article/details/23127777?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.edu_weight&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.edu_weight)
2. [linux 创建内核线程](https://blog.csdn.net/lemontree1945/article/details/73089444)
3. [linux内核线程 [创建]](https://blog.csdn.net/haopeng123321/article/details/54880885)
4. [内核驱动之内核线程示例](https://blog.csdn.net/chengm8/article/details/7945519)
5. [Linux 内核线程及普通进程总结](http://abcdxyzk.github.io/blog/2018/01/10/kernel-task-thread/)
6. [Linux内核线程kernel thread详解--Linux进程的管理与调度（十）](https://blog.csdn.net/gatieme/article/details/51589205)
7. [内核与用户层通信之四种方法](https://blog.csdn.net/vertor11/article/details/79622694)

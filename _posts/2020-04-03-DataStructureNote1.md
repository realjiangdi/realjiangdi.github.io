---
layout: post
title: 数据结构与算法 Part I
tags: 数据结构
---
数据结构的基本概念和链表的代码实现
<!--more-->

# 概要

# 0. 这篇笔记的目的

1. 首先，这篇笔记不是重复造轮子，知识都会遗忘，轮子造得再大再厚也没用。有的放矢的做笔记比无意义的摘抄效果更好。
2. 其次，这篇笔记得目的在于用自己的话解释一些晦涩的概念，也就是说自己一定要思考和总结。代码全部自己实现一遍。
3. 最后，学习并非一蹴而就，数据结构和算法的学习更是如此，**温故而知新**的道理老祖宗都唠叨了几万遍了，自己也吃了很多亏，一定要定期回头看。

## 0.1 修订时间：

| 时间 | 修订内容 | 修订人 |
| :------ |:--- | :--- |
| 2020-04-02 | 完成初稿第一章；<br>完成初稿第二章； | JiangDi |
| 2020-04-04 | 完成顺序存储代码实现；<br>完成链式存储代码实现； | JiangDi |
| 2020-04-05 | 完成单向链表反转代码实现；<br>完成链表中环检测代码实现；<br>完成两个有序链表合并代码实现； | JiangDi |
| 2020-04-07 | 完成删除链表倒数第N个点代码实现；<br>完成链表获取链表中间节点代码实现； | JiangDi |

# 1.什么是数据结构？

日常生活中，一般考试结束后，老师会统计一个班学生的各科成绩，制作成表格方便掌握班里学生的学习情况。通常一个学生有好几个科目的成绩，一个班又有多个学生。学生和科目，班级和学生，彼此之间的关系存在一种数据结构。
>再举一个工作中的例子，比如我司的网络设备，如果某用户终端接入我司设备后，相应用户终端的MAC和IP地址等信息会被记录下来。接入网络设备的用户数据，我们需要选择合适的数据结构去定义他们的存储结构，以方便我们对这些数据进行管理，和一些业务功能的实现（比如我们需要删除某个用户的数据，如何能快速的查找到需要删除的用户？）。

**通俗的来说，数据结构就是一组数据的存储的形式和结构。**
>* 数据存储的形式可以分为顺序存储，链式存储
>* 数据的彼此关系可以分为集合，线性，树形和图形等。

更具体一点来说，我们学数据结构就是要掌握一些常用的数据结构：**线性表，队列，栈和二叉树**等等。

## 算法与数据结构是什么样的关系？
>数据结构和算法是相辅相成，算法需要作用在特定的数据结构之上。

# 2.算法的评判指标有哪些？
数据结构与算法的核心是为了如何让代码运行效率更高并如何合理利用存储空间。
## 2.1算法空间复杂度：
> S(n)=O(f(n))

## 2.2算法时间复杂度
> T(n）=O(f(n)) 通常使用大写O()来体现时间复杂度。

为了表示代码在不同情况下的时间复制度，通常可以使用如下三项指标判断算法的时间复杂度:
1. 最好时间复杂度
2. 最坏时间复杂度
3. 平均时间复杂度

# 3. 线性表
>若干个数据元素的有序序列。

## 3.1 顺序存储
顺序存储指用一段地址连续的存储单元依次存储数据元素。

通常对于一个线性顺序存储的数据，我们需要具备如下函数接口操作数据:
- 初始化一个线性表
- 遍历一个线性表
- 判断线性表是否为空
- 清空线性表
- 获取列表长度
- 往列表第N个位置插入一个数据
- 删除列表某个位置的值
- 查询某个位置的值
- 查询某个值的位置


```c
Status InitList(pList L);
Status ListShow(pList L);
Status ListEmpty(List L);
Status ClearList(pList L);
Status GetLength(pList L);
Status InsertElem(pList L, int N, int value);
Status DeleElem(pList L, int N);
Status LocateElem(pList L, int N,int *value);
Status FindLocate(pList L, int value,int *N);
```

定义线性表顺序存储的数据结构:
```c
#define MAXSIZE 20
typedef int ElementType;
typedef struct {
    ElementType data[MAXSIZE];
    int length;
}List,*pList;
```
具体代码实现如下：[C语言代码实现](https://raw.githubusercontent.com/Jiangdiii/Data_structure_c_programming/master/dhsjjg/ch3/List.c)

### 顺序存储特点：
---
1. 数组支持随机访问，根据下标随机访问时间复杂度为O(1)。
2. 数组的插入和删除可能需要移动大量元素

什么是标记清楚垃圾算法？

## 3.2 链式存储

> 链式存储指用指针将一些零散的内存串联到一起。每个存储单元除了需要储存数据的信息，还需要存储它的后继存储单元的内存地址。如果是双向链表，还需要存储上一个存储单元的内存地址。

### 3.2.1 单链表
---
单链表的数据结构如下所示：
```c
typedef int ElemType;
typedef struct node{
ElemType data;
struct node *next;
}Node;
typedef struct Node *LinkList;
```
对于单链表，基本的操作都有：
- 初始化链表（创建单链表）：头插法和尾插法
- 链表节点插入: 
- 链表节点删除:
- 链表节点查询:
- 遍历打印链表:
- 判断链表是否为空：若为空则返回0，非空返回链表长度个数
- 获取链表长度：

代码实现：[C语言实现](https://raw.githubusercontent.com/Jiangdiii/Data_structure_c_programming/master/dhsjjg/ch3/single_list.c)

### 3.2.2 循环链表
---

### 3.2.3 双向链表
---
单链表的数据结构如下所示：
```c
typedef int ElemType;
typedef struct node{
ElemType data;
struct node *prior;
struct node *next;
}Node;
typedef struct Node *LinkList;
```

## 3.3链表LeetCode操作题

### 3.3.1单链表反转

问题描述：
>输入 1-2-3-4-5-NULL  
>输出 5-4-3-2-1-NULL

思路：
>迭代法：单链表反转的核心在于每一个节点的nxet指向上一个节点。具体实现我们需要如下操作：
>>先使用`temp`临时储存current->next，保存下一个节点的信息。接着对节点进行反转指向`pre`，`pre`保存的是上一个节点的位置。然后`pre`指向刚刚完成反转的节点，在将`current`指向`temp`，即下一个节点。如此遍历一遍链表后，即完成单链表的反转。

迭代法代码实现：
```c
Status LinkRevert(LinkList L){
    LinkList pre=NULL;
    LinkList current=L->next;
    LinkList temp=NULL;
    while(current){
        temp=current->next;
        current->next=pre;
        pre=current;
        current=temp;
    }
    L->next=pre;
    return OK;
}
```
### 3.3.2链表中环检测

问题描述：
>给定一个链表，判断链表中是否有环。

思路：
- 哈希表
>

- 双指针法
>

双指针法代码实现：
```c
Status HasCircle(LinkList L){
    LinkList fast,slow;
    if(L==NULL){
        printf("The ptr is Null\N);
        return ERROR;
    }
    if(L->next == NULL)
    {
        printf("There is No circle in this LinkedList\n");
        return OK;
    }
    slow = L->next;
    fast = L->next->next;
    while(slow != fast){
        if(fast == NULL || slow == NULL)
        {
            printf("There is No circle in this LinkedList\n");
            return OK;
        }
        fast = fast->next->next;
        slow = slow->next;
    }
    printf("There is a Circle in this LinkList\n");
    return OK;
}
```
### 3.3.3两个有序链表合并

问题：
>输入 1-3-5 1-2-6  
>输出 1-1-2-3-5-6

思路：
1. 迭代法
2. 递归法

迭代法代码实现：
```c
Status MergeLinkList(LinkList L1,LinkList L2){
    LinkList pre=L1,T1,T2;
    if(L1 == NULL || L2 == NULL){
        printf("Prt is NUll\n");
        return ERROR;
    }
    T1=L1->next;
    T2=L2->next;
    while(T1 && T2){
        if(T1->data < T2->data){
            pre->next=T1;
            T1=T1->next;
        }
        else
        {
            pre->next=T2;
            T2=T2->next;
        }
        pre=pre->next;
    }
    pre->next = T1 == NULL ? T2:T1;
}

```
### 3.3.4删除链表倒数第n个节点

问题：
> 给定一个链表，删除链表的倒数第n个节点，并且返回头节点。  
如：1->2->3->4->5 ，若n=2。 则输出应为： 1-2->3->5

思路：
1.两次遍历法
2.一次遍历法

**一次遍历法**代码实现：
```c
Status RemoveFromEnd(LinkList L, int n){
    LinkList T1,T2;
    T1=L->next;
    T2=L+sizeof(Node)*(n+1);
    while(T2->next){
        T1=T1->next;
        T2=T2->next;
    }
    T2=T1->next;
    T1->next = T2->next;
    free(T2);
    return OK;
}
```

### 3.3.5求链表的中间节点

问题：
>给定一个链表，求出链表的中间节点
>> 如1->2->3->4->5 则中间节点为3   
>> 如1->2->3->4->5->6 则中间节点为3或者4

思路：
1.遍历法
2.快慢指针法

快慢指针代码实现：
```
Status MiddleNodeGet(LinkList L){
    LinkList fast=L;
    LinkList slow=L;
    while(fast != NULL && fast->next != NULL){
        fast=fast->next->next;
        slow=slow->next;
    }
    printf("Find Middle Node is %d",slow->data);
    return OK;
}
```
# 0. 参考资料

1. 大话数据结构
2. 数据结构与算法之美
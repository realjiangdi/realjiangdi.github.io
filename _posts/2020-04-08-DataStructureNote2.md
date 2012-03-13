---
layout: post
title: 数据结构与算法 Part II
tags: 数据结构
key: dsn2
---
栈和队列以及代码实现
<!--more-->

# 概要

# 0.1修订时间：

| 时间 | 修订内容 | 修订人 |
| :------ |:--- | :--- |
| 2020-04-08 | 完成文章框架搭建； | JiangDi |
| 2020-04-09 | 完成栈和队列代码实现； | JiangDi |

# 4.栈
>栈是限定再表尾进行数据的插入和删除操作的线性表。

栈的数据结构
```c
typedef int ElementType;
struct stack{
    ElementType data[MAXSIZE];
    int top;
}Stack;
```
栈的操作函数：
> 1. 栈的初始化
> 2. 栈是否为空
> 3. 栈的深度
> 4. 入栈操作
> 5. 出栈操作
> 6. 清空栈
> 7. 查看栈顶元素

代码：C语言[实现代码](https://raw.githubusercontent.com/Jiangdiii/Data_structure_c_programming/master/dhsjjg/ch4/stack.c)
## 4.1两栈共享空间
两栈共享空间的数据结构
```c
typedef struct shareStack{
    ElementType data[MAXSIZE];
    int top1;
    int top2;
}ShareStack,*pShareStack;
```
共享栈的出栈和入栈操作
```c
/*
参数解释：
sst 传入的结构体指针
select 操作栈的选择 1表示从栈底开始操作 2表示从栈顶开始操作
*/
Status PushShareStack(pShareStack sst,int select){
    if(sst == NULL)
        return ERROR;
    if((sst->top2-sst->top1) <= 1 ){
        printf("Stack is full");
        return ERROR;
    }
    if(select == 1){
        sst->data[sst->top1]=rand()%100+1;
        sst->top1++;
        return OK;
    }
    else if(select ==2){
        sst->top2--;
        sst->data[sst->top2]=rand()%100+1;
        return OK;
    }
    else
        printf("select arguments error\n");
        return ERROR;
}
Status PopShareStack(pShareStack sst,int select){
    if(sst == NULL)
        return ERROR;
    if(select == 1){
        if(sst->top1>0){
            sst->top1--;
            printf("data is %d",sst->data[sst->top1]);
            return OK;
        }
        printf("Stack1 is Null\n");
        return ERROR;
    }
    else if (select == 2){
        if(sst->top2<20){
            printf("data is %d \n",sst->data[sst->top2]);
            sst->top2++;
        }
        printf("Stack2 is Null\n");
        return ERROR;
    }
    else
    {
        printf("select is error\n");
        return ERROR;
    }
    
}
```
## 4.2栈的链式存储结构及其实现
栈的链式数据结构
```c
typedef struct stacknode{
    ElementType data;
    struct stacknode *next;
}StackNode,*LinkStackPtr;

typedef struct linkstack{
    LinkStackPtr top;
    int count;
}LinkStack;
```

链式栈的出栈入栈操作
```c
Status PushLinkStack(LinkStack * lst){
    if(lst == NULL)
        return ERROR;
    LinkStackPtr newNode;
    
    newNode = malloc(sizeof(StackNode));
    newNode->data=rand()%100+1;
    newNode->next=lst->top;
    lst->top=newNode;
    lst->count++;
    return OK;
}

Status PopLinkStack(LinkStack *lst){
    LinkStackPtr temp;
    if(lst == NULL)
        return ERROR;
    if(lst->count <= 0 ){
        printf("Stack is Null\n");
        return ERROR;
    }
    temp = lst->top;
    lst->top=temp->next;
    free(temp);
    return ERROR;
}
```
# 5.递归
简言之，自己调用自己。但需要设置终止条件。

# 6.队列
> 队列是只允许在一端进行插入，而在另一端进行删除操作的线性表。

和前面的数据结构类似，通常我们对队列需要有如下操作：

* 初始化队列
* 销毁队列
* 清空队列
* 入队操作
* 出队操作
* 返回队列个数

顺序队列的基本数据结构：
```c
#define MAXSIZE 20

typedef struct {
    ElementType data[MAXSIZE];
    int front;
    int rear;
}Queue;
```
顺序队列的初始化、入队和出队代码实现：
```c
Status InitQueue(Queue * sq){
    if(sq == NULL)
        return ERROR;
    sq->front=0;
    sq->rear=0;
    return OK;
}
Status EnterQueue(Queue *sq){
    if(sq->rear == MAXSIZE)
        return ERROR;
    sq->data[sq->rear++]=rand()%100+1;
    return OK;
}
Status LeveQueue(Queue *sq){
    if(sq->front == sq->rear)
        return ERROR;
    sq->front++;
    return OK;
}
```

# 6.1 循环队列
循环队列的数据结构和和顺序队列一样，不同在于入队和出队的操作。
```c
Status EnterCircleQueue(Queue *scq){

}
Status LeveCircleQueue(Queue *scq){

}
```
# 6.2 链式队列
链式队列的数据结构
```
typedef struct node{
    ElementType data;
    struct node * next;
}Node,pNode;

typedef struct queue{
    pNode tail;
    pNode head;
    int count;
}Queue;
```
链式队列的入队操作:
```c
pQ->tail->next = newnode;
pQ->tail=newnode;
pQ->count++;
```
链式队列的出队操作：
```c
temp=pQ->head;
pQ->head=temp->next;
free(temp);
pQ->count--;
``` 
# 参考
1. 大话数据结构
2. 数据结构与算法之美
---
layout: post
title: 数据结构与算法 Part V
tag: 数据结构

---

查找与排序
<!--more-->

# 概要

# 0.1修订时间：

| 时间 | 修订内容 | 修订人 |
| :------ |:--- | :--- |
| 2020-04-28 | 完成文章框架搭建； | JiangDi |
| 2020-05-07 | 完成冒泡，简单选择和直接插入排序代码实现；  | JiangDi |
| 2020-04-28 | 完成希尔排序代码实现； | JiangDi |

# 9 查找
## 9.1 顺序查找

## 9.2 二分查找

## 9.3 散列表

## 9.4 哈希算法

# 10 排序
排序算法代码实现：[C Programm](https://raw.githubusercontent.com/Jiangdiii/Data_structure_c_programming/master/dhsjjg/ch7/sort.c)
## 10.1 冒泡排序
```c
void swap(PSortList L,int i,int j){
    L->array[0]=L->array[i];
    L->array[i]=L->array[j];
    L->array[j]=L->array[0];
}
```
初级版冒泡排序：
```c
Status BubleSort1(PSortList L){
    int i,j;

    for(i=1;i<L->length;i++){
        for(j=i;j<L->length;j++){
            if(L->array[i] > L->array[j+1]){
                swap(L,i,j+1);
            }
        }

    }
    return OK;
}
```
正版冒泡排序：
```c
Status BubleSort2(PSortList L){
    int i,j;
    for(i=L->length;i>0;i--){
        for(j=1;j<i;j++){
            if(L->array[j]>L->array[j+1]){
                swap(L,j,j+1);
            }
        }
    }
    return OK;
}
```
优化版冒泡排序：
```c
Status BubleSort3(PSortList L){
    int i,j;
    Status flag=1;
    for(i=L->length;i>0 && flag;i--){
        flag=0;
        for(j=1;j<i;j++){
            if(L->array[j]>L->array[j+1]){
                swap(L,j,j+1);
                flag=1;
            }
        }
    }
    return OK;
}
```
## 10.2 简单选择排序
```c
Status SelectSort(PSortList L){
    int i,j;
    int min;
    for(i=1;i<L->length;i++){
        min = i;
        for(j=i+1;j<=L->length;j++){
            if(L->array[min] > L->array[j]){
               min = j;
            }
        }
        if(min != i)
        {
            swap(L,min,i);
        }
    }
    return OK;
}
```
## 10.3 直接插入排序
```c
Status InsertSort(PSortList L){
    int i,j;
    for(i=1;i < L->length; i++)
        if(L->array[i] > L->array[i+1]){
            L->array[0]=L->array[i+1];
            
            for(j=i;j>0 && L->array[j] > L->array[0]; j--){
                L->array[j+1]=L->array[j];
            }
            L->array[j+1]=L->array[0];
        }
        
    return OK;
}
```
## 10.4 希尔排序
我自己的希尔排序算法：
```c
Status ShellSort(PSortList L){
    int i,j,gap;
    gap=L->length;
    do{
        gap/=2;
        for(i=1;i+gap <= L->length;i++)
            if(L->array[i] > L->array[i+gap]){
                L->array[0]=L->array[i+gap];
                for(j=i;j>0 && L->array[j] > L->array[0];j-=gap){
                    L->array[j+gap]=L->array[j];
                }
                L->array[j+gap]=L->array[0];
            }
    }while(gap>1);
    return OK;
}
```
《大话数据结构》的希尔排序算法：
```c
Status ShellSort1(PSortList L){
    int i, j, gap;
    gap=L->length;
    do{
        gap=gap/2;
        for(i=gap+1;i<=L->length;i++)
            if(L->array[i] < L->array[i-gap]){
                L->array[0]=L->array[i];
                for(j=i-gap;j>0 && L->array[j] > L->array[0];j-=gap)
                    L->array[j+gap]=L->array[j];
                 L->array[j+gap]=L->array[0];    
            }
    }while(gap>1);
}
```
## 10.5 归并排序

## 10.6 快速排序


## 10.X 参考资料
1. [排序：希尔排序（算法）](https://www.jianshu.com/p/d730ae586cf3)
2. [图解排序算法(二)之希尔排序](https://www.cnblogs.com/chengxiao/p/6104371.html)
3. [大话数据结构](https://book.douban.com/subject/6424904//)
4. [数据结构与算法之美](https://time.geekbang.org/column/intro/126)
5. [图解排序算法(四)之归并排序](https://www.cnblogs.com/chengxiao/p/6194356.html)
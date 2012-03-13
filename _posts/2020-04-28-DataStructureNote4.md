---
layout: post
title: 数据结构与算法 Part IV
tag: 数据结构

---

树的基本概念及二叉树的代码实现
<!--more-->

# 概要

# 0.1修订时间：

| 时间 | 修订内容 | 修订人 |
| :------ |:--- | :--- |
| 2020-04-28 | 完成文章框架搭建； | JiangDi |
| 2020-05-06 | 补充数据结构“树”的内容；<br>完成二叉树的创建及遍历（递归）代码实现；  | JiangDi |
| 2020-05-07 | 完成搜索二叉树的创建，查找，插入,删除代码实现； | JiangDi |

# 8. 树
## 8.1 树基本概念
>  tree is a widely used abstract data type (ADT) that simulates a hierarchical tree structure, with a root value and subtrees of children with a parent node, represented as a set of linked nodes. ---[wikepedia](https://en.wikipedia.org/wiki/Tree_(data_structure))

树的基本概念有：结点和根节点，节点之间的关系，树的深度（depth）与层次等。
## 8.2 二叉树
> “A binary tree is a tree data structure in which each node has at the most two children, which are referred to as the left child and the right child.” -[Wikipedia](https://en.wikipedia.org/wiki/Binary_tree)

二叉链表的数据结构：
```c
typedef char ElementType;
typedef int Status;
typedef struct bitree{
    ElementType data;
    struct bitree *lchild;
    struct bitree *rchild;
}Bitree,*pBitree;
```

### 8.2.1 二叉树的建立
这里我们使用的是前序建立二叉树，中序或者后序建立二叉树同理。
```c
Status CreateBiTree(pBitree *T){
    ElementType ch;

    scanf("%c",&ch);

    if(ch == '#'){
        *T=NULL;
        return OK;
    }
    else{
        (*T)=malloc(sizeof(Bitree));
        (*T)->data=ch;
        CreateBiTree(&(*T)->lchild);
        CreateBiTree(&(*T)->rchild);
        return OK;
    }
}
```
### 8.2.2 二叉树的遍历

![Traversals](https://www.cs.cmu.edu/~adamchik/15-121/lectures/Trees/pix/tree1.bmp)

- 前序遍历  
  PreOrder traversal - visit the parent first and then left and right children;(PreOrder - 8, 5, 9, 7, 1, 12, 2, 4, 11, 3)
- 中序遍历  
  InOrder traversal - visit the left child, then the parent and the right child;(InOrder - 9, 5, 1, 7, 2, 12, 8, 4, 3, 11)
- 后序遍历  
  PostOrder traversal - visit left child, then the right child and then the parent;(9, 1, 2, 12, 7, 5, 3, 11, 4, 8)

### 8.2.3 遍历二叉树代码实现（递归）
前序遍历代码实现：
```c
Status PreOrderTraverse(pBitree T){
    if(T!=NULL){
        printf("%c",T->data);
        PreOrderTraverse(T->lchild);
        PreOrderTraverse(T->rchild);
        return OK;
    }
    return ERROR;
}
```
中序遍历代码实现：
```c
Status InOrderTraverse(pBitree T){
    if(T!=NULL){
        InOrderTraverse(T->lchild);
        printf("%c",T->data);
        InOrderTraverse(T->rchild);
        return OK;
    }
    return ERROR;
}
```
后续遍历代码实现：
```c
Status PostOrderTraverse(pBitree T){
    if(T!=NULL){
        PostOrderTraverse(T->lchild);
        PostOrderTraverse(T->rchild);
        printf("%c",T->data);
    }
}
```
### 8.2.4 遍历二叉树代码实现（非递归）

## 8.3 二叉搜索树
二叉搜索树是二叉树的一种，也叫做“有序二叉树”。二叉搜索树在排序、搜索和删除操作上，效率均优于其他数据结构。  
### 8.3.1 二叉搜索树的基本概念
![BST](https://www.cs.cmu.edu/~adamchik/15-121/lectures/Trees/pix/pix03.bmp "BST")

二叉搜索树具有如下特点：
- each node contains one key (also known as data)
- the keys in the left subtree are less then the key in its parent node, in short L < P;
- the keys in the right subtree are greater the key in its parent node, in short P < R;
- duplicate keys are not allowed.

二叉树代码实现如下：[C语言实现](https://raw.githubusercontent.com/Jiangdiii/Data_structure_c_programming/master/dhsjjg/ch6/BSTree.c)
### 8.3.2 二叉搜索树的插入
```c
Status InsertBst(pBitree *T,ElementType n){

    if(!(*T)){
            printf("Insert succed\n");
            (*T)=malloc(sizeof(Bitree));
            (*T)->data=n;
            (*T)->lchild=NULL;
            (*T)->rchild=NULL;
            return OK;
    }
    else if ( (*T)->data < n ){
        return InsertBst(&(*T)->rchild,n);
    }
    else if( (*T)->data > n ){
        return InsertBst(&(*T)->lchild,n);
    }
    else
    {
        printf("dup value cann't insert\n");
        return ERROR;
    }
    
}
```
### 8.3.3 二叉搜索树的查找
```c
Status SearchBST(pBitree T,ElementType n){
    if(!T){
        printf("failed search\n");
        return ERROR;
    }
    else if(T->data == n){
        printf("find it \n");
        return OK;
    }
    else if(n < T->data){
       return SearchBST(T->lchild,n);
    }
    else{
        return SearchBST(T->rchild,n);
    }
}
```
### 8.3.4 二叉搜索树的创建
```
Status CreateBST(pBitree *T,int number){
    int value=0;
 
    for(;0<number;number--){
        value=rand()%100+number;
        InsertBst(&(*T),value);
    }
    return OK;
}
```
### 8.3.4 二叉搜索树的删除
```c
Status DeleteNode(pBitree *T){
    pBitree temp,par;
    if((*T)->lchild ==  NULL){
        temp=*T;
        *T=(*T)->rchild;
        free(temp);
    }
    else if((*T)->rchild == NULL){
        temp=*T;
        *T=(*T)->lchild;
        free(temp);
    }
    else{
        par=(*T);
        temp=(*T)->lchild;
        while(temp->rchild){
            par = temp;
            temp=temp->rchild;
        }
        (*T)->data=temp->data;
        if(par != (*T))
        {
            par->rchild=temp->lchild;
        }
        else{
            par->lchild=temp->lchild;
        }
        free(temp);
    }
    return OK;
}
Status PreOrderTraverse(pBitree T){
    if(T!=NULL){
        printf("%d\t",T->data);
        PreOrderTraverse(T->lchild);
        PreOrderTraverse(T->rchild);
        return OK;
    }
    return ERROR;
}
```
## 8.4 平衡二叉树（AVL 树）

## 8.5 多路查找树（B树）

# 参考资料
1. [大话数据结构](https://book.douban.com/subject/6424904/)
2. [数据结构与算法之美](https://datastructure.xiaoxiaoming.xyz/#/)
3. [8 Useful Tree Data Structures Worth Knowing](https://towardsdatascience.com/8-useful-tree-data-structures-worth-knowing-8532c7231e8c)
4. [Everything you need to know about tree data structures](https://www.freecodecamp.org/news/all-you-need-to-know-about-tree-data-structures-bceacb85490c/)
5. [Binary Trees](https://www.cs.cmu.edu/~adamchik/15-121/lectures/Trees/trees.html)
6. [Leetcode二叉树相关题目汇总，分析，总结](https://zhuanlan.zhihu.com/p/29800274)
7. [c语言实现二叉排序树的插入、查找、删除、打印树](https://blog.csdn.net/biglamp/article/details/77045193)
8. [C语言实现二叉查找树（BST）的基本操作](https://blog.csdn.net/CHENYUFENG1991/article/details/50901205)
9. [Binary search tree. Removing a node](http://www.algolist.net/Data_structures/Binary_search_tree/Removal)
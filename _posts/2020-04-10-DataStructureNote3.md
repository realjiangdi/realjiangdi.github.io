---
layout: post
title: 数据结构与算法 Part III
tag: 数据结构
---
字符串的一些基本操作和算法实现

<!--more-->

# 概要

# 0.1修订时间：

| 时间 | 修订内容 | 修订人 |
| :------ |:--- | :--- |
| 2020-04-10 | 完成文章框架搭建； | JiangDi |
| 2020-04-15 | 完成字符串反转代码实现； | JiangDi |
| 2020-04-21 | 完成最长回文字符串代码实现； | JiangDi |
| 2020-04-22 | 完成最长回文字符串代码实现； | JiangDi |
| 2020-04-24 | 完成无重复最长字符串代码实现； | JiangDi |
| 2020-04-25 | 完成无重复最长字符串代码实现；<br>完成最长公共子串代码实现；<br>完成左旋转字符串代码实现； | JiangDi |


# 7. 字符串
> 字符串是一种常见的数据结构，日常工作中也经常遇到对字符串相关的问题。比如配置文件里的对配置参数读取、字符串之间的比较等等。

## 7.1 字符串基本操作
在C语言下，字符串通常有如下基本操作：
1. 字符串拷贝：
   ```c
   strcpy(d,s);
   strncpy(d,s,n);
   ```
2. 字符串拼接
   ```c
   strcat(p,p1);
   strncat(p,p1,n);
   ```
3. 字符串查找
   ```c
   strchr(p,c);
   strrchr(p,c);
   strstr(p,p1);
   ```
4. 字符串比较
   ```c
    strcmp(p,p1);
    strncmp(p,p1,n);
   ```
5. 字符串长度
   `strlen(s);`
6. 字符串转化
   ```c
   atoi(p);//字符串转化为整型
   atof(p);//字符串转换为double浮点数
   atol(p);//字符串转换为long整型
   ```
7. 字符检查
   ```c
   isalpha();
   isupper();
   islower();
   isdigit();
   ```
   
## 7.2 字符串基本面试题/算法
### 7.2.0 反转字符串
```c
#include "stdio.h"
#include "stdlib.h"
#include "string.h"

#define OK 0
#define Error 1

typedef int Status;
Status StringReverse(char *s);
int main(int argc, char **argv){
    char StringTest1[]="Jiangs";
    puts("before Reverse:\n");
    puts(StringTest1);
    StringReverse(StringTest1);
    puts("After Reverse:\n");
    puts(StringTest1);
    return OK;
}
Status StringReverse(char *s){
    char *s1,*s2;
    char temp;
    s1=s;
    s2=s+strlen(s)-1;
    while(s1<s2){
        temp=*s1;
        *s1++=*s2;
        *s2--=temp;
    }
    return 0;
}
```
Tips:
>And the most important difference is that, we cannot edit the pointer type string. So this is read-only. But we can edit array representation of the string.

### 7.2.1 最长回文子串
```c
Status LongestPalindrome(char * s){
    int i=0;
    int maxLength=0;
    int length=0;
    int start=0;
    length=strlen(s);
    for(;i<length-1;i++){
        printf("i=%d,maxlength=%d\n",i,maxLength);
        expandSearch(s,i,i,&start,&maxLength,length);
        expandSearch(s,i,i+1,&start,&maxLength,length);
    }   
    if(maxLength > 0){
        printf("find the longest\n");
        printf("The Longest Palindrome is %d\n",maxLength);   
        for(i=start;i<maxLength;i++)
            putchar(s[i]);
        putchar('\n');
        return OK;
    }
    printf("There is no Palindrome\n");
    return Error;
}
Status expandSearch(char *s,int left,int right,int *start,int* maxlen,int length){

    while(left >= 0 && right < length  && s[left] == s[right]){
        left--;
        right++;
    }
    if(right - left - 1 > (*maxlen)){
        *start=left+1;
        *maxlen=right-left - 1;
    }
    return OK;
}
```
### 7.2.2 无重复字符的最长子串
```c
Status longestOfLengthSubstring(char *s){
    int i=0,j=0,maxLength=0;
    char *start;
    char *tstart;
    if(strlen(s) < 2 ){
        printf("The length of string is short\n");
        return Error;
    }
    start=s;
    for(i=0;i<strlen(s);i++){
        tstart=s+i;
        for(j=i+1;j<strlen(s);j++){
                if(isExistCharacter(j-i,tstart,s+j)){
                    
                    if(i==0 && maxLength < j-i){
                        maxLength=j-i;
                        start=s+i;
                        i=j-1;
                        
                        break;}
                }
                else if(maxLength < j-i +1){
                    maxLength=j-i+1;
                    start=s+i;
                    i=j-1;
                    break;
                }

        }
    }
    if(maxLength == 0)
        return Error;
    printf("The maxLength is %d\nThe LongestSubString is : ",maxLength);
    for(i=0;i<maxLength;i++){
        putchar(start[i]);
    }
    putchar('\n');
    return OK;
}

Status isExistCharacter(int index,char *start,char *temp){
    int i=0;
    for(;i<index;i++){
        if(start[i]==*temp){
            return Error;
        }
    }
    return OK;
}
```
### 7.2.3 最长公共子序列
```c
Status longestCommonSubsequence(char * text1, char * text2){
    int i;
    int maxLength=0;
    char *s1 = text1;
    char *s2 =text2;

    while(*s1 && *s2){
        if(*s2 == *s1){
            s2++;
            maxLength++;
        }
        s1++;
    }
    if(maxLength == 0){
        printf("failed\n");
        return Error;
    }
    printf("maxLength is %d\n longest common sub string is : ",maxLength);
    for(i=0;i<maxLength;i++)
        putchar(text2[i]);
        
        putchar('\n');
    return OK;
}
```
### 7.2.4 左旋转字符串
```c
Status reverseLeftWords(char* s, int n){
    char *temp;
    char *pStr=s;
    int i=0;
    temp = malloc(n+1);
    strncpy(temp,s,n);
    for(;i<strlen(s)-n;i++)
        *pStr++=*(pStr+n);
    strncpy(pStr+1,temp,n);
    printf("%s",s);
}
```
### 7.2.5 KMP和BM算法
```

```

# 参考资料
1. [数据结构和算法面试题系列—字符串](https://juejin.im/post/5b9513bd6fb9a05d2a1d4cc9#heading-1)
2. [大话数据结构](https://book.douban.com/subject/6424904/)
3. [数据结构与算法之美](https://datastructure.xiaoxiaoming.xyz/#/)
4. [[LeetCode] 5. Longest Palindromic Substring 最长回文子串](https://www.cnblogs.com/grandyang/p/4464476.html)
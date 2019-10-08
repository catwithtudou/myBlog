---
title: Leetcode-1-两数之和
date: 2019-10-08 21:53:54
tags: ["编程练习题"]
categories: "编程练习题"
---

# Leetcode-1-两数之和

## 暴力求解

这个解法没什么好说的，暴力就完事了

```c


/**
 * Note: The returned array must be malloced, assume caller calls free().
 */
int* twoSum(int* nums, int numsSize, int target, int* returnSize){
    int* res;
    res=(int *)malloc(2*sizeof(int));
    *returnSize=0;
    for(int i=0;i<numsSize-1;i++)
    {
        for(int j=i+1;j<numsSize;j++)
        {
            if(nums[i]+nums[j]==target){
                res[0]=i;
                res[1]=j;
                *returnSize=2;
                return res;
            }
        }
    }
    return res;
}


```

## 哈希表

这一种做法来自于官方题解中，它是通过Java语言中的`HashMap`来进行优化

具体思想为

我们知道两数之和的target,target减去其中一个数,那么剩下的那一个数就是我们所要找的元素,而如果哈希表存储了相关元素的索引,那么就可以直接通过哈希表的索引判断所要找的元素是否存在

这也是典型的以空间换时间的做法

而具体方法也有两种

> C语言中没有HashMap类型,所以下面必须自己构建
>
> HashMap中的结构大致是
>
> ![img](https://raw.githubusercontent.com/catwithtudou/photo/master/20191001173539.png)

### 两遍哈希表

采用两遍哈希表,也就是需要迭代两次,第一次是添加到哈希表中,第二次是判断是否存在目标元素的索引

```c

struct Hash_data{
    int key;
    int value;
    struct Hash_data * next;
};
typedef struct Hash_data* HashData;

struct Hash_table{
    struct Hash_data ** head;
    int width;
};
typedef struct Hash_table* HashTable;

//初始化
int Init(HashTable table,int width)
{
    if(width<=0)return-1;
    table->head=malloc(width*sizeof(HashData));
    memset(table->head,0,width *sizeof(HashData));
    if(table->head==NULL)
        return -1;
    table->width=width;
    return 0;
}

//释放空间
void Free(struct Hash_table table)
{
    if(table.head!=NULL)
    {
        for(int i=0;i<table.width;i++)
        {
            HashData element=table.head[i];
            while(element!=NULL) {
                HashData temp = element;
                element = element->next;
                free(temp);
            }
        }
        free(table.head);
        table.head=NULL;
    }
    table.width=0;
}

//哈希位置取模
int Addr(struct Hash_table  table,int key)
{
    int addr = abs(key) % table.width;
    return addr;
}

//增加元素
int Insert(struct Hash_table table,int key,int value)
{
    HashData temp=malloc(sizeof(struct Hash_data));
    if(temp==NULL)return -1;
    temp->key=key;
    temp->value=value;
    //插入位置addr中的头结点
    int addr=Addr(table,key);
    temp->next=table.head[addr];
    table.head[addr]=temp;
    return 0;
}

//查找元素
HashData Find(struct Hash_table table,int key)
{
    int addr=Addr(table,key);
    //头结点
    HashData element=table.head[addr];
    while(element!=NULL)
    {
        if(element->key==key)
        {
            return element;
        }
        element=element->next;
    }
    return NULL;
}

int* twoSum(int* nums, int numsSize, int target, int* returnSize)
{
    *returnSize=0;
    int* res;
    res=malloc(2* sizeof(int));
    struct Hash_table  table;
    Init(&table,100);
    for(int i=0;i<numsSize;i++)
    {
        Insert(table,nums[i],i);
    }
    for(int i=0;i<numsSize;i++)
    {
        int goal=target-nums[i];
        HashData  temp;
        temp=Find(table,goal);
        if(temp!=NULL&&temp->value!=i) //既不能为空,也不能为相同下标
        {
            res[0]=i;
            res[1]=temp->value;
            *returnSize=2;
            return res;
        }
    }
    Free(table);
    return res;
}
```

#### 复杂度分析

- 时间复杂度：O(n)

- 空间复杂度：O(n)

### 一遍哈希表

迭代一次哈希表的做法也就是在添加元素进入表中的时候就开始判断是否存在目标元素的索引,即将暴力求解的第二个循环换成哈希表中的查找

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

struct Hash_data{
    int key;
    int value;
    struct Hash_data * next;
};
typedef struct Hash_data* HashData;

struct Hash_table{
    struct Hash_data ** head;
    int width;
};
typedef struct Hash_table* HashTable;

//初始化
int Init(HashTable table,int width)
{
    if(width<=0)return-1;
    HashData* tmp=malloc(sizeof(HashData)*width);
    table->head=tmp;
    memset(table->head,0,width *sizeof(HashData));
    if(table->head==NULL)
        return -1;
    table->width=width;
    return 0;
}

//释放空间
void Free(struct Hash_table table)
{
    if(table.head!=NULL)
    {
        for(int i=0;i<table.width;i++)
        {
            HashData element=table.head[i];
            while(element!=NULL) {
                HashData temp = element;
                element = element->next;
                free(temp);
            }
        }
        free(table.head);
        table.head=NULL;
    }
    table.width=0;
}

//哈希位置取模
int Addr(struct Hash_table  table,int key)
{
    int addr = abs(key) % table.width;
    return addr;
}

//增加元素
int Insert(struct Hash_table table,int key,int value)
{
    HashData temp=malloc(sizeof(struct Hash_data));
    if(temp==NULL)return -1;
    temp->key=key;
    temp->value=value;
    //插入位置addr中的头结点
    int addr=Addr(table,key);
    temp->next=table.head[addr];
    table.head[addr]=temp;
    return 0;
}

//查找元素
HashData Find(struct Hash_table table,int key)
{
    int addr=Addr(table,key);
    //头结点
    HashData element=table.head[addr];
    while(element!=NULL)
    {
        if(element->key==key)
        {
            return element;
        }
        element=element->next;
    }
    return NULL;
}

int* twoSum(int* nums, int numsSize, int target, int* returnSize)
{
    *returnSize=0;
    int* res;
    res=malloc(2* sizeof(int));
    struct Hash_table  table;
    Init(&table,100);
    for(int i=0;i<numsSize;i++)
    {
        int goal=target-nums[i];
        HashData temp=Find(table,goal);
        if(temp!=NULL&&temp->value!=i)
        {
            res[0]=i;
            res[1]=temp->value;
            *returnSize=2;
            break;
        }
        Insert(table,nums[i],i);
    }
    Free(table);
    return res;
}
```

#### 复杂度分析

- 时间复杂度：O(n)
- 空间复杂度：O(n)

> 若没有Free(table)的这一步骤,执行效率会更快一些
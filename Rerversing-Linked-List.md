---
title: Rerversing Linked List
date: 2019-07-04 21:17:19
tags: [数据结构,线性结构]
categories: "编程练习题"
---

### Rerversing Linked List

02-线性结构3 Reversing Linked List (25 分)



Given a constant *K* and a singly linked list *L*, you are supposed to reverse the links of every *K* elements on *L*. For example, given *L* being 1→2→3→4→5→6, if *K*=3, then you must output 3→2→1→6→5→4; if *K*=4, you must output 4→3→2→1→5→6.

 Input Specification:

Each input file contains one test case. For each case, the first line contains the address of the first node, a positive *N* (≤105) which is the total number of nodes, and a positive *K* (≤*N*) which is the length of the sublist to be reversed. The address of a node is a 5-digit nonnegative integer, and NULL is represented by -1.

Then *N* lines follow, each describes a node in the format:

```
Address Data Next
```

where `Address` is the position of the node, `Data` is an integer, and `Next` is the position of the next node.

 Output Specification:

For each case, output the resulting ordered linked list. Each node occupies a line, and is printed in the same format as in the input.

Sample Input:

```in
00100 6 4
00000 4 99999
00100 1 12309
68237 6 -1
33218 3 00000
99999 5 68237
12309 2 33218
```

 Sample Output:

```out
00000 4 33218
33218 3 12309
12309 2 00100
00100 1 99999
99999 5 68237
68237 6 -1
```

#### 思路

起初看到这道题的示例,真没反应过来,后来才看懂,理解力有点差劲.

一开始的思路是想通过线性结构,而动态链表里的改变位置比较麻烦不容易实现反转,所以我采用了静态链表也就是用数组的方式去交换位置从而实现反转

但是地址始终是一个五位数的非负数,每次寻找位置的时候会比较麻烦,而且在将结点相连的时候也会比较繁琐.所以我运用的是哈希表的思想,即把位置直接使用到数组的索引位置,但是反转的时候可能是我没有理解到反转的函数(自己以为用两个数组从反转位置相两边寻找然后存入另外一个位置序列中),题目最终也只是部门正确

最终反转的时候运用了reverse函数,题目才完全正确

-----------------------------更新------------------------------

后来查询一些人的答案发现这道题也可以用单链表即动态链表来实现,主要逆转的思路比较困难,下面给出伪代码以供大家理解

大致过程是,在头结点后两位设置二个标志结点new,old,然后循环开始,首先设置一个在old后面的temp结点,然后将old结点下一位指向前一位(即是new结点),然后new结点向后进一步(即等于old结点),然后old结点也向后进一步(即temp结点),这样每次循环就逆转了一位,且old结点指向逆转长度后的结点,最后当循环的次数与要求的逆转长度相等时停止,而此时头结点下一位便指向的new结点,而old结点便是最初new结点的下一位

![](https://raw.githubusercontent.com/catwithtudou/photo/master/20190705211221.png)

```c++
Ptr Reverse(Ptr head,int K)
{
    int count=1;
    Ptr new =head->next;
    Ptr old =new->next;
    while(count<K)
    {
        Ptr temp =old->next;
        old->next = new;
        new = old;
        old = temp;
        count++;
    }
    head->next->next = old;
    return new;
}
```



#### 答案

```C++
//以下是部分正确的答案
#include <bits/stdc++.h>
#include <iostream>
#define MaxSize 1000010
using namespace std;
struct node
{
    int data;
    int next;
}node[MaxSize];

int  List[MaxSize];
int  ReList[MaxSize];
int main()
{
    int First,n,k;
    cin>>First>>n>>k;
    int Address,Data,Next;
    for(int i=0;i<n;i++)
    {
        cin>>Address>>Data>>Next;
        node[Address].data=Data;
        node[Address].next=Next;
    }
    int j=0;
    int p=First;
    //按顺序连接结点
    while(p!=-1)
    {
        List[j++]=p;
        p=node[p].next;
    }
    int i=0;
    //进行反转
    for(int j=k-1;j>-1;j--)
    {
        ReList[i++]=List[j];
    }
    for(int j=k;j<n;j++)
    {
        ReList[i++]=List[j];
    }

    for(i=0;i<n-1;i++)
    printf("%05d %d %05d\n",ReList[i],node[ReList[i]].data,ReList[i+1]);
    printf("%05d %d -1\n",ReList[i],node[ReList[i]].data);
    return 0;
}


//以下是全部正确的答案
#include<iostream>
#include<stdio.h>
#include<algorithm>    ///使用到reverse 翻转函数
using namespace std;
 
#define MAXSIZE 1000010   ///最大为五位数的地址
 
struct node    ///使用顺序表存储data和下一地址next
{
   int data;   
   int next;
}node[MAXSIZE];
 
int List[MAXSIZE];   ///存储可以连接上的顺序表
int main()
{
    int First, n, k;  
    cin>>First>>n>>k;   ///输入头地址 和 n，k；
    int Address,Data,Next;
    for(int i=0;i<n;i++)
    {
        cin>>Address>>Data>>Next;
        node[Address].data=Data;
        node[Address].next=Next;
    }
 
    int j=0;  ///j用来存储能够首尾相连的节点数
    int p=First;   ///p指示当前结点
    while(p!=-1)
    {
        List[j++]=p;
        p=node[p].next;
    }
    int i=0;
    while(i+k<=j)   ///每k个节点做一次翻转
    {
        reverse(&List[i],&List[i+k]);
        i=i+k;
    }
    for(i=0;i<j-1;i++)
        printf("%05d %d %05d\n",List[i],node[List[i]].data,List[i+1]);
    printf("%05d %d -1\n",List[i],node[List[i]].data);
    return 0;
}
```
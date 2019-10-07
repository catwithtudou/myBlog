---
title: File-Transfer
date: 2019-08-02 13:46:53
tags: "数据结构"
categories: "编程练习题"
---

#  **File Transfer**

We have a network of computers and a list of bi-directional connections. Each of these connections allows a file transfer from one computer to another. Is it possible to send a file from any computer on the network to any other?

### Input Specification:

Each input file contains one test case. For each test case, the first line contains *N* (2≤*N*≤104), the total number of computers in a network. Each computer in the network is then represented by a positive integer between 1 and *N*. Then in the following lines, the input is given in the format:

```
I c1 c2  
```

where `I` stands for inputting a connection between `c1` and `c2`; or

```
C c1 c2    
```

where `C` stands for checking if it is possible to transfer files between `c1` and `c2`; or

```
S
```

where `S` stands for stopping this case.

### Output Specification:

For each `C` case, print in one line the word "yes" or "no" if it is possible or impossible to transfer files between `c1` and `c2`, respectively. At the end of each case, print in one line "The network is connected." if there is a path between any pair of computers; or "There are `k` components." where `k` is the number of connected components in this network.

### Sample Input 1:

```in
5
C 3 2
I 3 2
C 1 5
I 4 5
I 2 4
C 3 5
S
```

### Sample Output 1:

```out
no
no
yes
There are 2 components.
```

### Sample Input 2:

```
5
C 3 2
I 3 2
C 1 5
I 4 5
I 2 4
C 3 5
I 1 3
C 1 5
S
```

### Sample Output 2:

```
no
no
yes
yes
The network is connected.
```

## 题意分析

- 题目大意

  建立n个结点,在建立边之后判断两结点是否能连通,且判断是否有多个顶点

- 这道题主要考察的就是并查集的运用,这里的并查集可以通过数组索引来作为结点位置而数组下标作为结点的父结点.通过并查集性质的Find函数可以找到其结点的父结点,通过判断父结点是否与要判断的结点相同若相同则说明连通,而在其插入边的过程中通过并查集中小树贴到大树上来进行按秩归并,最后通过数组下标为负数的个数来判断有多少个根结点,这道题就解决了

  > 其中Find函数采用了路径压缩的递归方法即将结点的父结点递归为根结点,从而缩小查询的复杂度

## 答案

```c
#include <stdio.h>
#include <stdlib.h>
#include <iostream>
#include <vector>
#include <algorithm>
#include <cmath>
#define MaxSize 10000
using namespace std;
typedef int SetType[MaxSize];
void Initialization(SetType S,int n);
void Input(SetType S);
void Check(SetType S);
void CheckAll(SetType S,int n);
int Find(SetType S,int X);
void Union(SetType S,int Root1,int Root2);
void Initialization(SetType S,int n)
{
    for(int i=0;i<n;i++)
    {
        S[i]=-1;
    }
}
void Input(SetType S)
{
    int u,v;
    int Root1,Root2;
    scanf("%d %d\n",&u,&v);
    Root1=Find(S,u-1);
    Root2=Find(S,v-1);
    if(Root1!=Root2)
        Union(S,Root1,Root2);
}
void Check(SetType S)
{
    int u,v;
    int Root1,Root2;
    scanf("%d %d\n",&u,&v);
    Root1=Find(S,u-1);
    Root2=Find(S,v-1);
    if(Root1==Root2){
        printf("yes\n");
    }else printf("no\n");
}
void CheckAll(SetType S,int n)
{
    int i,counter=0;
    for(i=0;i<n;i++){
        if(S[i]<0) counter++;
    }
    if(counter==1)
        printf("The network is connected.\n");
    else
        printf("There are %d components.\n",counter);
}
void Union(SetType S,int Root1,int Root2)
{
    if(S[Root2]<S[Root1]){
        S[Root2]+=S[Root1];
        S[Root1]=Root2;
    }else{
        S[Root1]+=S[Root2];
        S[Root2]=Root1;
    }
}
int Find(SetType S,int X)
{
    if(S[X]<0)
        return X;
    else
        return S[X]=Find(S,S[X]);
}
int main()
{
    SetType S;
    int n;
    char in;
    scanf("%d\n",&n);
    Initialization(S,n);
    do{
        scanf("%c",&in);
        switch (in){
            case 'I' : Input(S);break;
            case 'C' : Check(S);break;
            case 'S' : CheckAll(S,n);break;
        }
    }while(in!='S');
    return 0;
}
```
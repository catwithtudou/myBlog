---
title: 'Sort-with-Swap(0, i)'
date: 2019-09-09 08:49:57
tags: "数据结构"
categories: "编程练习题"
---

# Sort with Swap(0, i) 

Given any permutation of the numbers {0, 1, 2,..., *N*−1}, it is easy to sort them in increasing order. But what if `Swap(0, *)` is the ONLY operation that is allowed to use? For example, to sort {4, 0, 2, 1, 3} we may apply the swap operations in the following way:

```
Swap(0, 1) => {4, 1, 2, 0, 3}
Swap(0, 3) => {4, 1, 2, 3, 0}
Swap(0, 4) => {0, 1, 2, 3, 4}
```

Now you are asked to find the minimum number of swaps need to sort the given permutation of the first *N* nonnegative integers.

### Input Specification:

Each input file contains one test case, which gives a positive *N* (≤105) followed by a permutation sequence of {0, 1, ..., *N*−1}. All the numbers in a line are separated by a space.

### Output Specification:

For each case, simply print in a line the minimum number of swaps need to sort the given permutation.

### Sample Input:

```in
10
3 5 7 2 6 4 9 0 8 1
```

### Sample Output:

```out
9
```

## 题目分析

这道题是一道很明显的表排序的问题,其输出的结果就是表排序中每次环中对位置进行交换的次数

而我们需要注意的是环分三种

1.只有1个元素不需要交换

2.环里有n个元素包括0,则需要交换n-1次

3.第i个环里有n个元素,不包括0,先把0换到环里,再进行(n+1)-1次交换即n+1次交换

而这里我们可以利用一个指针数组T来存储元素i在A[T[i]]中存放,即`T[A[i]]=i`,这样我们可以更加容易地找到表排序中每次交换的下一个位置

## 代码实现

```c++
#include <iostream>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <math.h>
#define MAX 100001
using namespace std;
int N;
int A[MAX];


int main()
{
    int *T;
    int num=0;
    scanf("%d",&N);
    T=(int *)malloc(N*sizeof(int));
    for(int i=0;i<N;i++)
        scanf("%d",&A[i]);
    for(int i=0;i<N;i++)
    {
        T[A[i]]=i;
    }
    for(int i=0;i<N;i++)
    {
        if(A[i]==i)continue;
        int flag=0;
        int sinNum=1;
        int j=T[i];
        int value=A[i];
        int k=i;
        while(i!=j)
        {
            if(T[j]==0)flag=1;
            A[k]=A[j];
            k=j;
            j=T[j];
            sinNum++;
        }
        A[k]=value;
        if(flag)sinNum--;
        else sinNum++;
        num+=sinNum;
    }
    printf("%d",num);
    return 0;
}

```


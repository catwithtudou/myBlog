---
title: How-Long-Does-It-Take
date: 2019-09-09 08:47:42
tags: "数据结构"
categories: "编程练习题"
---

# How Long Does It Take

Given the relations of all the activities of a project, you are supposed to find the earliest completion time of the project.

### Input Specification:

Each input file contains one test case. Each case starts with a line containing two positive integers *N* (≤100), the number of activity check points (hence it is assumed that the check points are numbered from 0 to *N*−1), and *M*, the number of activities. Then *M* lines follow, each gives the description of an activity. For the `i`-th activity, three non-negative numbers are given: `S[i]`, `E[i]`, and `L[i]`, where `S[i]` is the index of the starting check point, `E[i]` of the ending check point, and `L[i]` the lasting time of the activity. The numbers in a line are separated by a space.

### Output Specification:

For each test case, if the scheduling is possible, print in a line its earliest completion time; or simply output "Impossible".

### Sample Input 1:

```in
9 12
0 1 6
0 2 4
0 3 5
1 4 1
2 4 1
3 5 2
5 4 0
4 6 9
4 7 7
5 7 4
6 8 2
7 8 4
```

### Sample Output 1:

```out
18
```

### Sample Input 2:

```
4 5
0 1 1
0 2 2
2 1 3
1 3 4
3 2 5
```

### Sample Output 2:

```
Impossible
```

## 题目分析

这是一道典型拓扑排序的题

其中涉及了AOE网络即关键路径问题(绝不允许延误的活动组成的路径)

## 答案

```c++
#include <iostream>
#include <cstdlib>
#include <cstdio>
#include <queue>
using namespace std;
#define MAX 100
#define INFINITY 65535
int N,M,A[MAX][MAX],ECT;

int getMax(int a[])
{
    int value=a[0];
    for(int i=1;i<N;i++)
    {
        if(value < a[i])
            value=a[i];
    }
    return value;
}

int TopSort()
{
    int EarlyTimes[MAX]={0};
    int Indegree[MAX]={0};
    int cnt=0;
    queue<int> q;
    int V;

    for(int i=0;i<N;i++)
     for(int j=0;j<N;j++)
     {
         if(A[i][j]!=INFINITY)
            Indegree[j]++;
     }

     for(int i=0;i<N;i++)
     {
         if(Indegree[i]==0)
            q.push(i);
     }

     while(!q.empty())
     {
         V=q.front();
         q.pop();
         cnt++;
         for(int i=0;i<N;i++)
         {
             if(A[V][i]!=INFINITY)
             {
                 if(EarlyTimes[V]+A[V][i]>EarlyTimes[i])
                 {
                     EarlyTimes[i]=EarlyTimes[V]+A[V][i];
                 }
                 if(--Indegree[i]==0)
                 {
                     q.push(i);
                 }
             }
         }
     }
     ECT=getMax(EarlyTimes);
     if(cnt!=N)
        return 0;
     return 1;
}



int main()
{
    scanf("%d %d",&N,&M);

    for(int i=0;i<N;i++)
        for(int j=0;j<N;j++)
        A[i][j]=INFINITY;

    for(int i=0;i<M;i++)
    {
        int a,b;
        scanf("%d %d",&a,&b);
        scanf("%d",&A[a][b]);
    }

    if(!TopSort())
    {
        printf("Impossible\n");
    }else{
        printf("%d\n",ECT);
    }

    return 0;
}
```


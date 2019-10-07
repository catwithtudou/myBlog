---
title: Saving-James-Bond-Hard-Version
date: 2019-08-20 16:07:06
tags: "数据结构"
categories: "编程练习题"
---

# Saving James Bond - Hard Version

This time let us consider the situation in the movie "Live and Let Die" in which James Bond, the world's most famous spy, was captured by a group of drug dealers. He was sent to a small piece of land at the center of a lake filled with crocodiles. There he performed the most daring action to escape -- he jumped onto the head of the nearest crocodile! Before the animal realized what was happening, James jumped again onto the next big head... Finally he reached the bank before the last crocodile could bite him (actually the stunt man was caught by the big mouth and barely escaped with his extra thick boot).

Assume that the lake is a 100 by 100 square one. Assume that the center of the lake is at (0,0) and the northeast corner at (50,50). The central island is a disk centered at (0,0) with the diameter of 15. A number of crocodiles are in the lake at various positions. Given the coordinates of each crocodile and the distance that James could jump, you must tell him a shortest path to reach one of the banks. The length of a path is the number of jumps that James has to make.

### Input Specification:

Each input file contains one test case. Each case starts with a line containing two positive integers *N* (≤100), the number of crocodiles, and *D*, the maximum distance that James could jump. Then *N* lines follow, each containing the (*x*,*y*) location of a crocodile. Note that no two crocodiles are staying at the same position.

### Output Specification:

For each test case, if James can escape, output in one line the minimum number of jumps he must make. Then starting from the next line, output the position (*x*,*y*) of each crocodile on the path, each pair in one line, from the island to the bank. If it is impossible for James to escape that way, simply give him 0 as the number of jumps. If there are many shortest paths, just output the one with the minimum first jump, which is guaranteed to be unique.

### Sample Input 1:

```in
17 15
10 -21
10 21
-40 10
30 -50
20 40
35 10
0 -10
-25 22
40 -40
-30 30
-10 22
0 11
25 21
25 10
10 10
10 35
-30 10
```

### Sample Output 1:

```out
4
0 11
10 21
10 35
```

### Sample Input 2:

```
4 13
-12 12
12 12
-12 -12
12 -12
```

### Sample Output 2:

```out
0
```

## 题目分析

- 这道题与上次我们做的简易版007其实没有多大的差别,只不过这道题中多加了一些计数和记录步数的操作

  其中使用的是Dijkstra算法来计算最短路径.而其中Dijkstra算法对第一跳做处理。每一个第一跳Dijkstra，如果跳的步少，更新存储鳄鱼下标。如果相同则加入。。然后找到第一步跳的最少的。题目保证唯一。从第一步跳的最少的，再一次Dijkstra来保存步数

  还有就是注意到其中上岸的跳跃也算一步距离	

## 代码实现

```c++
#include <stdio.h>
#include <stdlib.h>
#include <iostream>
#include <queue>
#include <math.h>
#define inf 1000001
int n,d; //结点数目,跳跃距离
int flag=0;//能否上岸标志变量
int exist=0;//标志变量
int newp;
int visited[101];//是否访问
int path[101]; //遍历时的邻接结点
int dist[101]; //该结点与源点的最短路径
struct node
{
    int x;
    int y;
} stu[101];

double dis(int x,int y)
{
    return sqrt(x*x+y*y);
}

int canjump(int x1,int y1,int x2,int y2)
{
    if(sqrt(pow((x1-x2),2)+pow(y1-y2,2))<=d)
    {
        return 1;
    }
    else
    {
        return 0;
    }
}
void init()
{
    int i;
    flag=0;
    for(i=0; i<n; i++)
    {
        visited[i]=0;
        path[i]=-1;
        dist[i]=inf;
    }
}
void dijkstra(int s)
{
    visited[s]=1;
    dist[s]=1;
    path[s]=-2;//第一个结点用-2表示,其余未访问结点用-1表示
    int i;
    newp=s;
    while(1)
    {
        if(50-abs(stu[newp].x)<=d||50-abs(stu[newp].y)<=d)
        {
            dist[newp]++;//还要计算跳到岸上的距离
            flag=1;
            exist=1;
            break;
        }
        for(i=0; i<n; i++)
        {
            if(visited[i]==0&&canjump(stu[newp].x,stu[newp].y,stu[i].x,stu[i].y))
            {
                if(dist[newp]+1<dist[i])
                {
                    dist[i]=dist[newp]+1;
                    path[i]=newp;
                }
            }
        }
        //判断每一步是否为最短路径
        int min=inf;
        for(i=0; i<n; i++)
        {
            if(visited[i]==0&&dist[i]<inf)
            {
                if(dist[i]<min)
                {
                    min=dist[i];
                    newp=i;
                }
            }
        }
        visited[newp]=1;
        if(min==inf)
        {
            break;
        }
    }
}
int main()
{
    int i;
    scanf("%d %d",&n,&d);
    for(i=0; i<n; i++)
    {
        scanf("%d %d",&stu[i].x,&stu[i].y);
    }
    if(d>=50-7.5)
    {
        printf("1\n");
        return 0;
    }
    int firstjump[n];//可第一步跳跃的下标
    int numfirst=0;
    for(i=0; i<n; i++)
    {
        if(d+7.5>=dis(stu[i].x,stu[i].y))
        {
            firstjump[numfirst++]=i;
        }
    }
    int min=inf,index;
    int store[n]; //记录最短路径结点
    int count=0;
    for(i=0; i<numfirst; i++)
    {
        init();
        dijkstra(firstjump[i]);
        if(flag) //若找到路径
        {
            if(dist[newp]<min)  //要寻找最短路径
            {
                min=dist[newp];
                count=0;
                store[count++]=firstjump[i];
            }
            else if(dist[newp]==min)
            {
                store[count++]=firstjump[i];
            }
        }
    }
    if(exist==0)
    {
        printf("0");
        return 0;
    }
    min=inf;
    int indexmin;//如果最短路径相同，则距离源点最近的点唯一（题目保证）
    for(i=0; i<count; i++)
    {
        int index=store[i];
        if(min>dis(stu[index].x,stu[index].y)-7.5)
        {
            min=dis(stu[index].x,stu[index].y)-7.5;
            indexmin=store[i];
        }
    }
    init();
    dijkstra(indexmin);
    printf("%d\n",dist[newp]);
    count=0;
    while(1)
    {
        store[count++]=newp;
        newp=path[newp];
        if(newp==-2)
        {
            break;
        }
    }
    for(i=count-1; i>=0; i--)
    {
        printf("%d %d\n",stu[store[i]].x,stu[store[i]].y);
    }
}


```


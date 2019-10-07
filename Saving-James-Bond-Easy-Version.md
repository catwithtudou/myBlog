---
title: Saving James Bond - Easy Version
date: 2019-08-14 09:50:18
tags: "数据结构"
categories: "编程练习题"
---

# Saving James Bond - Easy Version

This time let us consider the situation in the movie "Live and Let Die" in which James Bond, the world's most famous spy, was captured by a group of drug dealers. He was sent to a small piece of land at the center of a lake filled with crocodiles. There he performed the most daring action to escape -- he jumped onto the head of the nearest crocodile! Before the animal realized what was happening, James jumped again onto the next big head... Finally he reached the bank before the last crocodile could bite him (actually the stunt man was caught by the big mouth and barely escaped with his extra thick boot).

Assume that the lake is a 100 by 100 square one. Assume that the center of the lake is at (0,0) and the northeast corner at (50,50). The central island is a disk centered at (0,0) with the diameter of 15. A number of crocodiles are in the lake at various positions. Given the coordinates of each crocodile and the distance that James could jump, you must tell him whether or not he can escape.

### Input Specification:

Each input file contains one test case. Each case starts with a line containing two positive integers *N* (≤100), the number of crocodiles, and *D*, the maximum distance that James could jump. Then *N* lines follow, each containing the (*x*,*y*) location of a crocodile. Note that no two crocodiles are staying at the same position.

### Output Specification:

For each test case, print in a line "Yes" if James can escape, or "No" if not.

### Sample Input 1:

```in
14 20
25 -15
-25 28
8 49
29 15
-35 -2
5 28
27 -29
-8 -28
-20 -35
-25 -20
-13 29
-30 15
-35 40
12 12
```

### Sample Output 1:

```out
Yes
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

```output
No
```

## 题目分析

- 这道题显而易见是需要采用图中的深度遍历DFS来得到结果,其中图结构在这道题中最适合的是邻接矩阵来表示

## 代码实现

```c
#include <stdio.h>
#include <stdlib.h>
#include <iostream>
#include <queue>
#include <math.h>

const double LAND_RADIUS=15.0/2;
const double LAKE_SIZE=100.0;
const int N=110;

typedef struct{
    double x,y;
}Position;

Position P[N];
bool Visited[N];
int n;
double d;

void Save007();
bool DFS(int V);
bool Jump(int V1,int V2);
bool FirstJump(int V);
bool IsSave(int V);

int main()
{
    scanf("%d%lf",&n,&d);
    for(int i=0;i<n;i++)
    {
        scanf("%lf%lf",&P[i].x,&P[i].y);
    }
    for(int i=0;i<n;i++)
        Visited[i]=false;
    Save007();
}


bool DFS(int V)
{
    Visited[V]=true;
    bool answer=false;
    if(IsSave(V))
        return true;
    for(int i=0;i<n;i++)
    {
        if(!Visited[i]&&Jump(V,i))
            answer=DFS(i);
        if(answer)
            break;
    }
    return answer;
}

void Save007()
{
    bool answer=false;
    for(int i=0;i<n;i++)
    {
        if(!Visited[i]&&FirstJump(i)){
            answer=DFS(i);
            if(answer)
            break;
        }
    }
    if (answer){
        printf("Yes\n");
    }else{
        printf("No\n");
    }
}

bool IsSave(int V)
{
    return (abs(P[V].x)>=50-d||
            abs(P[V].y)>=50-d);
}

bool FirstJump(int V)
{
    return sqrt(P[V].x*P[V].x+P[V].y*P[V].y)<=d+LAND_RADIUS;
}

bool Jump(int V1,int V2)
{
    return sqrt((P[V1].x-P[V2].x)*(P[V1].x-P[V2].x)+
                (P[V1].y-P[V2].y)*(P[V1].y-P[V2].y))<=d;
}

```
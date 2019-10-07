---
title: List-leaves
date: 2019-07-06 19:05:14
tags: "数据结构"
categories: "编程练习题"
---

# **List Leaves** 

Given a tree, you are supposed to list all the leaves in the order of top down, and left to right.

### Input Specification:

Each input file contains one test case. For each case, the first line gives a positive integer *N* (≤10) which is the total number of nodes in the tree -- and hence the nodes are numbered from 0 to *N*−1. Then *N* lines follow, each corresponds to a node, and gives the indices of the left and right children of the node. If the child does not exist, a "-" will be put at the position. Any pair of children are separated by a space.

### Output Specification:

For each test case, print in one line all the leaves' indices in the order of top down, and left to right. There must be exactly one space between any adjacent numbers, and no extra space at the end of the line.

### Sample Input:

```in
8
1 -
- -
0 -
2 7
- -
- -
5 -
4 6
```

### Sample Output:

```out
4 1 5
```

## 求解思路

- 二叉树表示

- 二叉树构建

- 叶子结点从左到右输出

  主要是运用了层序遍历的思想,并运用队列的数据结构输出保证从左至右输出

## 答案

```c
#include <stdio.h>
#define MaxTree 10
#define Null -1
struct node{
    int data;
    int left;
    int right;
}T[MaxTree],Q[MaxTree];

int first=-1;
int last=-1;

void Push(struct node Node)
{
    Q[++last]= Node;
}

struct node Pop()
{
    return Q[++first];
}


int BuildTree(struct node T[],int * n)
{
    int root=-1;
    int N;
    int j;
    char cl,cr;
    scanf("%d",&N);
    *n=N;
    if(N)
    {
        int * check=(int *)malloc(N*sizeof(int));
        for(int i=0;i<N;i++) check[i]=0;
        for(int i=0;i<N;i++)
        {
            T[i].data=i;
            scanf(" %c %c",&cl,&cr);
            if(cl!='-')
            {
                T[i].left=cl-'0';
                check[T[i].left]=1;
            }else{
                T[i].left=Null;
            }
            if(cr!='-')
            {
                T[i].right=cr-'0';
                check[T[i].right]=1;
            }else{
                T[i].right=Null;
            }
        }

        for(j=0;j<N;j++)
            if(!check[j]) break;
        root=j;
    }
    return root;
}

void printNode(int root,int n)
{
   int leaves[MaxTree];
   int k=0;
   Push(T[root]);
   for(int i=0;i<n;i++)
   {
       struct node temp=Pop();
       if(temp.left == Null && temp.right == Null)
        leaves[k++]=temp.data;
        if(temp.left!=Null)
        Push(T[temp.left]);
        if(temp.right!=Null)
        Push(T[temp.right]);
   }
   for(int i=0;i<k-1;i++)
    printf("%d ",leaves[i]);
   printf("%d\n",leaves[k-1]);
}

int main()
{
    int root;
    int n;
    root=BuildTree(T,&n);
    printNode(root,n);
    return 0;
}

```
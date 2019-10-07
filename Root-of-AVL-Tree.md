---
title: Root of AVL Tree
date: 2019-07-26 00:17:40
tags: "数据结构"
categories: "编程练习题"
---

# Root of AVL Tree

An AVL tree is a self-balancing binary search tree. In an AVL tree, the heights of the two child subtrees of any node differ by at most one; if at any time they differ by more than one, rebalancing is done to restore this property. Figures 1-4 illustrate the rotation rules.



![img](https://images.ptausercontent.com/31) 



![img](https://images.ptausercontent.com/33) ![img](https://images.ptausercontent.com/34)

Now given a sequence of insertions, you are supposed to tell the root of the resulting AVL tree.



### Input Specification:

Each input file contains one test case. For each case, the first line contains a positive integer *N* (≤20) which is the total number of keys to be inserted. Then *N*distinct integer keys are given in the next line. All the numbers in a line are separated by a space.

### Output Specification:

For each test case, print the root of the resulting AVL tree in one line.

### Sample Input 1:

```in
5
88 70 61 96 120
```

### Sample Output 1:

```out
70
```

### Sample Input 2:

```
7
88 70 61 96 120 90 65
```

### Sample Output 2:

```
88
```

## 题目分析

AVL树即平衡二叉搜索树, **在AVL树中，任何节点的两个子子树的高度最多相差一个**

只有一个节点的树显然是AVL树，之后我们向其插入节点。然而在插入过程中可能破坏AVL树的特性，因此我们需要对树进行简单的修正，即AVL树的旋转。

设a节点在插入下一个节点后会失去平衡，这种插入可能出现四种情况：

1. 对a的左儿子的左子树进行一次插入。（左-左）    LL　
2. 对a的左儿子的右子树进行一次插入。（左-右）    LR
3.  对a的右儿子的左子树进行一次插入。（右-左）   RL
4.  对a的右儿子的右子树进行一次插入。（右-右）   RR

在此问题上,我觉得[llhthinker](https://home.cnblogs.com/u/llhthinker/)博主说的简单易懂

情形1和4，情形2和3分别是关于A节点的镜像对称，故在理论上是两种情况，而编程具体实现还是需要考虑四种。

- 单旋转--情形1和4：

  ![img](https://images2015.cnblogs.com/blog/788978/201508/788978-20150829231236140-178469526.jpg)

- 双旋转--情形2和3：

  　　情形2和3就是向上图中的子树Y插入一个节点，由上图可知，无论是左单旋还是右单旋都无法改变子树Y的高度。解决办法是再将子树Y分解成根节点和相应的左子树和右子树，然后对相应的节点做相应的旋转，如下图：

  ![img](https://images2015.cnblogs.com/blog/788978/201508/788978-20150829231713594-2020040829.jpg)

## 答案

```c++
#include <cstdio>
#include <vector>
#include <stack>
#include <cstring>
#include <cstdlib>
typedef struct TreeNode* AvlTree;
typedef struct TreeNode* Position;

struct TreeNode{
    int Data;
    AvlTree Left;
    AvlTree Right;
    int Height;
};

AvlTree Insert(int x,AvlTree T); //每次插入新结点需要调整高度
Position LL(Position a);
Position RR(Position b);
Position LR(Position c);
Position RL(Position d);
int Max(int x,int y); //返回较大的那个
int Height(Position P); //返回该结点的高度

int main()
{
    int value,n;
    AvlTree T=NULL;
    scanf("%d",&n);
    for(int i=0;i<n;i++)
    {
        scanf("%d",&value);
        T=Insert(value,T);  //得到根结点的值
    }
    printf("%d\n",T->Data);
    return 0;
}

AvlTree Insert(int x,AvlTree T)
{
    if(T==NULL)
    {
        T = (AvlTree)malloc(sizeof(struct TreeNode));
        T->Data=x;
        T->Left=T->Right=NULL;
        T->Height=0;
    }
    else if(x<T->Data)  //若比根结点小则插入左子树
    {
        T->Left=Insert(x,T->Left);
        //插入后要调整高度
        if(Height(T->Left)-Height(T->Right)==2)
        {
            if(x<T->Left->Data) T=LL(T);
            else  T=LR(T);
        }
    }
    else if(x>T->Data) //若比根结点大则插入右子树
    {
        T->Right=Insert(x,T->Right);
        //调整高度
        if(Height(T->Right)-Height(T->Left)==2)
        {
            if(x>T->Right->Data) T=RR(T);
            else T=RL(T);
        }
    }

    /*更新结点高度*/
    T->Height=Max(Height(T->Left),Height(T->Right))+1;
    return T;
}

Position LL(Position a)
{
    Position b=a->Left;
    a->Left = b->Right;
    b->Right= a;
    /*更新a,b结点高度*/
    a->Height = Max(Height(a->Left), Height(a->Right)) + 1;
    b->Height = Max(Height(b->Left), Height(b->Right)) + 1;
    return b; //得到新的根结点
}

Position RR(Position b)
{
    Position a=b->Right;
    b->Right = a->Left;
    a->Left = b;
    /*更新a,b结点高度*/
    a->Height = Max(Height(a->Left), Height(a->Right)) + 1;
    b->Height = Max(Height(b->Left), Height(b->Right)) + 1;
    return a; //得到新的根结点
}

Position LR(Position a)
{
    a->Left = RR(a->Left);
    return LL(a);
}

Position RL(Position b)
{
    b->Right = LL(b->Right);
    return RR(b);
}

int Max(int x,int y)
{
    return x>y ? x :y;
}

int Height(Position P)
{
    if(P==NULL)
    {
        return -1; //空结点高度为-1
    }
    return P->Height;
}

```
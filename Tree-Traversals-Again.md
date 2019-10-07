---
title: Tree Traversals Again
date: 2019-07-26 00:18:45
tags: "数据结构"
categories: "编程练习题"
---

# Tree Traversals Again

An inorder binary tree traversal can be implemented in a non-recursive way with a stack. For example, suppose that when a 6-node binary tree (with the keys numbered from 1 to 6) is traversed, the stack operations are: push(1); push(2); push(3); pop(); pop(); push(4); pop(); pop(); push(5); push(6); pop(); pop(). Then a unique binary tree (shown in Figure 1) can be generated from this sequence of operations. Your task is to give the postorder traversal sequence of this tree.

![img](https://images.ptausercontent.com/30)
Figure 1

### Input Specification:

Each input file contains one test case. For each case, the first line contains a positive integer *N* (≤30) which is the total number of nodes in a tree (and hence the nodes are numbered from 1 to *N*). Then 2*N* lines follow, each describes a stack operation in the format: "Push X" where X is the index of the node being pushed onto the stack; or "Pop" meaning to pop one node from the stack.

### Output Specification:

For each test case, print the postorder traversal sequence of the corresponding tree in one line. A solution is guaranteed to exist. All the numbers must be separated by exactly one space, and there must be no extra space at the end of the line.

### Sample Input:

```in
6
Push 1
Push 2
Push 3
Pop
Pop
Push 4
Pop
Pop
Push 5
Push 6
Pop
Pop
```

### Sample Output:

```out
3 4 2 6 5 1
```

## 题目解析

- 题目大意:

  **用栈的形式操作给出一棵二叉树的建立的顺序，求这棵二叉树的后序遍历**

- 从题目中首先可以看出该栈的出栈顺序是中序遍历,其实仔细观察你会发现其入栈顺序是先序遍历,从而可以得到该题目就是运用先序遍历和中序遍历来推出后序遍历

  前序遍历首先访问的是根节点，中序遍历首先访问的是左子树，而后序遍历最后才访问根节点。因此，前序遍历中第一个元素为后序遍历的最后一个元素，在中序遍历中根节点左边的为左子树的节点，右边为右子树的节点。所以需要递归的求解这个问题。

  root为当前子树的根结点在前序pre中的下标，start和end为当前子树的最左边和最右边的结点在中序in中的下标。用i找到当前子树的根结点root在中序中的下标，然后左边和右边就分别为当前根结点root的左子树和右子树。最后通过递归实现

  ![](https://raw.githubusercontent.com/catwithtudou/photo/master/20190726111043.png)

  ![](https://raw.githubusercontent.com/catwithtudou/photo/master/20190726111107.png)

## 答案

```c++
#include <cstdio>
#include <vector>
#include <stack>
#include <cstring>

using namespace std;

vector<int> pre,in,post,value;

void postorder(int root,int start,int end)
{
    if(start>end) return;
    int i=start;
    while(i<end&&in[i]!=pre[root]) i++;
    postorder(root+1,start,i-1);
    postorder(root+1+i-start,i+1,end);
    post.push_back(pre[root]);
}

int main()
{
    int N;
    stack<int> s;
    char str[5];
    int key=0;
    scanf("%d",&N);
    while(~scanf("%s",str)){
        if(strlen(str)==4){
            int num;
            scanf("%d",&num);
            value.push_back(num);
            pre.push_back(key);
            s.push(key++);
        }else{
            in.push_back(s.top());
            s.pop();
        }
    }
    postorder(0,0,N-1);
    printf("%d",value[post[0]]);
    for(int i=1;i<N;i++)
    {
        printf(" %d",value[post[i]]);
    }
    return 0;
}

```



> 参考
>
> https://www.liuchuo.net/archives/2168
>
> https://blog.csdn.net/shinanhualiu/article/details/49279051
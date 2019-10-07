---
title: Insert-or-Merge
date: 2019-09-09 08:48:35
tags: "数据结构"
categories: "编程练习题"
---

# Insert or Merge

According to Wikipedia:

**Insertion sort** iterates, consuming one input element each repetition, and growing a sorted output list. Each iteration, insertion sort removes one element from the input data, finds the location it belongs within the sorted list, and inserts it there. It repeats until no input elements remain.

**Merge sort** works as follows: Divide the unsorted list into N sublists, each containing 1 element (a list of 1 element is considered sorted). Then repeatedly merge two adjacent sublists to produce new sorted sublists until there is only 1 sublist remaining.

Now given the initial sequence of integers, together with a sequence which is a result of several iterations of some sorting method, can you tell which sorting method we are using?

### Input Specification:

Each input file contains one test case. For each case, the first line gives a positive integer *N* (≤100). Then in the next line, *N* integers are given as the initial sequence. The last line contains the partially sorted sequence of the *N* numbers. It is assumed that the target sequence is always ascending. All the numbers in a line are separated by a space.

### Output Specification:

For each test case, print in the first line either "Insertion Sort" or "Merge Sort" to indicate the method used to obtain the partial result. Then run this method for one more iteration and output in the second line the resuling sequence. It is guaranteed that the answer is unique for each test case. All the numbers in a line must be separated by a space, and there must be no extra space at the end of the line.

### Sample Input 1:

```in
10
3 1 2 8 7 5 9 4 6 0
1 2 3 7 8 5 9 4 6 0
```

### Sample Output 1:

```out
Insertion Sort
1 2 3 5 7 8 9 4 6 0
```

### Sample Input 2:

```in
10
3 1 2 8 7 5 9 4 0 6
1 3 2 8 5 7 4 9 0 6
```

### Sample Output 2:

```out
Merge Sort
1 2 3 8 4 5 7 9 0 6
```

## 题目分析

这一道题主要是考察的是插入排序和归并排序的迭代过程,其中也有两个思路

第一个思路是将排序函数每做一次迭代的时候都要判断一下这一次迭代的结果是否与输入的迭代结果相同,若相同则再迭代一次并退出排序过程

第二个思路是通过Sort函数来模拟排序过程,首先判断插入排序比较简单,也就是在插入排序中是从左到右的顺序来进行排序的,我们可以通过比较排序过程中未排序的部分是否相等,若相等则是插入排序,且下一次的序列是`sort(A,A+i+2)`,若不相等则是归并排序,而归并排序就不用考录中间序列了,直接对原来的序列进行模拟归并过程,即i从0到n/k,每一次迭代过程是`sort(A+i*k,A+(i+1)*k)`,而最后的时候合并剩余部分即`sort(A+n/k*k,A+n)`,这样就是一次归并的过程.而在迭代的过程中判断是否与输入的迭代序列相同,若相同则再归并一次,然后退出循环

## 代码实现

```c
#include <stdio.h>
#include <stdlib.h>
#define MAX 100005
typedef int ElementType;
int N;
ElementType A[MAX];
ElementType AA[MAX];
ElementType Copy[MAX];
int IsInsertion=0; //若不是插入排序则判断是否为归并排序

//判断序列是否相同
int IsIdentical(int a[])
{
   for(int i=0;i<N;i++)
        if(a[i]!=Copy[i])
         return 0;
   return 1;
}

//插入排序
void InsertSort()
{
    ElementType temp;
    int j;
    int flag=0;
    for(int i=1;i<N;i++)
    {
        temp=A[i];
        for(j=i;j>0&&temp<A[j-1];j--)
            A[j]=A[j-1];
        A[j]=temp;
        if(IsIdentical(A))
        {
            printf("Insertion Sort\n");
            IsInsertion=1;
            flag=1;
            continue;
        }
        if(flag)break; //再做一次迭代就退出
    }
}





//归并排序
void  Mergel(ElementType tempA[],int L,int R,int RightEnd)
{
	//将有序的A[L]~A[R-1]和A[R]~A[RightEnd]归并成一个有序序列
	int LeftEnd, NumElements, temp;
	int i;
	LeftEnd = R - 1;	//左边终点位置
	temp = L;	//有序序列的起始位置
	NumElements = RightEnd - L + 1;		//元素总个数
	while( L <= LeftEnd && R <= RightEnd ) {
		if( AA[L] <= AA[R] )
			tempA[temp++] = AA[L++];	//将左边元素复制到TmpA
		else
			tempA[temp++] = AA[R++];	//将右边元素复制到TmpA
	}
	while( L <= LeftEnd )
		tempA[temp++] = AA[L++];	//直接复制左边剩下的
	while( R <= RightEnd )
		tempA[temp++] = AA[R++];	//直接复制右边剩下的
    for(i=0;i<NumElements;i++,RightEnd--)
        AA[RightEnd]=tempA[RightEnd];
}
void MsortByPass(ElementType tempA[],int N,int length)
{
    //两两归并有序子列
    int i,j;
    for(i=0;i<=N-2*length;i+=2*length)
        Mergel(tempA,i,i+length,i+2*length-1);
    if(i+length<N) //归并最后2个子列
        Mergel(tempA,i,i+length,N-1);
    else //最后只剩1个子列
        for(j=i;j<N;j++)
        tempA[j]=AA[j];

}
void MergeSortByLoop()
{
    int length=1;
    int flag=0;
    ElementType *tempA;
    tempA=(ElementType *)malloc(N*sizeof(ElementType));
    if(tempA!=NULL)
    {
        while(length<N){
            MsortByPass(tempA,N,length);
            if(IsIdentical(AA))
            {
                printf("Merge Sort\n");
                flag=1;
                length*=2;
                continue;
            }
            length*=2;
            if(flag) break;
        }
        free(tempA);
    }else{
        printf("空间已满\n");
    }
}

int main()
{
    scanf("%d",&N);
    for(int i=0;i<N;i++){
        scanf("%d",&A[i]);
        AA[i]=A[i];
    }
    for(int i=0;i<N;i++)
        scanf("%d",&Copy[i]);
    InsertSort();
    if(IsInsertion){
        printf("%d",A[0]);
        for(int i=1;i<N;i++)
            printf(" %d",A[i]);
    }else{
        MergeSortByLoop();
        printf("%d",AA[0]);
        for(int i=1;i<N;i++)
            printf(" %d",AA[i]);
    }

    return 0;
}

```

```c++
#include <iostream>
#include <cstdio>
#include <algorithm>
#define MAX 100
using namespace std;
int A[MAX],B[MAX];
int N;
int main()
{
    int i,j;
    scanf("%d",&N);
    for(i=0;i<N;i++)
        cin>>A[i];
    for(i=0;i<N;i++)
        cin>>B[i];
    for(i=0;i<N-1&&B[i+1]>=B[i];i++);
    for(j=i+1;j<N&&A[j]==B[j];j++);
    if(j==N){
        cout<<"Insert Sort"<<endl;
        sort(A,A+i+2);
    }else{
        cout<<"Merge Sort"<<endl;
        int k=1;
        int flag=1;
        while(flag){
            flag=0;
            for(i=0;i<N;i++){
                if(A[i]!=B[i])
                    flag=1;
            }
            k*=2;
            for(i=0;i<N/k;i++)
                sort(A+i*k,A+(i+1)*k);
            sort(A+N/k*k,A+N);
        }
    }
    j=0;
    printf("%d",A[j]);
    for(j=1;j<N;j++){
        printf(" %d",A[j]);
    }
}

```


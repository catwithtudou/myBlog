---
title: Hashing - Hard Version
date: 2019-09-09 08:40:59
tags: "数据结构"
categories: "编程练习题"
---

# Hashing - Hard Version

Given a hash table of size *N*, we can define a hash function *H*(*x*)=*x*%*N*. Suppose that the linear probing is used to solve collisions, we can easily obtain the status of the hash table with a given sequence of input numbers.

However, now you are asked to solve the reversed problem: reconstruct the input sequence from the given status of the hash table. Whenever there are multiple choices, the smallest number is always taken.

### Input Specification:

Each input file contains one test case. For each test case, the first line contains a positive integer *N* (≤1000), which is the size of the hash table. The next line contains *N* integers, separated by a space. A negative integer represents an empty cell in the hash table. It is guaranteed that all the non-negative integers are distinct in the table.

### Output Specification:

For each test case, print a line that contains the input sequence, with the numbers separated by a space. Notice that there must be no extra space at the end of each line.

### Sample Input:

```in
11
33 1 13 12 34 38 27 22 32 -1 21
```

### Sample Output:

```out
1 13 12 21 33 34 38 27 22 32
```

## 题目分析

这道题大意很容易明白，也就是用已经存储好的Hash表来反推出其Hash表的插入顺序

由于是线性探测所以说每次有冲突情况都是向后移一位即`j=(j+1)%N`,而由于是反推插入元素的冲突,所以说在线性探测目标位置之前都已经有元素插入

而如何存储插入的顺序呢

这里我们可以利用图的邻接矩阵来存储每一次的顺序,即入度和出度,而最后求出其插入顺序其实也就是拓扑排序,所以这道题最巧妙的一点就是在于其反推出的顺序即可以用拓扑排序还原其插入的顺序

## 代码实现

```c++
#include <cstdio>
#include <cstdlib>
#include <set>
using namespace std;
#define MAX 1000
#define INFINITY 0
int A[MAX][MAX];
 
void BuildGraph ( int Hash[], int N ) {
	int i, j, tmp;
	for ( i = 0; i < N; i++ ) {
		if ( Hash[i] >= 0 ) {
			tmp = Hash[i] % N; //求余数
			if ( Hash[tmp] != Hash[i]) { //如果这个位置已被占有
				for ( j = tmp; j != i; j = ( j + 1 ) % N )
					A[j][i] = 1; //位置j到i的元素都在i之前进入哈希表
			}
		}
	}
}
 
int TopSort( int Hash[], int HashMap[], int N , int num){
	int V, W, cnt = 0, Indegree[MAX] = {0};
	set<int> s;
	//计算各结点的入度
	for( V = 0; V < N; V++ )
		for( W = 0; W < N; W++ )
			if( A[V][W] != INFINITY )
				Indegree[W]++;	//对于有向边<V,W>累计终点W的入度
	//入度为0的入队
	for( int i = 0; i < N; i++ )
		if( Indegree[i] == 0 && Hash[i] > 0 )
			s.insert( Hash[i] );  //将大于0且入度为0的顶点放入集合，自动排序
	while( !s.empty() ) {
		V = HashMap[ *s.begin() ]; //获取最小的元素的下标
		s.erase( s.begin() );
		cnt++;
		printf("%d", Hash[V]);
		if ( cnt != num )
			printf(" ");
		for( W = 0; W < N; W++ )
			if( A[V][W] != INFINITY ) {  //<V, W>有有向边
				if( --Indegree[W] == 0 )  //去掉V后，如果W的入度为0
					s.insert( Hash[W] );
			}
	}
	if( cnt != num ) return 0;	//如果没有取出所有元素，说明图中有回路
	else return 1;
}
 
int main () {
	int N, Hash[1005],HashMap[100000], num = 0;
	scanf("%d", &N);
	for ( int i = 0 ;i < N; i++ ) {
		scanf("%d", &Hash[i]);
		HashMap[ Hash[i] ] = i;	//记录下标
		if ( Hash[i] > 0 ) num++; //记录哈希表中的元素个数
	}
	BuildGraph( Hash, N );
	TopSort( Hash, HashMap, N ,num );
	system("pause");
	return 0;
}
```


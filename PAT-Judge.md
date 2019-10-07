---
title: PAT-Judge
date: 2019-09-09 08:49:13
tags: "数据结构"
categories: "编程练习题"
---

# PAT Judge

The ranklist of PAT is generated from the status list, which shows the scores of the submissions. This time you are supposed to generate the ranklist for PAT.

### Input Specification:

Each input file contains one test case. For each case, the first line contains 3 positive integers, *N* (≤104), the total number of users, *K* (≤5), the total number of problems, and *M* (≤105), the total number of submissions. It is then assumed that the user id's are 5-digit numbers from 00001 to *N*, and the problem id's are from 1 to *K*. The next line contains *K* positive integers `p[i]` (`i`=1, ..., *K*), where `p[i]` corresponds to the full mark of the i-th problem. Then *M* lines follow, each gives the information of a submission in the following format:

```
user_id problem_id partial_score_obtained
```

where `partial_score_obtained` is either −1 if the submission cannot even pass the compiler, or is an integer in the range [0, `p[problem_id]`]. All the numbers in a line are separated by a space.

### Output Specification:

For each test case, you are supposed to output the ranklist in the following format:

```
rank user_id total_score s[1] ... s[K]
```

where `rank` is calculated according to the `total_score`, and all the users with the same `total_score` obtain the same `rank`; and `s[i]` is the partial score obtained for the `i`-th problem. If a user has never submitted a solution for a problem, then "-" must be printed at the corresponding position. If a user has submitted several solutions to solve one problem, then the highest score will be counted.

The ranklist must be printed in non-decreasing order of the ranks. For those who have the same rank, users must be sorted in nonincreasing order according to the number of perfectly solved problems. And if there is still a tie, then they must be printed in increasing order of their id's. For those who has never submitted any solution that can pass the compiler, or has never submitted any solution, they must NOT be shown on the ranklist. It is guaranteed that at least one user can be shown on the ranklist.

### Sample Input:

```in
7 4 20
20 25 25 30
00002 2 12
00007 4 17
00005 1 19
00007 2 25
00005 1 20
00002 2 2
00005 1 15
00001 1 18
00004 3 25
00002 2 25
00005 3 22
00006 4 -1
00001 2 18
00002 1 20
00004 1 15
00002 4 18
00001 3 4
00001 4 2
00005 2 -1
00004 2 0
```

### Sample Output:

```out
1 00002 63 20 25 - 18
2 00005 42 20 0 22 -
2 00007 42 - 25 - 17
2 00001 42 18 18 4 2
5 00004 40 15 0 25 -
```

## 题目分析

这道题的大意也就是我们平常PAT排名标准,根据每个人提交的题目通过题数和获得的分数来进行排名

根据题目所说的标准首先是按总分至高到低排名,若总分相同则名次相同,且总分相同的时候根据通过的满分题数排名,若满分题数相同时则按照id从低至高排序

最后我们需要注意的是没有提交记录或者题目都未通过编译者,不计入排序

理解了题目大意之后,我们来思考一下这道题如何做,我们知道分数的排序可以直接通过sort来进行排序,而最重要的用户相关信息的存储,这里我们可以通过散列值存储来保存用户的id,每道题的分数情况

其中我们需要注意的是若该题目的下一次分数比以前的分数更低,则不需要更新值

## 代码实现

```c++
#include <iostream>
#include <cstdio>
#include <algorithm>
#include <vector>
using namespace std;

struct user{
    int id=0;
    int total=0;
    int rank=0;
    vector<int> score;
    int passNum=0;
    bool isPass=false;
};

bool cmp(user a,user b)
{
    if(a.total!=b.total)
        return a.total>b.total;
    else if(a.passNum!=b.passNum)
        return a.passNum>b.passNum;
    else
        return a.id<b.id;
}


int main()
{
   int N,K,M;
   scanf("%d %d %d",&N,&K,&M);
   vector<user>v(N+1); //结构体动态数组
   for(int i=1;i<=N;i++)
    //将用户的题目分数初始化为-1,因为得到的分数为0,但未提交过的分数为'-',所以每次未通过的则置为-2
     v[i].score.resize(K+1,-1);
   vector<int>full(K+1); //记录题目满分分数
   for(int i=1;i<=K;i++)
    scanf("%d",&full[i]);
   for(int i=0;i<M;i++)
   {
       int score,id,num;
       scanf("%d %d %d",&id,&num,&score);
       v[id].id=id;
       v[id].score[num]=max(v[id].score[num],score); //更新最大值
       if(score!=-1)
        v[id].isPass=true; //说明该用户已有提交题目通过
       else if(v[id].score[num]==-1)
        v[id].score[num]=-2;
   }

   for(int i=1;i<=N;i++)
    for(int j=1;j<=K;j++)
   {
       if(v[i].score[j]!=-1&&v[i].score[j]!=-2)
        v[i].total+=v[i].score[j];
       if(v[i].score[j]==full[j])
        v[i].passNum++;
   }

   sort(v.begin()+1,v.end(),cmp); //根据自定义排序标准将用户进行排序

   for(int i=1;i<=N;i++)
   {
       v[i].rank=i;
       if(i!=1&&v[i].total==v[i-1].total)
        v[i].rank=v[i-1].rank;
   }

   for(int i=1;i<=N;i++)
   {
       if(v[i].isPass==true) //有提交通过记录的才能进行输出
       {
           printf("%d %05d %d",v[i].rank,v[i].id,v[i].total);
           for(int j=1;j<=K;j++)
           {
               if(v[i].score[j]!=-1&&v[i].score[j]!=-2)
               printf(" %d",v[i].score[j]);
               else if(v[i].score[j]==-1) //未提交过的题目为'-'
                printf(" -");
               else  //已提交但是分数为0分的
                printf(" 0");

           }
           printf("\n");
       }
   }
   return 0;
}

```


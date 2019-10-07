---
title: KMP 
date: 2019-07-05 21:15:29
tags: [算法,数据结构]
categories: "算法"
---

### KMP

```c
    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    int  ViolentMatch(char * s,char * p)
    {
        int sLength=strlen(s);
        int pLength=strlen(p);

        int i,j;
        i=0;j=0;

        while(i<sLength && j<pLength)
        {
            if(*(s+i)==*(p+j))
            {
                i++;
                j++;
            }
            else
            {
                i=i-j+1;
                j=0;
            }
        }
        if(j==pLength)
        {
            return i-j;
        }
        return 0;
    }
    int KMP(char *s,char *p)
    {
        int i=0;
        int j=0;
        int pLength=strlen(p);
        int sLength=strlen(s);
        int * next;
        next=(int *)malloc(pLength * sizeof(int));
        GetNext(p,next);
        while(i<sLength && j<pLength)
        {
            if(j==-1||*(s+i)==*(p+j))
            {
                i++;
                j++;
            }
            else
            {
                j=next[j];
            }
        }
        if(j==pLength)
        {
            return i-j;
        }
        return 0;
    }
    void GetNext(char * p,int next[])
    {
        int j=0;
        int length=strlen(p);
        next[0]=-1;
        int k=-1;
        while(j<length-1)
        {
            if(k==-1||*(p+j)==*(p+k))
            {
                j++;
                k++;
                if(*(p+j)!=*(p+k))
                {
                    next[j]=k;
                }
                else
                {
                    next[j]=next[k];
                }
            }
            else
            {
                k=next[k];
            }
        }
    }
    int main()
    {
        printf("Hello world!\n");
        char s[10]={"hello"};
        char p[10]={"lloo"};
        if(ViolentMatch(s,p))
        {
            printf("Find it!\n");
        }
        if(KMP(s,p))
        {
            printf("Find it!\n");
        }
        return 0;
    }
```
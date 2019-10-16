---
title:  Leetcode-19-删除链表的倒数第 n 个节点
date: 2019-10-13 23:33:45
tags: ["编程练习题","链表"]
categories: "编程练习题"
---

# Leetcode-19-删除链表的倒数第 *n* 个节点

## question

给定一个链表，删除链表的倒数第 *n* 个节点，并且返回链表的头结点。

## analyse

这道题考察的是关于链表的题,题目非常的简单直接

我们知道这道题考察的地方在于如何找到要删除的那个结点,也就是倒数第n个结点

链表的长度不像数组可以直接得到,而链表的长度需要遍历才能知道,因为它每一个结点都存放在不同的内存地址

首先我们最容易想到的一种做法就是

- 两次遍历链表

  第一次遍历是为了求链表长度

  第二次遍历是为了找到length-n-1索引即要删除的那个结点位置

但是这道题还有一种更简便的做法,那就是通过双指针来进行遍历,而且只需要遍历一次

大致思路就是

定义两个指针,先让一个指针走n步

然后判断第一个走的指针是否为空,若不为空则两个指针同时指向走x步

而length-n=x

就这样找到要删除的结点

## answer

第一种做法

```c++
public ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode flag = new ListNode(0); //设置哨兵来处理临界情况
    flag.next = head;
    int length  = 0;
    ListNode first = head;
    while (first != null) {
        length++;
        first = first.next;
    }
    length -= n;
    first = flag;
    while (length > 0) {
        length--;
        first = first.next;
    }
    first.next = first.next.next;
    return flag.next;
}
```

第二种做法

```c++
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
     ListNode* flag = new ListNode(0);
        flag->next = head;

        ListNode* p = flag;
        ListNode* q = flag;
        for( int i = 0 ; i < n + 1 ; i ++ ){
            q = q->next;
        }

        while(q){
            p = p->next;
            q = q->next;
        }

        ListNode* delNode = p->next;
        p->next = delNode->next;
        delete delNode;

        ListNode* retNode = flag->next;
        delete flag;

        return retNode;
        
    }
};

 */
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        if(head==NULL)
            return NULL;
        ListNode *prePtr = NULL, *curPtr = head, *nextPtr=head->next;
        while(nextPtr != NULL){
            curPtr->next = prePtr;
            prePtr = curPtr;
            
            
            curPtr = nextPtr;
            nextPtr = nextPtr->next;
        }

        curPtr->next=prePtr;
       
        return curPtr;  
    }
};


```


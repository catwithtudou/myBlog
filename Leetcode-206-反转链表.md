---
title: Leetcode-206-反转链表
date: 2019-10-16 19:10:11
tags: ["编程练习题"]
categories: "编程练习题"
---

# Leetcode-206-反转链表

## 题目

反转一个单链表。

**示例:**

```
输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL
```

## 题目分析

其实这道题在以前我就做过

所以我在这里就不再说啦,详情请点开[Rerversing-Linked-List](./Rerversing-Linked-List.md)

## 代码

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        if(head==NULL)
            return NULL;
        ListNode* flag = NULL;
        ListNode* New = head;
        ListNode* Old = New->next;
        while(Old)
        {
            New->next=flag;
            flag = New;
            
            New = Old;
            Old = Old->next;
        }
        New->next = flag;
        return New;
    }
};
```

![](https://raw.githubusercontent.com/catwithtudou/photo/master/20191016205521.png)
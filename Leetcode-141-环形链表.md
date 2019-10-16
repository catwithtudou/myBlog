---
title: Leetcode-141-环形链表
date: 2019-10-16 19:10:11
tags: ["编程练习题"]
categories: "编程练习题"
---

# Leetcode-141-环形链表

## 题目描述

给定一个链表，判断链表中是否有环。

为了表示给定链表中的环，我们使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。 如果 pos 是 -1，则在该链表中没有环。

**示例 1：**

```
输入：head = [3,2,0,-4], pos = 1
输出：true
解释：链表中有一个环，其尾部连接到第二个节点。
```

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2018/12/07/circularlinkedlist.png)

## 题目分析

这道题也是非常的简单,属于链表中的一个基础题

但是通过力扣里面的大神写的代码,还是学到了很多东西

这道题我目前想到的就只有三种做法

第一种就是大家熟悉的暴力求解

即采用两遍遍历,第一个遍历指定一个结点,第二遍历则从指定的结点开始往下遍历若有与指定结点相等则说明是有环的

当然这种方法的时间复杂度是熟悉的O(N^2)

第二种方法呢应该很多都想的到,那就是采取索引的思想,即遍历的结点为它加上索引来判断其是否遍历过,其实现索引呢可以采用哈希的思想就比较容易,当然每个语言里面都有类似的容器,往容器里加入则只需判断是否有重复元素即可

时间和空间复杂度都是O(N)

我这里用的CPP那就用熟悉的set容器就行了

第三种方法也非常值得学习

快慢指针(双指针有时候真的有奇效)

快指针会每次走两步,慢指针会每次走一步

如果链表中有环的话,则快指针始终会追到慢指针.若无环则直接走到尾节点为空

> [来自力扣一位老哥讲解](https://leetcode-cn.com/problems/linked-list-cycle-ii/solution/shuang-zhi-zhen-qing-xi-ti-jie-zhen-zheng-cong-shu/)
>
> 假设非环部分的长度是x，从环起点到相遇点的长度是y。环的长度是c。
> 现在走的慢的那个指针走过的长度肯定是x+n1*c+y，走的快的那个指针的速度是走的慢的那个指针速度的两倍。这意味着走的快的那个指针走的长度是2(x+n1*c+y)。
>
> 还有一个约束就是走的快的那个指针比走的慢的那个指针多走的路程一定是环长度的整数倍。根据上面那个式子可以知道2(x+n1*c+y)-x+n1*c+y=x+n1*c+y=n2*c。
>
> 所以有x+y=(n2-n1)*c,这意味着什么？我们解读下这个数学公式：非环部分的长度+环起点到相遇点之间的长度就是环的整数倍。这在数据结构上的意义是什么？现在我们知道两个指针都在离环起点距离是y的那个相遇点，而现在x+y是环长度的整数倍，这意味着他们从相遇点再走x距离就刚刚走了很多圈，这意味着他们如果从相遇点再走x就到了起点。
> 那怎么才能再走x步呢？答：让一个指针从头部开始走，另一个指针从相遇点走，等这两个指针相遇那就走了x步此时就是环的起点。

时间复杂度是O(N),空间复杂度则是O(1)

在刷这道题的时候,我琢磨着应该就这三种方法,为什么我还是只击败了40%几

看了其他人的代码发现确实是同样的方法,但是别人的代码实现着实优雅高效许多值得学习

## 代码实现

### 第二种方法

```c++
class Solution {
public:
    bool hasCycle(ListNode *head) {
        //第一种方法
        //通过向容器里面添加结点,若有环时则Set集合里面便有重复元素,而Set集合不允许重复元素的插入
        ListNode* node = head;
        set<ListNode*> res;
        int i=0;
        while(node){
            res.insert(node);
            if(i==res.size()){
                return true;
            }
            node=node->next;
            i++;
        }
        return false;
    }
};
```

### 第三种方法

```c++
class Solution {
public: 
    bool hasCycle(ListNode *head) {
        if(head == nullptr || head->next == nullptr)
            return false;  
        ListNode* slow = head->next;
        ListNode* fast = slow->next;
        while(slow != fast){
            if(slow == nullptr || fast == nullptr)
                return false;
            slow = slow->next;
            fast = fast->next;
            if(fast != nullptr)
                fast = fast->next;
        }
        return true;
    }
};
```

```c++
class Solution {
public:
    bool hasCycle(ListNode *head) {
        ListNode* a = head,*b = head;
        while(a != NULL && a->next != NULL){
            a = a->next->next;
            b = b->next;
            if(a == b) return true;
        }
        return false;
    
    
    }
};
```


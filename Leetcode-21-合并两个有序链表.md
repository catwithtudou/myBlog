---
title:  Leetcode-21-合并两个有序链表
date: 2019-10-14 01:09:25
tags: ["编程练习题","链表"]
categories: "编程练习题"
---

# Leetcode-21-合并两个有序链表

## question

将两个有序链表合并为一个新的有序链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 

## analyse

这道题题目也非常的简单,但是扩展开来确实花了很多时间

见识了很多种方法

首先第一种思想也是我们比较容易想到的,就是通过创建一个新的链表,将题目中的两个有序链表进行比较,若哪一方的某结点比较小那么就放入新链表,并让它指向它下一个结点,直至某一方为空后,剩下的就直接拼接到新的链表中

而这种想法呢实现起来既可以使用迭代,也可以使用递归

> 迭代与遍历的时间复杂度一致,只是空间复杂度很容易知道递归所需的空间比较大,而迭代则是O(1)

在这里要吐槽一下C/C++的指针操作,本来一道很容易实现的题结果Bug贼多,我太难了

搞的后面直接用JAVA实现的这一种思想

然后第二种思想是总的来说就是头结点的插入,怎么个插入法呢

简单点来说就是若某一方的某结点的值比较小,那么就将该结点插入另外一方的头结点中,然后再将拼接了较小的结点的那一方插入一个新的链表存储下来,是为了不影响下面两方的比较

第三种思想呢是类似于第一种思想,只不过它的实现方式类似于循环队列的链表实现方式,且其中在合并的时候并不是一个一个去合并,而是采用连续的块即直到比较出关于小于某结点的值之前的最大值,这样的操作可以减少读写

在以上三个思想实现的时候,大家需注意

这道题的临界条件即若l1,l2一来便是NULL的处理(我就在这里吃了很大的亏)

还有就是哨兵的作用,来处理指针的临界,非常的方便安全

## answer

- 第一种思想

  - 递归

    ```java
    class Solution {
        public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
            if (l1 == null) {
                return l2;
            }
            else if (l2 == null) {
                return l1;
            }
            else if (l1.val < l2.val) {
                l1.next = mergeTwoLists(l1.next, l2);
                return l1;
            }
            else {
                l2.next = mergeTwoLists(l1, l2.next);
                return l2;
            }
    
        }
    }
    ```

  - 迭代

    ```java
    class Solution {
        public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
            ListNode prehead = new ListNode(-1);//在返回头结点的前一个头结点即哨兵,我们只需要关注它的next
            ListNode prev = prehead;
            while (l1 != null && l2 != null) {
                if (l1.val <= l2.val) {
                    prev.next = l1;
                    l1 = l1.next;
                } else {
                    prev.next = l2;
                    l2 = l2.next;
                }
                prev = prev.next;
            }
            prev.next = l1 == null ? l2 : l1;
            return prehead.next;
        }
    }
    ```

    

- 第二种思想

  受教于[zip-2](https://leetcode-cn.com/problems/merge-two-sorted-lists/solution/jian-dan-cyu-yan-jie-fa-ji-bai-9776gong-chu-xue-zh/)

  ```c
  struct ListNode* mergeTwoLists(struct ListNode* l1, struct ListNode* l2){
      if(l1 == NULL) return l2;
      if(l2 == NULL) return l1;
      
      struct ListNode newhead, *p1, *p2;
      newhead.next = l1;
      l1 = &newhead;
      
      while(l2 != NULL) {
          p1 = l1->next;
          p2 = l2->next;
          if(p1 == NULL) {
              l1->next = l2;
              break;
          }
          if(l2->val <= p1->val) {
              l2->next = l1->next; //直接将较小的结点插入头结点中
              l1->next = l2; //将头结点更新
              l2 = p2;
          }
          l1 = l1->next;
      }
      return newhead.next;
  }
  ```

- 第三种思想

  ```java
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
      ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
          //类似于循环队列的链表实现形式进行头尾的拼接
          ListNode* resultListHead = NULL;
          ListNode* resultListEnd = NULL;
          
          if (l1==NULL)
              return l2;
          if (l2==NULL)
              return l1;
          
          if (l1->val < l2->val)
          {
              resultListHead = resultListEnd = l1;
              l1 = l1->next;
          }else
          {
              resultListHead = resultListEnd = l2;
              l2 = l2->next;
          }
          
          while(l1 != NULL && l2 != NULL){
               if (l1->val < l2->val)
               {
                  // 将l1这段拼接到结果串中
                   resultListEnd->next = l1;
                  // 如果l1中有一段比l2->val小那就找到l1这段中的最后一个节点，并让结果串指向这个节点
                  while (l1->next!=NULL && l1->next->val < l2->val)
                  {
                      l1 = l1->next;
                  }
                  resultListEnd = l1;     
                  l1 = l1->next;
               }
              
              if(l1!=NULL && l2->val <= l1->val)
              {
                  resultListEnd->next = l2;
                  while (l2->next!=NULL && l2->next->val <= l1->val)
                  {
                      l2 = l2->next;
                  }
                  resultListEnd = l2;   
                  l2 = l2->next;
              }          
          }
          // 拼接较长的那个链剩下的串
          resultListEnd->next = (l1==NULL)? l2 : l1;   
          return resultListHead;    
      }
  };
  ```

  
---
title: LeetCode 19. Remove Nth Node From End of List
date: 2018-01-09 18:46:44
category: LeetCode
tags: [leetcode, c++]
math: true
---

## Problem

Give a linked list, remove the $n^{th}$ node from the end of list and return its head.

<!--more-->

### Example

> Given linked list: **1->2->3->4->5**, and **n = 2**
After removing the second node from the end, the linked list becomes **1->2->3->5**.

### Note

Given *n* will always be valid.
Try do this in one pass.

## Answer

### 单链表结构定义

```c++
// Definition for singly-linked list.
struct ListNode {
    int val;
    ListNode *next;
    ListNode(int x) : val(x), next(NULL) {}
};
```

### 普通方法(遍历两次链表)

先遍历一次链表，获取链表的长度，然后第二次遍历链表删除导数第n个元素：

```c++
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n)
    {
        int cnt = 0;
        ListNode *p1 = head, *p2 = head;

        while (p1)
        {
            ++cnt;
            p1 = p1->next;
        }

        int j = 0;
        if (cnt == n) return head->next;
        while (j < cnt - n - 1)
        {
            j++;
            p2 = p2->next;
        }
        auto tmp = p2->next;
        p2->next = p2->next->next;
        delete tmp;
        return head;
    }
};
```

### New Meta(使用二级指针，遍历一次链表)

```c++
class Solution{
public:
    ListNode* removeNthFromEnd(ListNode* head, int n)
    {
        // 利用二级指针删除元素
        ListNode **t1 = &head, *t2 = head;

        // 将t2向后移动n位，此时t2的位置在len(head) - n
        for (int i = 1; i < n; ++i)
            t2 = t2->next;
        // 当t2的下一个节点不为空时，开始移动t1
        while (t2->next)
        {
            // t1指向下一个元素指针的地址
            t1 = &((*t1)->next);
            t2 = t2->next;
        }
        // 当t2为nullptr时，表示t1所指的元素指针的地址即为倒数第n个元素指针的地址
        // 因此我们需要删除该元素,并使t1指向下一个元素指针的地址
        auto tmp = *t1;
        *t1 = (*t1)->next;

        delete tmp;
        return head;
    }
};
```

使用二级指针后可以不用考虑n的位置是否为链表的头或者尾。
---
title: LeetCode 147.insert-sort-list
comments: true
date: 2020-03-25 17:18:23
tags: LeetCode
categories: LeetCode
description: LeetCode第147题题解，包括思路分析和算法实现
---
### insert-sort-list

#### 题目描述：
使用插入排序对链表进行排序。
Sort a linked list using insertion sort.

<!--more-->

#### 思路分析：
- 如果链表为空,返回null
- 链表非空，定义一个头结点，遍历整个链表
- 定义一个变量pre用来指示应该插入的位置，如果这个变量所在位置的值小于待插入元素的值，并且这个变量存在下一个结点，则变量的位置后移一位，最终所在位置即为待插入的位置。
- 用一个变量cur来指示待插入数所在位置，并修改待插入数所指向下一个结点的位置,让它等于pre.next
- 修改带插入位置的下一个结点,让它等于cur

#### 算法实现：

```java
public class Solution {
    public ListNode insertionSortList(ListNode head) {
        if(head == null) return null;
        ListNode pHead = new ListNode(0);
        ListNode cur = head;
        while(cur != null){
            ListNode pre = pHead;
            ListNode next = cur.next;
            while(pre.next != null && pre.next.val < cur.val){
                pre = pre.next;
            }
            cur.next = pre.next;
            pre.next = cur;
            cur = next;
        }
        return pHead.next;
    }
}
```
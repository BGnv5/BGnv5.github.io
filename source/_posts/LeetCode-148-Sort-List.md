---
title: LeetCode 148.Sort List
comments: true
date: 2020-03-25 15:35:45
tags: LeetCode
categories: LeetCode
description: LeetCode 148题题解，包括思路分析和算法实现
---
### sort-list

#### 题目描述：
在O(n log n)的时间内使用常数级空间复杂度对链表进行排序。
Sort a linked list in O(n log n) time using constant space complexity.

<!--more-->

#### 思路分析：

**1. 归并排序（递归法）：**
- 如果链表为空，或者只有一个结点则直接返回这个链表
- 不为空，利用快慢指针把链表为成左右两部分，当快指针所指向结点不为空并且存在下一个元素时，快指针走两步，慢指针走一步。
- 慢指针所指向的下一个结点即为右半部分的开始，并把slow指向的下一个结点置为空。
- 依次递归直到左右两半部分只含有一个结点为止。
- 最后在进行归并。

时间复杂度：O(n log n)
空间复杂度：O(n)

**2. 归并排序（迭代法）：**
- 定义一个头结点，让这个头结点的next指向head
- 统计链表中一共有几个结点，并遍历整个链表
- 每次截取长度为size的链表进行归并，每轮过后size长度*2，直到size长度等于链表的长度，设size初始值为1。
- 最后返回头结点指向的下一个结点 

时间复杂度：O(n log n)               
空间复杂度：O(1)

#### 算法实现：

**1.归并排序（递归法）：**

```java
public class Solution {
    public ListNode sortList(ListNode head) {
        if(head == null || head.next == null) return head;
        ListNode slow = head;
        ListNode fast = head.next;
        while(fast != null && fast.next != null){
            slow = slow.next;
            fast = fast.next.next;
        }
        ListNode mid = slow.next;
        slow.next = null;
        ListNode left = sortList(head);
        ListNode right = sortList(mid);
        ListNode temp = new ListNode(0);
        ListNode p = temp;
        while(left != null && right != null){
            if(left.val < right.val){
                p.next = left;
                left = left.next;
            }else{
                p.next = right;
                right = right.next;
            }
            p = p.next;
        }
        if(left != null) {
            p.next = left;
        }else{
            p.next = right;
        }
        return temp.next;
    }
}
```
**2. 归并排序（迭代法）**
```java
public class Solution {
    public ListNode sortList(ListNode head) {
        //求链表长度
       ListNode p = head;
       int len = 0;
       while(p != null){
           len ++;
           p = p.next;
       }
       
        ListNode pHead = new ListNode(0);
        pHead.next = head;//主要是考虑到只有一个结点的情况
       for(int size = 1; size < len; size *= 2){
           ListNode cur = pHead.next;//一轮结束cur要回到链表首部
           ListNode next = pHead;//一轮结束next回到初始位置
           while(cur != null){
               ListNode left = cur;
               ListNode right = cut(left,size);
               cur = cut(right,size);
               next.next = merge(left,right); 
               while(next.next != null){
                   next = next.next;
               }
           }
       }
        return pHead.next;
    }
    
    //根据步长分割链表
    private ListNode cut(ListNode head, int size){
        if(head == null) return head;
        for (int i = 1; head.next != null && i < size; i++) {
            head = head.next;
        }
        ListNode nextHead = head.next;
        head.next = null;
        return nextHead;
    }
    
    //归并链表
    private ListNode merge(ListNode left, ListNode right){
        ListNode head = new ListNode(0);
        ListNode p = head;
        while(left != null && right != null){
            if(left.val < right.val){
                p.next = left;
                left = left.next;
            }else{
                p.next = right;
                right = right.next;
            }
            p = p.next;
        }
        if(left != null)
            p.next = left;
        else
            p.next = right;
        return head.next;
    }
}
```

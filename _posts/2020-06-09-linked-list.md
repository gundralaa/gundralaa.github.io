---
layout: post
title: "Linked List"
---

Article on Linked List manipulation techniques and examples problems.

#### TODO
* Research different techniques
* Practice various other linked list problems this week

#### Problems
* https://leetcode.com/problems/reverse-linked-list/
* https://leetcode.com/problems/reverse-linked-list-ii/

``` python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next

class Solution:
    # Iterative Solution
    def reverseListIter(self, head: ListNode) -> ListNode:
        if not head:
            return head      
        prev = head
        out = head.next
        while out:
            head = ListNode(out.val, head)
            prev.next = out.next
            out = out.next
        return head
    
    # Recursive Solution
    def reverseList(self, head: ListNode) -> ListNode:
        if not head or not head.next:
            return head
        else:
            cur_node = head
            next_node = head.next
            next_node = self.reverseList(next_node)
            cur_node.next.next, cur_node.next = cur_node, None
            return next_node
```
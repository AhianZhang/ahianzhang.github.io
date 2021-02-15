---
rtitle: "leetcode206"
date: "2019-05-30 09:43:49"
tags: ["LeetCode"]
categories: ["算法"]
---



![](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/Paper.Paper%20%E5%B7%A5%E5%85%B7.21.PNG)

**Consideration** 

- make three pointers , pre,current,next;
- initial pre as null
- use tmp to save current's next node info
- change current's next to link pre node(first is null)
- move pre pointer to next node
- move current pointer to next node

**soultion**

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode reverseList(ListNode head) {
     ListNode pre = null;
        ListNode current = head;

        while (current != null){

            ListNode next = current.next;
            current.next = pre;
            pre = current;
            current = next;
        }
        return pre;
        
    }
}
```


---
title: 从`反转链表`到`反转链表Ⅱ`再到`k个一组翻转链表`
date: 2025-08-23 23:09:26
tags: [算法,链表]
---



## 介绍

在力扣上这三道题是逐层递进的，在解题时进行联系与辨析，运用就会能够更加灵活。
题目我就不给了，这里简要说下：
反转链表：给定head头节点，反转整个链表。

反转链表Ⅱ：给定头节点和索引如（2，5），仅反转从第二个节点到第五个节点之间的部分。

k个一组翻转链表：给定头节点 head 和k，每 k 个节点一组进行翻转。（若不够k则直接返回）

<!-- more -->

## 反转链表

作为基础款，解题当然也是最简单的一部分。

这里直接给出代码：

```java
public ListNode reverseList(ListNode head) {
        ListNode pre=null;
        ListNode cur=head;
        while(cur!=null){
            ListNode temp=cur.next;
            cur.next=pre;
            pre=cur;
            cur=temp;
        }
        return pre;
    }
```

核心还是在循环中首先记录cur的下一位（因为改变了cur的next指针以后就无法访问它了），接着再改变pre、cur；而这也衍生出了递归解法：

```java
public ListNode reverseList(ListNode head) {
        return reverse(null,head);
        
    }
    public ListNode reverse(ListNode pre,ListNode cur){
        if(cur==null){
            return pre;
        }
        ListNode temp=cur.next;
        cur.next=pre;
        return reverse(cur,temp);
    }
```

## 反转链表Ⅱ

在给定索引以后，我们在内部的逻辑依然是遵循反转链表的，只是在内部逻辑执行前与执行后需要稍作改变：

**执行前：**首先需要将p0指针遍历到更改节点的前一位，再从p0.next开始执行逻辑。

**执行后：**对更改后的头指针与尾指针做更改:**尾：**将p0节点的next的next指向cur，**头：** 再将p0的next指向pre。

如图：

![反转链表2](/images/反转链表2.png)

代码：

```java
public ListNode reverseBetween(ListNode head, int left, int right) {
      ListNode dummy=new ListNode();
      dummy.next=head;
      ListNode p0=dummy;
      int temp=left-1;
      int temp1=right-left+1;
      while(temp>0){
        p0=p0.next;
        temp--;
      }
      ListNode pre=null;
      ListNode cur=p0.next;
      while(temp1>0){
        ListNode nxt=cur.next;
        cur.next=pre;
        pre=cur;
        cur=nxt;
        temp1--;
      }
       p0.next.next=cur;
       p0.next=pre;
       return dummy.next;
    }
```



## k个一组翻转链表

该题解答和反转链表Ⅱ相似，只是需要在每k个反转之后再更新p0，而在反转链表Ⅱ解题末尾会更新p0.next，因此需要先用变量nxt记录p0.next，接着在最后将其赋给p0。

而整体就是在while循环内部进行遍历，每k个节点使用for循环遍历，其内部依然使用`反转链表`的逻辑，当出for循环以后就是用`反转链表Ⅱ`的逻辑对指针进行更改。细节是需要额外记录p0.next 用来作为下一次的p0。初始化时是使用dummy节点。

![k个一组](/images/k个一组.png)

以下是代码逻辑：

```
思路：首先遍历得链表的长度，为链表最后面不满k个一组就可以直接退出做准备。接着将虚拟头节点设置为p0，cur、pre同理，进入while循环（条件为len>=k）在进入for循环，对k个节点进行反转，反转逻辑同理。出for之后，就将p0的next记录，然后同反转链表Ⅱ的操作：完善po与po的next指针指向，最后将nxt赋给p0，便于执行下一次k个一组反转。最后退出循环即可。
```

联系：这里在for循环的内部就是反转链表i的操作，而出来之后就是反转链表Ⅱ的操作。

以下是代码：

```java
public ListNode reverseKGroup(ListNode head, int k) {
        if(head==null)return null;
        ListNode dummy=new ListNode(0,head);
        int len=0;
        ListNode cur=head;
        while(cur!=null){
            len++;
            cur=cur.next;
        }
        cur=dummy.next;
        ListNode pre=null;
        ListNode p0=dummy;
        while(len>=k){
            for(int i=0;i<k;i++){
                ListNode temp=cur.next;
                cur.next=pre;
                pre=cur;
                cur=temp;
            }
        ListNode nxt=p0.next;
        p0.next.next=cur;
        p0.next=pre;
        p0=nxt;
        len-=k;
        }
        return dummy.next;
    }
```


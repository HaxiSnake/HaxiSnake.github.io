---
title: 刷题TIPS
date: 2019-11-07 10:20:59
tags:
---

## 链表类题目常用技巧

***
- "Dummy node" 节点技巧

    新建一个空节点，其中的next指向头结点，可以处理很多极端情况，比如涉及到头结点的处理。最后处理完之后返回dummy->next即可

- 双指针技巧

    可以使用双指针，利用双指针之间的距离和指针更新速度解决一些问题，也通用适用于数组类问题。

    - 快慢指针

        快慢指针可以用来解决链表中间节点的相关问题。例如判断是否为回文链表。

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
                bool isPalindrome(ListNode* head) {
                    if(head==NULL || head->next==NULL) return true;
                    ListNode* slow=head;
                    ListNode* fast=head;
                    ListNode* pre=head;
                    ListNode* prepre=NULL;
                    while(fast!=NULL && fast->next!=NULL){
                        pre = slow;
                        slow = slow->next;
                        fast = fast->next->next;
                        pre->next = prepre;
                        prepre = pre;
                    }
                    ListNode* tmp = slow;
                    if(fast!=NULL) tmp=tmp->next;
                    slow = pre;
                    while(slow!=NULL && tmp!=NULL){
                        if(slow->val!=tmp->val) return false;
                        slow = slow->next;
                        tmp = tmp->next;
                    }
                    return true;
                }
            };

- 尾结点的注意事项

    处理链表问题时，尤其是用到多链表指针加原地算法的状况，要注意各个节点尾指针的处理，避免出现循环链表。


## 判断素数

素数一定满足 n%6==1 或者 n%6==5 这个条件。
然后因子只需要遍历到 sqrt(n) 即可。

    bool isPrime(int num) {
        if (num <= 3) {
            return num > 1;
        }
        // 不在6的倍数两侧的一定不是质数
        if (num % 6 != 1 && num % 6 != 5) {
            return false;
        }
        int sqrt = .sqrt(num);
        for (int i = 5; i <= sqrt; i += 6) {
            if (num % i == 0 || num % (i + 2) == 0) {
                return false;
            }
        }
        return true;
    }

## 寻找二进制中1的技巧

利用当前n与n-1做与运算，通过循环即可统计出二进制中1出现的个数。

## heap的使用注意事项

- 创建堆

        vector<int> v;
        make_heap(v.begin(),v.end(),cmp);
- 压入元素

        v.push_back(value);
        push_heap(v.begin(),v.end());
- 弹出元素

        pop_heap(v.begin(),v.end());
        v.pop_back();

注意cmp函数中，A>B是建立小顶堆，A<B是建立大顶堆

## 数学类题目小技巧

- 判断两个数是否同号

    bool sign = (a>0) ^ (b>0);

- 大整数的边界问题

    由于INT_MIN转化为INT_MAX会溢出，所有正数的边界情况较多，这时如果题目逻辑允许，可以从负数的角度来分析问题，简化边界条件。


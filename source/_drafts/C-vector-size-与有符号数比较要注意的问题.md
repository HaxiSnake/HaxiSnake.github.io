---
title: C++ vector size()与有符号数比较要注意的问题
date: 2019-01-14 16:49:26
tags:
- bug
- c++
categories: 
- 调试记录
---
# C++ vector size()与有符号数比较要注意的问题

今天在刷题时要写一个循环体，为了保证能包括初始情况，其中一个循环变量要从j=-1开始计数，但是导致终止条件j<num.size(),一直为false,后经过调试发现是因为vector的size()方法返回的是无符号整形，因此与有符号数进行比较时要显式的进行一次类型转换。即j<(int)num.size(),即要注意下面代码中第一个循环体结束语句的写法。


    ```
        class Solution {
        public:
            int minSubArrayLen(int s, vector<int>& nums) {
                if(0==nums.size()) return 0;
                int sum=0;
                int subarraylen=nums.size()+1;
                for(int i=0,j=-1;(i<nums.size()) && (j<(int)nums.size());){    
                    if(sum<s){
                        sum+=nums[++j];
                    }else{
                        if(i==j){
                            return 1;
                        }
                        subarraylen=(j-i+1)<subarraylen?(j-i+1):subarraylen;  
                        sum-=nums[i++];
                    }
                    
                }
                if(subarraylen!=nums.size()+1){
                    return subarraylen;
                }else{
                    return 0;
                }
            }
        };
    ```
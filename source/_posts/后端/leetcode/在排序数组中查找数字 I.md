---
title: 在排序数组中查找数字 I
date:  2020-06-14T20:20:03
categories: leetcode
tags:
- 每日一题
- 二分查找
---

统计一个数字在排序数组中出现的次数。

**示例 1:**

```
输入: nums = [5,7,7,8,8,10], target = 8
输出: 2
```

**示例 2:**

```
输入: nums = [5,7,7,8,8,10], target = 6
输出: 0
```

**限制：**

`0 <= 数组长度 <= 50000`

## 题目详解

本题数组是一个排序数字，可以采用二分查找的方式定位到target所在的位置。然后基于该位置向前和向后统计该数到数量。

## 代码

```go
func search(nums []int, target int) int {
    if len(nums)==0{
        return 0
    }
    // 通过二分查找定位target
    var start=0
    var end=len(nums)-1
    for start<=end{
        var mid=(start+end)/2
        // 如果相等，则判断相等的个数
        if nums[mid]==target{ 
            return getCount(nums,mid,target)
        }else if nums[mid]<target {
            start=mid+1
        }else{
            end=mid-1
        }
    }
    return 0
}

func getCount(nums []int, mid, target int) int{
    var count int
    for index:=mid;index>=0&&nums[index]==target;{
        count++
        index--
    }
    for index:=mid+1;index<len(nums)&&nums[index]==target;{
        count++
        index++
    }
    return count
}
```




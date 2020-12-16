---
title: leetcode-1
date: 2020-11-19 16:57:44
tags: [leetcode]
catergary: leetcode
---

### 题目链接
[两数之和](https://leetcode-cn.com/problems/two-sum/)

### 题目描述
给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。

你可以假设每种输入只会对应一个答案。但是，数组中同一个元素不能使用两遍。

### 实例
>
>给定 nums = [2, 7, 11, 15], target = 9
>
>因为 nums[0] + nums[1] = 2 + 7 = 9
>所以返回 [0, 1]


### 题目难度
简单

### 解法
```java
public class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> map = new HashMap<>();
        for(int i = 0; i < nums.length; i++) {
            if(map.containsKey(nums[i])) {
                return new int[] {map.get(nums[i]), i};
            } else {
                map.put(target - nums[i], i);
            }
        }
        return new int[]{0, 0};
    }
}
``` 
双循环的暴力解法，时间复杂度是O(n^2)，此解法的时间复杂度为O(n)，但是空间复杂度为O(n);
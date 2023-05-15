---
title: 由“俩数之和问题”所想
date: 2019-01-16 22:01:25
tags: [think]
categories: [leetcode]
---

leetcode 第一个问题的一些反思
<!-- more -->
- 问题
    > Given an array of integers, return indices of the two numbers such that they add up to a specific target.
    > You may assume that each input would have exactly one solution, and you may not use the same element twice.
    >
    > Example:
    > Given nums = [2, 7, 11, 15], target = 9,
    > Because nums[0] + nums[1] = 2 + 7 = 9,
    > return [0, 1].

- 对于基本知识的理解，`Hash Table` 的使用不够灵活
- 对于问题的思考方式，需要把问题简单化，但也不能因为简单而把代码搞复杂

  ```python
  class Solution:
    def twoSum1(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: List[int]
        """
        for i in range(len(nums)):
            for j in range(i+1, len(nums)):
                if (nums[i] + nums[j]) == target:
                    return [i, j]

    def twoSum2(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: List[int]
        """
        d = dict()
        for i in range(len(nums)):
            d[nums[i]] = i

        for i in range(len(nums)):
            a = target - nums[i]
            if a in d.keys() and d[a] != i:
                return [i, d[a]]

    def twoSum(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: List[int]
        """
        d = dict()
        for i in range(len(nums)):
            a = target - nums[i]
            if a in d.keys():
                return [d[a], i]
            d[nums[i]] = i
  ```

---
title: "用Go语言刷LeetCode"
date: 2020-09-15
lastmod: 2020-09-15
draft: false
tags: ["Go","LeetCode"]
categories: ["Go"]
author: "jalen"
---

这篇文章记录了本人通过Go语言来刷LeetCode，解法不能保证是最优解，希望能抛转引玉

# 数据结构与算法题

## 两数之和

[LeetCode链接](https://leetcode-cn.com/problems/two-sum/)

### 题目描述

给定一个整数数组 `nums` 和一个目标值 `target`，请你在该数组中找出和为目标值的那 两个**整数**，并返回他们的数组下标。

你可以假设每种输入只会对应一个答案。但是，数组中同一个元素不能使用两遍。

示例:

```
给定 nums = [2, 7, 11, 15], target = 9

因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]
```

### 我的解法

```go
func twoSum(nums []int, target int) []int {
	var result = make([]int, 2, 2)
	for i, num := range nums {
		for j, num1 := range nums {
			if target-num == num1 {
				return []int{j, i}
			}
		}
	}
	return result
}
```

## 整数反转

[LeetCode链接](https://leetcode-cn.com/problems/reverse-integer/)

### 题目描述

给出一个 32 位的有符号整数，你需要将这个整数中每位上的数字进行反转。

#### 示例 1:

```
输入: 123
输出: 321
```

####  示例 2:

```
输入: -123
输出: -321
```

#### 示例 3:

```
输入: 120
输出: 21
```

#### 注意:

假设我们的环境只能存储得下 32 位的有符号整数，则其数值范围为 [−231,  231 − 1]。请根据这个假设，如果反转后整数溢出那么就返回 0。

### 我的解法

```go
func reverse(x int) int {
    res := 0
	var maxValue = 1<<31 - 1
	var minValue = - 1 << 31
	var pop = 0                     //pop记录上次计算的结果，第一次记录的结果为0
	for ; x != 0; x /= 10 {
		res = pop*10 + x%10
		//判断是否溢出
		if res > maxValue || res < minValue {
			return 0
		}
		pop = res
	}
	return res
}
```


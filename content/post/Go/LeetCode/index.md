---
title: "用Go语言刷LeetCode"
date: 2020-09-15
lastmod: 2020-09-15
draft: false
tags: ["Go","LeetCode"]
categories: ["Go"]
author: "Jalen"


---

这篇文章记录了本人通过Go语言来刷LeetCode，个人能力有限，解法不能保证是最优解，希望能抛转引玉

# 数据结构与算法题

## 1.两数之和

[LeetCode链接](https://leetcode-cn.com/problems/two-sum/)

### 题目描述

给定一个整数数组 `nums` 和一个目标值 `target`，请你在该数组中找出和为目标值的那 两个**整数**，并返回他们的数组下标。

你可以假设每种输入只会对应一个答案。但是，数组中同一个元素不能使用两遍。

**示例**:

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

## 2.整数反转

[LeetCode链接](https://leetcode-cn.com/problems/reverse-integer/)

### 题目描述

给出一个 32 位的有符号整数，你需要将这个整数中每位上的数字进行反转。

**示例 1**:

```
输入: 123
输出: 321
```

**示例 2**:

```
输入: -123
输出: -321
```

**示例 3**:

```
输入: 120
输出: 21
```

**注意**:

假设我们的环境只能存储得下 32 位的有符号整数，则其数值范围为 [−231,  231 − 1]。请根据这个假设，如果反转后整数溢出那么就返回 0。

### 我的解法

```go
/**
思路：比如123 要变成321，循环除以10之后取余数（这样会取到个位上的数），直到被除数x为0，那么会循环3次，依次取到 3 2 1
第一次取到3 计算结果= 0 * 10 + 3 为3
第二次取到2 计算结果= 3 * 10 + 2 为 32
第三次取到1 计算结果= 32 * 10 + 1 为321
*/
func reverse(x int) int {
    res := 0
	var maxValue = 1<<31 - 1
	var minValue = - 1 << 31
	var last = 0 //last记录上次计算的结果，第一次记录的结果为0
	for ; x != 0; x /= 10 { //每循环一次，x的个位数被取出
		res = last*10 + x%10
		//判断是否溢出
		if res > maxValue || res < minValue {
			return 0
		}
		last = res
	}
	return res
}
```

## 3.两数相加

### 题目描述

给出两个 **非空** 的链表用来表示两个非负的整数。其中，它们各自的位数是按照 **逆序** 的方式存储的，并且它们的每个节点只能存储 **一位** 数字。

如果，我们将这两个数相加起来，则会返回一个新的链表来表示它们的和。

您可以假设除了数字 0 之外，这两个数都不会以 0 开头。

**示例：**

```
输入：(2 -> 4 -> 3) + (5 -> 6 -> 4)
输出：7 -> 0 -> 8
原因：342 + 465 = 807
```

本题中的链表定义如下

```go
//定义链表Node
type ListNode struct {
	Val  int
	Next *ListNode
}
```

### 错误思路

题目要求用两数相加的和用新的链表输出，我首先想到的是如下的思路：

> 1.先将输入的两个链表l1和l2分别转换为整数v1和v2
>
> 2.然后将v1+v2的值构成新的链表

代码如下：

```go
func addTwoNumbers(l1 *ListNode, l2 *ListNode) *ListNode {
	var v1, v2 int
	//遍历链表，从L1和L2中取出数字,构成数字v1和v2
	var p = l1
	for i := 0; p != nil; p = p.Next {
		v1 = p.Val*int(math.Pow10(i)) + v1
		i++
	}
	p = l2
	for i := 0; p != nil; p = p.Next {
		v2 = p.Val*int(math.Pow10(i)) + v2
		i++
	}
	var resList = &ListNode{0, nil}
	var curr = resList
	r := v1 + v2 //807 7=>0=>8
	fmt.Println(v1, v2, r)
	//将v1+v2的值从个位开始依次取出，存入到一个新的链表，构成结果链表
	for r != 0 {
		pop := int(r % 10)
		r /= 10
		curr.Val = pop
		if r != 0 {
			curr.Next = &ListNode{}
			curr = curr.Next
		}
	}
	return resList
}
```

这个思路存在致命缺陷：

>1.将链表还原成整数的做法比较耗时，需要用到求指数的函数math.Pow10，计算量很大
>
>2.题目没有限制链表的长度，如果输入的链表很长，那么int类型将不够存
>
>3.进行了3次循环过程，时间复杂度较高

```
由于int类型长度限制，导致测试用例不通过
输入：
[1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1] [5,6,4]
输出：
[-2,-4,-3,-5,-7,-7,-4,-5,-8,-6,-3,0,-2,-7,-3,-3,-2,-2,-9]
预期：
[6,6,4,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,1]
```

原因是第一个链表太长，超出了整数int类型的长度限制。

### 正解

> 从第1个节点开始遍历L1和L2，则每次取到的值必是0-9中的一个，将取到的值相加
>
> 1.如果小于10，则直接保存到结果链表
>
> 2.如果大于10，取个位并置进位carry=1
>
> 3.下次计算如果carry = 1，则加上进位
>
> 4.如果循环结束后，carry = 1,说明最后有一次进位，追加1到末尾

![两数相加图示](http://cdn1.jalen-qian.com/Hugo/tow_sum_20200915174417.png)

```go
func addTwoNumbers(l1 *ListNode, l2 *ListNode) *ListNode {
	var p, q = l1, l2
	//存储进位1
	var carry int
	var dannyHead = &ListNode{}
	curr := dannyHead
    //开始遍历，当且仅当两个链表都为空时，结束循环
	for p != nil || q != nil {
		//分别取Node的值
		var x, y int
		if p != nil {
			x = p.Val
			p = p.Next
		}
		if q != nil {
			y = q.Val
			q = q.Next
		}
        //临时结果:两个链表中取出的值相加，再加上前一步的进位（carry=0相加也无妨）
		val := x + y + carry
		//使用完carry后，将carry清零
		carry = 0
        //如果当前计算结果>=10,说明重新产生了进位，当前应该保存的值是 val % 10
		if val >= 10 {
			val = val % 10
			carry = 1
		}
		//将val保存到Result中
		curr.Next = new(ListNode)
		curr.Next.Val = val
		curr = curr.Next
	}
	//最后如果有进位，则将1存入最后一位
	if carry == 1 {
		curr.Next = &ListNode{1, nil}
	}
	return dannyHead.Next
}
```

## 4.青蛙跳台阶问题

[LeetCode链接](https://leetcode-cn.com/problems/qing-wa-tiao-tai-jie-wen-ti-lcof/)

### 题目描述

> 一只青蛙一次可以跳上1级台阶，也可以跳上2级台阶。求该青蛙跳上一个 n 级的台阶总共有多少种跳法。
>
> 答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。

**示例**

```
示例 1：
输入：n = 2
输出：2

示例 2：
输入：n = 7
输出：21

示例 3：
输入：n = 0
输出：1
提示：
0 <= n <= 100
```

**分析：**

> 此类求 **多少种可能性** 的题目一般都有 **递推性质** ，即 f(n)和f(n-1),f(n-2)...f(1)之间是有联系的

假设青蛙跳到n级台阶的跳法个数为函数$f(n)$

当n=0时，$f(n)=1$

当n=1时，$f(n)=1$

当n=2时，$f(n)=2$，即要么跳两次1级，要么跳1次2级

当n>2时，有如下两种情况：

1. 如果青蛙第一次选择跳1级，那么剩下`n-1`级台阶，跳法个数为`f(n-1)`

2. 如果青蛙第一次选择跳2级，那么剩下`n-2`级台阶，跳法个数为`f(n-2)`

所以当n>2时，总共的跳法个数为：$ f(n) = f(n-1) + f(n-2) $，这个公式也兼容了n=2的情况

讲到这里，我们明确了，青蛙跳法$f(n)$是一个斐波那契数列（只不过和真实的菲波那切数列初始值不同）：


$$
f(n)=\begin{cases} 1，n=0\\\\ 1， n=1  \\\\ f(n-1) + f(n-2), n>=2            \end{cases}
$$


那么这道题就变成了，求斐波那契数列的题了。

### 解法1：通过递归

由于斐波那契数列是一个递归的数列，我们可以通过递归思想计算

```go
func numWays(n int) int {
	if n <= 1 {
		return 1
	}
	return numWays(n-1) % 100000007 + numWays(n-2) % 100000007
}
```

这种方式比较简单，但是存在一个问题，当n越大时，计算的次数是呈指数往上升的，**而且冗余计算很严重**，时间复杂度为$O(2^n)$

### 解法2：线性计算

由于$f(n)$和$f(n-1)$是有关联的，我们可以从$f(2)$开始循环计算到$f(n)$，代码如下：

```go
func numWays(n int) int {
    if n < 2 {
		return 1
	}
	//当n>2的情况
	//stepOne:f(n-1)
	//stepTwo:f(n-2)
	stepOne, stepTwo := 1, 1
	//记录第n种选择
	stepN := 0
	for i := 2; i <= n; i++ {
		stepN = stepOne % 1000000007 + stepTwo % 1000000007 //f(n)=f(n-1) + f(n-2)
		stepTwo = stepOne                                  //下一次循环的f(n-2) = 这一次的f(n-1)
		stepOne = stepN                                    //下一次循环的f(n-1) = 这一次的f(n)
	}
	return stepN % 1000000007
}

```

这种方法的时间复杂度是$O(n)$，空间复杂度是$O(1)$

...持续更新中
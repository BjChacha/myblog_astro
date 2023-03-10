---
layout: '../../layouts/MarkdownPost.astro'
title: '[LeetCode]编辑距离问题'
pubDate: '2020-03-17'
description: 'LeetCode 72. Edit Distance'
author: 'BjChacha'
cover:
tags: ['algorithm', 'dp']
theme: 'light'
featured: true
draft: false
---

## 0. 前言

好久没刷 leetcode 了，加上要准备考研复试，所以决定尝试开个算法系列，每篇解决一道比较经典或者稍有难度的算法题，注重从思路到代码、从无到有的过程。  
其实之前有写一篇类似的刷题总结文章，但只写了题目和代码答案，没有记录思路过程，导致回忆的时候十分痛苦。而且 leetcode 的答案网上一搜一大把，那篇文章没啥意义，所以决定关掉并重开这个系列。  
在这里建议**不要一卡住就看答案**，很多问题卡个一两天很正常。但自己亲手解决每一道算法题真的会带来很大的成就感，这种正反馈也是后续刷题的重要精神源泉。在这里顺便贴一个我觉得不错的学习参考：[labuladong 的算法小抄](https://labuladong.gitbook.io/algo/)。  
这是开篇之作，所以废话很多~~（skr~）~~马上进入本文主题——一道颇有难度的动规[^1]问题：**编辑距离**问题（[72. Edit Distance](https://leetcode.com/problems/edit-distance/)）。

## 1. 问题描述

- 给定两个字符串`s1`和`s2`，计算出将`s1`转换成`s2`所使用的最少操作数。
- 你可以对一个字符串进行以下三种操作：
  - 插入一个字符
  - 删除一个字符
  - 替换一个字符
- 示例 1：
  - 输入：s1 = "horse", s2 = "ros"
  - 输出：3
  - 解释：
    - horse -> rorse （将'h'替换为'r'）
    - rorse -> rose （删除'r'）
    - rose -> ros （删除'e'）
- 示例 2：
  - 输入：s1 = "intention", s2 = "execution"
  - 输出：5
  - 解释：
    - intention -> inention （删除't'）
    - inention -> enention （将'i'替换为'e'）
    - enention -> exention （将'n'替换为'x'）
    - exention -> exection （将'n'替换为'c'）
    - exection -> exection （插入'u'）

## 2. 思路

动态规划是一个好算法，它适合解决**求最值的优化问题**，尤其是对于那些不能很直观地得出方法的问题，第一反应就是考虑用动态规划。动态规划易于实现，只要处理好**分支条件**和**边界条件**，往往就能得出结果。  
![pic1](/static/images/posts/2020-02-17-algorithm-dp-edit-distance/1.png)
但不是所有问题都适用于动态规划。使用动态规划有一个条件，就是问题必须符合**最优子结构**，即**子问题的最优解必须是问题最优解的子集**。比如求全级最高分，可以通过求每班最高分，再从中选最高分。但求全级最大分差就不能这么做了，因为最高分和最低分不一定在同一个班。  
动态规划就是把问题通过递归把问题不断拆分成子问题来求解。显然子问题的优化难度更小，因此递归实质上就是**递归**，但实现的手段不一定是递归，常见的有 dp 表等。

## 3. 代码

回到问题本身，处理两个字符串一般用两个指针各自遍历，这里我们从字符串尾部开始，下面先给出遍历过程的伪代码：

```python
i = len(s1) - 1, j = len(s2) - 1
for i, j to 0:
    if s1[i] == s2[j]:
        skip    # 若字符相同，则不做任何操作
    else:
        insert, delect or replace   # 若字符不相同，则从三种操作中选一
```

这就是动态规划的整体框架，而当中关键就在于“三种操作中选一”。下面我们来讨论一下如何处理这个分支。

- 插入：  
  在 i 指针处插入一个字符后，i 指针指向新插入的字符，即`i=i+1`，`j`不变。
- 删除：  
  在 i 指针处删除一个字符后，i 指针指向前继字符，即`i=i-1`，`j`不变。
- 替换：  
   在 i 指针处替换一个字符后，`i`，`j`位置均不变。
  剩下还有两种边界需要处理，很简单：
- `i`指针先遍历完：  
  此时说明`s1`是`s2`的后置子串，因此只需把`s2`余下字符**插入**到 s1 前面即可。
- `j`指针先遍历完：  
  此时说明`s2`是`s1`的后置子串，因此只需要把`s1`前面的字符**删除**即可。

现在分支逻辑都理清楚了，可以从上面框架中填充条件。

```python
def edit_distance(s1, s2):
    def dp(i, j):
        if i < 0:
            return j + 1    # 插入s2余下字符
        elif j < 0:
            return i + 1    # 删除s1余下字符
        elif s1[i] == s2[j]:
            return dp(i-1, j-1) # 跳过
        else:
            return min(
                dp(i-1, j) + 1, # 删除
                dp(i, j-1) + 1, # 插入
                dp(i-1, j-1) + 1    # 替换
            )

    return dp(len(s1) - 1, len(s2) - 1)
```

到这里上面这段代码已经可以正确求解了，但如果把上面的代码一般是不能通过的，因为太暴力了。  
我们来估算一下时间复杂度：如果每次都执行最后一个分支的话，即**每次执行都最多调用自己三次**，那么整个执行结构就是一个**三叉树**。树的高度取决于字符串的长度。而一个高度为 h 的三叉数最多有$\frac{3^h-1}{2}$个节点，故$O(n)=3^n$。这个时间复杂度就非常夸张了，所以接下来要想办法优化我们的算法。

## 优化

不作优化的动态规划一般会有很严重的冗余。比如这一题中 dp(i-1,j) ->dp(i,j-1)跟 dp(i-1,j-1)重复了，即 dp(i-1,j-1)的值被求了两次。一旦发现出现路线的重叠，那么整个算法就会有大量的重复调用，非常暴力。  
动态规划的优化就是**去冗余**，也就是给上面那棵三叉树剪枝。最常见就是用备忘录去记录每个节点是否被访问过，或者直接用 dp 表直接在数组上操作。  
这里我们在原来的基础上加上备忘录。这里操作的主体有两个：指针 i 和 j，因此备忘录是个二维数组 memo[i][j]。遍历的最长长度就二维数组的大小，故$O(n)=len(s1)*len(s2)$。最后代码如下。

```python
def edit_distance(s1, s2):
    max_len = max(len(s1), len(s2))
    memo = [[max_len for _ in range(len(s2) + 1)] for _ in range(len(s1) + 1)]
    def dp(i, j):
        if i == 0:
            memo[i][j] = j
        elif j == 0:
            memo[i][j] = i
        elif s1[i-1] == s2[j-1]:
            memo[i][j] = memo[i-1][j-1] if memo[i-1][j-1] < max_len else dp(i - 1, j - 1) # skip
        else:
            d1 = memo[i-1][j] + 1 if memo[i-1][j] < max_len else dp(i-1, j) + 1
            d2 = memo[i][j-1] + 1 if memo[i][j-1] < max_len else dp(i, j-1) + 1
            d3 = memo[i-1][j-1] + 1 if memo[i-1][j-1] < max_len else dp(i-1, j-1) + 1
            memo[i][j] = min(d1, d2, d3)

        return memo[i][j]

    return dp(len(s1), len(s2))
```

## 总结

1. 动态规划适合符合最优子结构的最优化问题求解；
2. 动态规划可以使用递归或 dp 表实现，递归简单直观，dp 表比较复杂但执行快。
3. 快速写出循环体，简单呈现整体框架；
4. 动态规划使用递归时需要用备忘录 memo 进行优化。

## 引用

[^1]: 即动态规划啦（其实是想 test 一下 fastpages 的引用功能

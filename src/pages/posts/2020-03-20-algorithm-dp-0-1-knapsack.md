---
layout: '../../layouts/MarkdownPost.astro'
title: '[LeetCode]01背包问题'
pubDate: '2020-03-20'
description: '经典动态规划问题'
author: 'BjChacha'
cover:
tags: ['algorithm', 'dp']
theme: 'light'
featured: true
draft: false
---

## 1. 问题描述

- 给定一个可承重`W`的背包和`N`个物品，其中第`i`个物品的重量为`wt[i]`，价值为`val[i]`。求背包可承载物品的最大价值。
- 示例：
  - 输入：W = 4, N = 3, wt = [2, 1, 3], val = [4, 2, 3]
  - 输出：6
  - 解释：装进背包的物品为`[0, 1]`，重量为`wt[0]+wt[1]=3<W=4`，价值为`val[0]+val[1]=6`。

## 2. 思路

每次放入第`i`个物品，然后求`W-wt[i]`的 01 背包问题，不难看出这是一道非常典型的动态规划问题。接下来就是怎么遍历的问题。  
0-1 背包问题的一个特点，就是物品的状态只有两个：放进背包和不放进背包（这也是 0-1 的由来），故可以先对物品堆进行遍历，防止重复放入同一物品；然后每一个物品中，对每一个**有效背包状态**进行遍历。即这里有两层遍历：

```python
for item in items:
  for stage in stages:
    try to put item in stage
```

这里的“有效背包状态”是什么意思？举个例子，我们第一次向背包放入一个重量为`2`的物品，那么对于下一个物品来说，有效背包状态为`[0,2]`，即不放入第一个物品时背包的负载，以及放入第一个物品时背包的负载。`1`不是有效的背包状态，因为之前没有遍历过重量为`1`的物品。  
另外，每一个背包状态都是以**最大价值**进行记录，所以可以用一个集合`set`来记录有效背包状态，比如`set[4]=5`表示背包负载为`4`时可承载物品的最大价值为`5`。自然，最后答案就是所有状态中的最大值。

## 3. 代码

上面对这次动态规划两层遍历的逻辑就理得差不多了，代码：

```python
def zero_one_knapsack(w, n, wt, val):
    dp = [0 for _ in range(w+1)]  # 每个容量记录其最大容纳价值
    storage = set([0])  # 记录可放入物品的有效背包状态
    for j in range(n):  # 遍历每个物品
        for i in list(storage): # 遍历每个有效背包状态
            if i + wt[j] <= w:  # 若背包可容纳该物品
                dp[i + wt[j]] = max(dp[i + wt[j]], dp[i] + val[j])
                storage.add(i + wt[j])
    return max(dp)
```

## 4. 总结

- 确定主次遍历
- 理清背包状态

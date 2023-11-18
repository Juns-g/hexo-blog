---
title: dp动态规划
tags: 算法
categories: 算法
keywords: 算法 dp 动态规划
cover: "https://dogefs.s3.ladydaily.com/~/source/unsplash/photo-1676816823266-a8bb9a998de7?ixid=M3w0MjI2NjN8MHwxfHRvcGljfHxxUFlzRHp2Sk9ZY3x8fHx8Mnx8MTY4Mzc3MDMyN3w&ixlib=rb-4.0.3&w=2560&h=1440&fmt=webp"
abbrlink: 17372
date: 2023-05-09 16:44:47
---

# dp 动态规划

> https://programmercarl.com/0509.%E6%96%90%E6%B3%A2%E9%82%A3%E5%A5%91%E6%95%B0.html#%E6%80%9D%E8%B7%AF

## 入门

斐波那契数列(直接给出了状态转移方程):`dp[i] = dp[i-1] + dp[i-2]`

[爬楼梯](https://leetcode.cn/problems/climbing-stairs/description/)，一次只能爬 1 层或者 2 层，爬到 n 层有几种方法？

1. 一层：1
2. 二层：1+1,2
3. 三层：一层+2，二层+1
4. 四层：二层+2，三层+1
5. 总结: `dp[i] = dp[i-1] + dp[i-2]`

```js
/**
 * @param {number} n
 * @return {number}
 */
var climbStairs = function (n) {
  let dp = [];
  dp[1] = 1;
  dp[2] = 2;
  for (let i = 3; i <= n; i++) {
    dp[i] = dp[i - 1] + dp[i - 2];
  }
  return dp[n];
};
```

[使用最小花费爬楼梯](https://leetcode.cn/problems/climbing-stairs/description/) : `dp[i] = Math.min(dp[i-1] + cost[i-1], dp[i-2] + cost[i-2])`

[不同路径](https://leetcode.cn/problems/unique-paths/) : `dp[i][j] = dp[i - 1][j] + dp[i][j - 1];`

[不同路径 2](https://leetcode.cn/problems/unique-paths-ii/) : 有障碍的情况下将初始值为 0 就可以

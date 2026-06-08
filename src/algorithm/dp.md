# 动态规划

## 动态规划简介
### 什么是动态规划？
动态规划是一种将复杂问题分解为子问题来解决的方法，通过存储子问题的解来避免重复计算。

### 动态规划的特点
- **最优子结构**：问题的最优解包含子问题的最优解
- **重叠子问题**：不同的问题可能包含相同的子问题
- **无后效性**：当前状态只依赖于之前的状态

### 动态规划解题步骤
1. 定义状态
2. 确定状态转移方程
3. 确定初始状态
4. 确定计算顺序

---

## 1. 斐波那契数列

### 问题描述
计算第n个斐波那契数。

### 递归解法（暴力）
```go
func Fibonacci(n int) int {
    if n <= 1 {
        return n
    }
    return Fibonacci(n-1) + Fibonacci(n-2)
}
```

### 动态规划解法
```go
func FibonacciDP(n int) int {
    if n <= 1 {
        return n
    }
    
    dp := make([]int, n+1)
    dp[0] = 0
    dp[1] = 1
    
    for i := 2; i <= n; i++ {
        dp[i] = dp[i-1] + dp[i-2]
    }
    
    return dp[n]
}
```

### 空间优化
```go
func FibonacciOptimized(n int) int {
    if n <= 1 {
        return n
    }
    
    a, b := 0, 1
    for i := 2; i <= n; i++ {
        a, b = b, a+b
    }
    
    return b
}
```

### 复杂度
- **时间复杂度**：O(n)
- **空间复杂度**：O(1)（优化后）

---

## 2. 最长递增子序列 (LIS)

### 问题描述
找到数组中最长的递增子序列长度。

### 动态规划解法
```go
func LengthOfLIS(nums []int) int {
    if len(nums) == 0 {
        return 0
    }
    
    dp := make([]int, len(nums))
    for i := range dp {
        dp[i] = 1
    }
    
    maxLen := 1
    for i := 1; i < len(nums); i++ {
        for j := 0; j < i; j++ {
            if nums[i] > nums[j] {
                dp[i] = max(dp[i], dp[j]+1)
            }
        }
        maxLen = max(maxLen, dp[i])
    }
    
    return maxLen
}

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

### 二分查找优化
```go
func LengthOfLISOptimized(nums []int) int {
    tails := []int{}
    
    for _, num := range nums {
        left, right := 0, len(tails)
        
        for left < right {
            mid := left + (right-left)/2
            if tails[mid] < num {
                left = mid + 1
            } else {
                right = mid
            }
        }
        
        if left == len(tails) {
            tails = append(tails, num)
        } else {
            tails[left] = num
        }
    }
    
    return len(tails)
}
```

### 复杂度
- **动态规划**：O(n²)
- **二分优化**：O(n log n)

---

## 3. 最长公共子序列 (LCS)

### 问题描述
找到两个字符串的最长公共子序列。

### 动态规划解法
```go
func LongestCommonSubsequence(text1, text2 string) int {
    m, n := len(text1), len(text2)
    
    dp := make([][]int, m+1)
    for i := range dp {
        dp[i] = make([]int, n+1)
    }
    
    for i := 1; i <= m; i++ {
        for j := 1; j <= n; j++ {
            if text1[i-1] == text2[j-1] {
                dp[i][j] = dp[i-1][j-1] + 1
            } else {
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
            }
        }
    }
    
    return dp[m][n]
}
```

### 复杂度
- **时间复杂度**：O(m * n)
- **空间复杂度**：O(m * n)

---

## 4. 背包问题

### 4.1 0-1背包

#### 问题描述
每种物品只能选一次，求最大价值。

#### 动态规划解法
```go
func Knapsack01(weights, values []int, capacity int) int {
    n := len(weights)
    dp := make([]int, capacity+1)
    
    for i := 0; i < n; i++ {
        for j := capacity; j >= weights[i]; j-- {
            dp[j] = max(dp[j], dp[j-weights[i]]+values[i])
        }
    }
    
    return dp[capacity]
}
```

### 4.2 完全背包

#### 问题描述
每种物品可以选多次，求最大价值。

#### 动态规划解法
```go
func KnapsackComplete(weights, values []int, capacity int) int {
    n := len(weights)
    dp := make([]int, capacity+1)
    
    for i := 0; i < n; i++ {
        for j := weights[i]; j <= capacity; j++ {
            dp[j] = max(dp[j], dp[j-weights[i]]+values[i])
        }
    }
    
    return dp[capacity]
}
```

### 复杂度
- **时间复杂度**：O(n * capacity)
- **空间复杂度**：O(capacity)

---

## 5. 爬楼梯

### 问题描述
每次可以爬1或2个台阶，求爬到第n阶的方法数。

### 动态规划解法
```go
func ClimbStairs(n int) int {
    if n <= 2 {
        return n
    }
    
    dp := make([]int, n+1)
    dp[1] = 1
    dp[2] = 2
    
    for i := 3; i <= n; i++ {
        dp[i] = dp[i-1] + dp[i-2]
    }
    
    return dp[n]
}
```

### 空间优化
```go
func ClimbStairsOptimized(n int) int {
    if n <= 2 {
        return n
    }
    
    a, b := 1, 2
    for i := 3; i <= n; i++ {
        a, b = b, a+b
    }
    
    return b
}
```

### 复杂度
- **时间复杂度**：O(n)
- **空间复杂度**：O(1)

---

## 6. 最大子数组和

### 问题描述
找到数组中和最大的连续子数组。

### 动态规划解法（Kadane算法）
```go
func MaxSubArray(nums []int) int {
    maxSum := nums[0]
    currentSum := nums[0]
    
    for i := 1; i < len(nums); i++ {
        currentSum = max(nums[i], currentSum+nums[i])
        maxSum = max(maxSum, currentSum)
    }
    
    return maxSum
}
```

### 复杂度
- **时间复杂度**：O(n)
- **空间复杂度**：O(1)

---

## 7. 编辑距离

### 问题描述
计算将一个字符串转换为另一个字符串所需的最少操作次数。

### 动态规划解法
```go
func MinDistance(word1, word2 string) int {
    m, n := len(word1), len(word2)
    
    dp := make([][]int, m+1)
    for i := range dp {
        dp[i] = make([]int, n+1)
    }
    
    for i := 0; i <= m; i++ {
        dp[i][0] = i
    }
    for j := 0; j <= n; j++ {
        dp[0][j] = j
    }
    
    for i := 1; i <= m; i++ {
        for j := 1; j <= n; j++ {
            if word1[i-1] == word2[j-1] {
                dp[i][j] = dp[i-1][j-1]
            } else {
                dp[i][j] = min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1]) + 1
            }
        }
    }
    
    return dp[m][n]
}

func min(a, b, c int) int {
    if a < b && a < c {
        return a
    }
    if b < c {
        return b
    }
    return c
}
```

### 复杂度
- **时间复杂度**：O(m * n)
- **空间复杂度**：O(m * n)

---

## 动态规划算法对比

| 问题 | 时间复杂度 | 空间复杂度 | 特点 |
|------|----------|-----------|------|
| 斐波那契 | O(n) | O(1) | 简单递推 |
| LIS | O(n²) / O(n log n) | O(n) | 可二分优化 |
| LCS | O(mn) | O(mn) | 二维DP |
| 0-1背包 | O(nc) | O(c) | 逆序遍历 |
| 完全背包 | O(nc) | O(c) | 顺序遍历 |
| 爬楼梯 | O(n) | O(1) | 斐波那契变体 |
| 最大子数组 | O(n) | O(1) | Kadane算法 |
| 编辑距离 | O(mn) | O(mn) | 经典DP |

## 常见面试题

### 1. 动态规划和贪心算法的区别？

**答案：**
- **动态规划**：考虑所有可能的选择，保存子问题的解，保证全局最优
- **贪心算法**：每一步选择当前最优，不考虑全局，可能得不到最优解

### 2. 什么时候使用动态规划？

**答案：**
当问题具有最优子结构和重叠子问题特性时，适合使用动态规划。

### 3. 如何优化动态规划的空间复杂度？

**答案：**
- 滚动数组：用一维数组代替二维数组
- 状态压缩：只保存必要的状态

### 4. 什么是状态转移方程？

**答案：**
状态转移方程描述了当前状态如何从之前的状态转移得到，是动态规划的核心。

### 5. 0-1背包和完全背包的区别？

**答案：**
- **0-1背包**：每种物品只能选一次，内层循环逆序
- **完全背包**：每种物品可以选多次，内层循环顺序

<center>...未完待续...</center>
---  
***  

<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/gitalk@1/dist/gitalk.css">
<script src="https://cdn.jsdelivr.net/npm/gitalk@1/dist/gitalk.min.js"></script>
<div id="gitalk-container"></div>
<script>
  var gitalk = new Gitalk({
    "clientID": "44d7c96f948be236a8c9",
    "clientSecret": "fb9fb3178db6640131c4e3eb69f9449e42bba661",
    "repo": "blog",
    "owner": "Zhang-jie-jun",
    "admin": ["Zhang-jie-jun"],
    "id": location.pathname,      
    "distractionFreeMode": false  
  });
  gitalk.render("gitalk-container");
</script>

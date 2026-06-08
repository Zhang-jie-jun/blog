# 回溯算法

## 回溯算法简介
### 什么是回溯算法？
回溯算法是一种通过探索所有可能的候选解来找到所有解的算法。当探索到某一步发现当前候选解不可能是最终解时，就回溯到上一步尝试其他路径。

### 回溯算法的特点
- **尝试所有可能性**：穷举所有可能的解
- **剪枝优化**：提前排除不可能的路径
- **深度优先搜索**：通常使用递归实现

### 回溯算法适用场景
- **组合问题**：子集、排列、组合求和
- **分割问题**：分割回文串
- **排列问题**：全排列、下一个排列
- **棋盘问题**：N皇后、数独

---

## 1. 子集问题

### 问题描述
找出数组的所有子集（幂集）。

### 实现
```go
func Subsets(nums []int) [][]int {
    result := [][]int{}
    backtrack(nums, 0, []int{}, &result)
    return result
}

func backtrack(nums []int, start int, path []int, result *[][]int) {
    *result = append(*result, append([]int{}, path...))
    
    for i := start; i < len(nums); i++ {
        path = append(path, nums[i])
        backtrack(nums, i+1, path, result)
        path = path[:len(path)-1]
    }
}
```

### 复杂度
- **时间复杂度**：O(n * 2^n)
- **空间复杂度**：O(n)

---

## 2. 组合求和

### 问题描述
找出所有和为目标值的组合，元素可重复使用。

### 实现
```go
func CombinationSum(candidates []int, target int) [][]int {
    result := [][]int{}
    backtrackSum(candidates, target, 0, []int{}, &result)
    return result
}

func backtrackSum(candidates []int, target, start int, path []int, result *[][]int) {
    if target < 0 {
        return
    }
    if target == 0 {
        *result = append(*result, append([]int{}, path...))
        return
    }
    
    for i := start; i < len(candidates); i++ {
        path = append(path, candidates[i])
        backtrackSum(candidates, target-candidates[i], i, path, result)
        path = path[:len(path)-1]
    }
}
```

### 复杂度
- **时间复杂度**：O(n^(target/min))
- **空间复杂度**：O(target/min)

---

## 3. 全排列

### 问题描述
找出数组的所有排列。

### 实现
```go
func Permute(nums []int) [][]int {
    result := [][]int{}
    used := make([]bool, len(nums))
    backtrackPermute(nums, []int{}, used, &result)
    return result
}

func backtrackPermute(nums []int, path []int, used []bool, result *[][]int) {
    if len(path) == len(nums) {
        *result = append(*result, append([]int{}, path...))
        return
    }
    
    for i := 0; i < len(nums); i++ {
        if used[i] {
            continue
        }
        
        used[i] = true
        path = append(path, nums[i])
        backtrackPermute(nums, path, used, &result)
        path = path[:len(path)-1]
        used[i] = false
    }
}
```

### 复杂度
- **时间复杂度**：O(n!)
- **空间复杂度**：O(n)

---

## 4. N皇后问题

### 问题描述
在N×N棋盘上放置N个皇后，使得它们互不攻击。

### 实现
```go
func SolveNQueens(n int) [][]string {
    result := [][]string{}
    board := make([][]byte, n)
    for i := range board {
        board[i] = make([]byte, n)
        for j := range board[i] {
            board[i][j] = '.'
        }
    }
    
    backtrackQueens(n, 0, board, &result)
    return result
}

func backtrackQueens(n, row int, board [][]byte, result *[][]string) {
    if row == n {
        solution := make([]string, n)
        for i := range board {
            solution[i] = string(board[i])
        }
        *result = append(*result, solution)
        return
    }
    
    for col := 0; col < n; col++ {
        if isValid(board, row, col) {
            board[row][col] = 'Q'
            backtrackQueens(n, row+1, board, result)
            board[row][col] = '.'
        }
    }
}

func isValid(board [][]byte, row, col int) bool {
    // 检查列
    for i := 0; i < row; i++ {
        if board[i][col] == 'Q' {
            return false
        }
    }
    
    // 检查左上方
    for i, j := row-1, col-1; i >= 0 && j >= 0; i, j = i-1, j-1 {
        if board[i][j] == 'Q' {
            return false
        }
    }
    
    // 检查右上方
    for i, j := row-1, col+1; i >= 0 && j < len(board); i, j = i-1, j+1 {
        if board[i][j] == 'Q' {
            return false
        }
    }
    
    return true
}
```

### 复杂度
- **时间复杂度**：O(n!)
- **空间复杂度**：O(n²)

---

## 5. 分割回文串

### 问题描述
将字符串分割成若干回文子串。

### 实现
```go
func Partition(s string) [][]string {
    result := [][]string{}
    backtrackPartition(s, 0, []string{}, &result)
    return result
}

func backtrackPartition(s string, start int, path []string, result *[][]string) {
    if start == len(s) {
        *result = append(*result, append([]string{}, path...))
        return
    }
    
    for i := start; i < len(s); i++ {
        if isPalindrome(s, start, i) {
            path = append(path, s[start:i+1])
            backtrackPartition(s, i+1, path, result)
            path = path[:len(path)-1]
        }
    }
}

func isPalindrome(s string, left, right int) bool {
    for left < right {
        if s[left] != s[right] {
            return false
        }
        left++
        right--
    }
    return true
}
```

### 复杂度
- **时间复杂度**：O(n * 2^n)
- **空间复杂度**：O(n)

---

## 6. 单词搜索

### 问题描述
在二维网格中搜索单词。

### 实现
```go
func Exist(board [][]byte, word string) bool {
    for i := 0; i < len(board); i++ {
        for j := 0; j < len(board[0]); j++ {
            if backtrackWord(board, word, i, j, 0) {
                return true
            }
        }
    }
    return false
}

func backtrackWord(board [][]byte, word string, i, j, k int) bool {
    if k == len(word) {
        return true
    }
    
    if i < 0 || i >= len(board) || j < 0 || j >= len(board[0]) {
        return false
    }
    
    if board[i][j] != word[k] {
        return false
    }
    
    temp := board[i][j]
    board[i][j] = '#'
    
    found := backtrackWord(board, word, i+1, j, k+1) ||
             backtrackWord(board, word, i-1, j, k+1) ||
             backtrackWord(board, word, i, j+1, k+1) ||
             backtrackWord(board, word, i, j-1, k+1)
    
    board[i][j] = temp
    
    return found
}
```

### 复杂度
- **时间复杂度**：O(mn * 3^L)（L为单词长度）
- **空间复杂度**：O(L)

---

## 回溯算法对比

| 问题 | 时间复杂度 | 空间复杂度 | 描述 |
|------|----------|-----------|------|
| 子集 | O(n*2^n) | O(n) | 幂集 |
| 组合求和 | O(n^(target/min)) | O(target/min) | 可重复选择 |
| 全排列 | O(n!) | O(n) | 排列组合 |
| N皇后 | O(n!) | O(n²) | 棋盘问题 |
| 分割回文串 | O(n*2^n) | O(n) | 字符串分割 |
| 单词搜索 | O(mn*3^L) | O(L) | 网格搜索 |

## 常见面试题

### 1. 回溯算法和深度优先搜索的关系？

**答案：**
回溯算法是一种特殊的深度优先搜索，在搜索过程中会"回溯"——当发现当前路径无法到达目标时，撤销上一步的选择，尝试其他路径。

### 2. 如何进行剪枝优化？

**答案：**
剪枝是在搜索过程中提前排除不可能的路径，常见方法：
- 可行性剪枝：提前判断当前路径是否可行
- 最优性剪枝：如果当前解已经不可能优于已知最优解，停止搜索
- 顺序优化：优先搜索更可能找到解的分支

### 3. 子集问题和组合问题的区别？

**答案：**
- **子集问题**：元素不考虑顺序，每个元素可选可不选
- **组合问题**：通常有目标约束（如和为target），元素选择有条件限制

### 4. 排列问题如何处理重复元素？

**答案：**
先排序，然后在选择元素时跳过重复元素：

```go
func PermuteUnique(nums []int) [][]int {
    sort.Ints(nums)
    result := [][]int{}
    used := make([]bool, len(nums))
    backtrackPermuteUnique(nums, []int{}, used, &result)
    return result
}

func backtrackPermuteUnique(nums []int, path []int, used []bool, result *[][]int) {
    if len(path) == len(nums) {
        *result = append(*result, append([]int{}, path...))
        return
    }
    
    for i := 0; i < len(nums); i++ {
        if used[i] || (i > 0 && nums[i] == nums[i-1] && !used[i-1]) {
            continue
        }
        
        used[i] = true
        path = append(path, nums[i])
        backtrackPermuteUnique(nums, path, used, result)
        path = path[:len(path)-1]
        used[i] = false
    }
}
```

### 5. N皇后问题的时间复杂度为什么是O(n!)？

**答案：**
第一行有n种选择，第二行最多有n-2种选择（排除同一列和对角线），第三行最多有n-4种选择，以此类推，总共有约n!种可能的放置方式。

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

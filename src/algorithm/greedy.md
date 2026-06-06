# 贪心算法

## 贪心算法简介
### 什么是贪心算法？
贪心算法是一种在每一步选择中都采取当前状态下最优的选择，从而希望最终得到全局最优解的算法。

### 贪心算法的特点
- **局部最优**：每一步选择当前最优
- **无回溯**：一旦做出选择就不再改变
- **不一定全局最优**：需要问题具备贪心选择性质

### 贪心算法适用条件
1. **贪心选择性质**：局部最优选择能导致全局最优
2. **最优子结构**：问题的最优解包含子问题的最优解

---

## 1. 活动选择问题

### 问题描述
有若干活动，每个活动有开始时间和结束时间，选择最多的互不重叠的活动。

### 贪心策略
按结束时间排序，每次选择结束最早的活动。

### 实现
```go
type Activity struct {
    start, end int
}

func ActivitySelection(activities []Activity) []Activity {
    // 按结束时间排序
    sort.Slice(activities, func(i, j int) bool {
        return activities[i].end < activities[j].end
    })
    
    result := []Activity{}
    lastEnd := -1
    
    for _, activity := range activities {
        if activity.start >= lastEnd {
            result = append(result, activity)
            lastEnd = activity.end
        }
    }
    
    return result
}
```

### 复杂度
- **时间复杂度**：O(n log n)
- **空间复杂度**：O(1)

---

## 2. 区间调度问题

### 问题描述
选择最少的点，使得每个区间至少包含一个点。

### 贪心策略
按右端点排序，每次选择右端点作为点。

### 实现
```go
func IntervalPoint(intervals [][]int) []int {
    sort.Slice(intervals, func(i, j int) bool {
        return intervals[i][1] < intervals[j][1]
    })
    
    points := []int{}
    lastPoint := -1
    
    for _, interval := range intervals {
        if interval[0] > lastPoint {
            lastPoint = interval[1]
            points = append(points, lastPoint)
        }
    }
    
    return points
}
```

### 复杂度
- **时间复杂度**：O(n log n)
- **空间复杂度**：O(1)

---

## 3. 哈夫曼编码

### 问题描述
根据字符出现频率构建最优前缀编码树。

### 贪心策略
每次选择频率最低的两个节点合并。

### 实现
```go
import "container/heap"

type HuffmanNode struct {
    char  rune
    freq  int
    left  *HuffmanNode
    right *HuffmanNode
}

type PriorityQueue []*HuffmanNode

func (pq PriorityQueue) Len() int { return len(pq) }

func (pq PriorityQueue) Less(i, j int) bool {
    return pq[i].freq < pq[j].freq
}

func (pq PriorityQueue) Swap(i, j int) {
    pq[i], pq[j] = pq[j], pq[i]
}

func (pq *PriorityQueue) Push(x interface{}) {
    *pq = append(*pq, x.(*HuffmanNode))
}

func (pq *PriorityQueue) Pop() interface{} {
    old := *pq
    n := len(old)
    item := old[n-1]
    *pq = old[:n-1]
    return item
}

func BuildHuffmanTree(freq map[rune]int) *HuffmanNode {
    pq := make(PriorityQueue, 0)
    heap.Init(&pq)
    
    for char, f := range freq {
        heap.Push(&pq, &HuffmanNode{char: char, freq: f})
    }
    
    for pq.Len() > 1 {
        left := heap.Pop(&pq).(*HuffmanNode)
        right := heap.Pop(&pq).(*HuffmanNode)
        
        merged := &HuffmanNode{
            freq:  left.freq + right.freq,
            left:  left,
            right: right,
        }
        
        heap.Push(&pq, merged)
    }
    
    if pq.Len() == 0 {
        return nil
    }
    return heap.Pop(&pq).(*HuffmanNode)
}
```

### 复杂度
- **时间复杂度**：O(n log n)
- **空间复杂度**：O(n)

---

## 4. 硬币找零问题

### 问题描述
用最少的硬币凑成指定金额。

### 贪心策略
每次选择面值最大的硬币。

### 实现
```go
func CoinChange(coins []int, amount int) int {
    sort.Slice(coins, func(i, j int) bool {
        return coins[i] > coins[j]
    })
    
    count := 0
    remaining := amount
    
    for _, coin := range coins {
        if remaining <= 0 {
            break
        }
        
        num := remaining / coin
        count += num
        remaining -= num * coin
    }
    
    if remaining != 0 {
        return -1 // 无法凑成
    }
    
    return count
}
```

### 注意
贪心算法不总能得到最优解，取决于硬币面值系统。

### 复杂度
- **时间复杂度**：O(n log n)
- **空间复杂度**：O(1)

---

## 5. 跳跃游戏

### 问题描述
判断能否到达数组最后一个位置。

### 贪心策略
维护能到达的最远位置。

### 实现
```go
func CanJump(nums []int) bool {
    maxReach := 0
    
    for i := 0; i < len(nums); i++ {
        if i > maxReach {
            return false
        }
        maxReach = max(maxReach, i+nums[i])
        if maxReach >= len(nums)-1 {
            return true
        }
    }
    
    return false
}
```

### 复杂度
- **时间复杂度**：O(n)
- **空间复杂度**：O(1)

---

## 6. 最大子数组和

### 问题描述
找到数组中和最大的连续子数组。

### 贪心策略（Kadane算法）
维护当前子数组和，若为负则重新开始。

### 实现
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

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

### 复杂度
- **时间复杂度**：O(n)
- **空间复杂度**：O(1)

---

## 7. 合并区间

### 问题描述
合并重叠的区间。

### 贪心策略
按起点排序，依次合并重叠区间。

### 实现
```go
func Merge(intervals [][]int) [][]int {
    if len(intervals) == 0 {
        return intervals
    }
    
    sort.Slice(intervals, func(i, j int) bool {
        return intervals[i][0] < intervals[j][0]
    })
    
    result := [][]int{intervals[0]}
    
    for i := 1; i < len(intervals); i++ {
        last := result[len(result)-1]
        
        if intervals[i][0] <= last[1] {
            last[1] = max(last[1], intervals[i][1])
        } else {
            result = append(result, intervals[i])
        }
    }
    
    return result
}
```

### 复杂度
- **时间复杂度**：O(n log n)
- **空间复杂度**：O(1)

---

## 贪心算法对比

| 问题 | 贪心策略 | 时间复杂度 | 空间复杂度 |
|------|---------|----------|-----------|
| 活动选择 | 选择结束最早的 | O(n log n) | O(1) |
| 区间调度 | 选择右端点 | O(n log n) | O(1) |
| 哈夫曼编码 | 合并频率最低的 | O(n log n) | O(n) |
| 硬币找零 | 选择面值最大的 | O(n log n) | O(1) |
| 跳跃游戏 | 维护最远可达 | O(n) | O(1) |
| 最大子数组 | Kadane算法 | O(n) | O(1) |
| 合并区间 | 按起点排序合并 | O(n log n) | O(1) |

## 常见面试题

### 1. 贪心算法和动态规划的区别？

**答案：**
- **贪心**：每步选择局部最优，无回溯，可能不全局最优
- **动态规划**：保存子问题解，保证全局最优，有回溯

### 2. 什么时候贪心算法能得到最优解？

**答案：**
当问题具备贪心选择性质和最优子结构时，贪心算法能得到最优解。

### 3. 硬币找零问题为什么贪心不一定有效？

**答案：**
贪心算法依赖于硬币面值系统。例如，面值为1, 3, 4时，凑6元：贪心选4+1+1=3枚，最优解是3+3=2枚。

### 4. 活动选择问题为什么按结束时间排序？

**答案：**
选择结束最早的活动可以留下更多时间给后续活动，从而选择更多活动。

### 5. 哈夫曼编码的最优性如何证明？

**答案：**
哈夫曼编码通过每次合并频率最低的节点，保证了带权路径长度最小，是最优前缀编码。

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
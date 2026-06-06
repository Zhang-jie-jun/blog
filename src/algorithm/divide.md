# 分治算法

## 分治算法简介
### 什么是分治算法？
分治算法是一种将复杂问题分解为多个子问题来解决的方法，递归地解决每个子问题，然后将子问题的解合并得到原问题的解。

### 分治算法的特点
- **分解**：将问题分解为较小的子问题
- **解决**：递归地解决子问题
- **合并**：将子问题的解合并为原问题的解

### 分治算法适用条件
1. **问题可以分解**：原问题可以分解为多个相似的子问题
2. **子问题独立**：子问题之间相互独立
3. **子问题可合并**：子问题的解可以合并为原问题的解

---

## 1. 归并排序 (Merge Sort)

### 原理
将数组分成两半，分别排序后合并。

### 实现
```go
func MergeSort(arr []int) []int {
    if len(arr) <= 1 {
        return arr
    }
    
    mid := len(arr) / 2
    left := MergeSort(arr[:mid])
    right := MergeSort(arr[mid:])
    
    return merge(left, right)
}

func merge(left, right []int) []int {
    result := make([]int, 0, len(left)+len(right))
    i, j := 0, 0
    
    for i < len(left) && j < len(right) {
        if left[i] <= right[j] {
            result = append(result, left[i])
            i++
        } else {
            result = append(result, right[j])
            j++
        }
    }
    
    result = append(result, left[i:]...)
    result = append(result, right[j:]...)
    return result
}
```

### 复杂度
- **时间复杂度**：O(n log n)
- **空间复杂度**：O(n)

---

## 2. 快速排序 (Quick Sort)

### 原理
选择基准元素，将数组分成两部分，左边小于基准，右边大于基准，递归排序。

### 实现
```go
func QuickSort(arr []int) {
    quickSortHelper(arr, 0, len(arr)-1)
}

func quickSortHelper(arr []int, low, high int) {
    if low < high {
        pivot := partition(arr, low, high)
        quickSortHelper(arr, low, pivot-1)
        quickSortHelper(arr, pivot+1, high)
    }
}

func partition(arr []int, low, high int) int {
    pivot := arr[high]
    i := low - 1
    
    for j := low; j < high; j++ {
        if arr[j] < pivot {
            i++
            arr[i], arr[j] = arr[j], arr[i]
        }
    }
    
    arr[i+1], arr[high] = arr[high], arr[i+1]
    return i + 1
}
```

### 复杂度
- **时间复杂度**：O(n log n)（平均），O(n²)（最坏）
- **空间复杂度**：O(log n)

---

## 3. 二分查找 (Binary Search)

### 原理
利用数组有序性，每次将搜索范围缩小一半。

### 实现
```go
func BinarySearch(arr []int, target int) int {
    left, right := 0, len(arr)-1
    
    for left <= right {
        mid := left + (right-left)/2
        
        if arr[mid] == target {
            return mid
        } else if arr[mid] < target {
            left = mid + 1
        } else {
            right = mid - 1
        }
    }
    
    return -1
}
```

### 复杂度
- **时间复杂度**：O(log n)
- **空间复杂度**：O(1)

---

## 4. 最大子数组和 (Divide and Conquer)

### 原理
将数组分成两半，最大子数组可能在左半部分、右半部分或跨越中间。

### 实现
```go
func MaxSubArrayDivide(nums []int) int {
    if len(nums) == 0 {
        return 0
    }
    return maxSubArrayHelper(nums, 0, len(nums)-1)
}

func maxSubArrayHelper(nums []int, left, right int) int {
    if left == right {
        return nums[left]
    }
    
    mid := left + (right-left)/2
    
    leftMax := maxSubArrayHelper(nums, left, mid)
    rightMax := maxSubArrayHelper(nums, mid+1, right)
    crossMax := maxCrossingSum(nums, left, mid, right)
    
    return max(leftMax, rightMax, crossMax)
}

func maxCrossingSum(nums []int, left, mid, right int) int {
    leftSum := int(^uint(0) >> 1 * -1)
    sum := 0
    
    for i := mid; i >= left; i-- {
        sum += nums[i]
        if sum > leftSum {
            leftSum = sum
        }
    }
    
    rightSum := int(^uint(0) >> 1 * -1)
    sum = 0
    
    for i := mid + 1; i <= right; i++ {
        sum += nums[i]
        if sum > rightSum {
            rightSum = sum
        }
    }
    
    return leftSum + rightSum
}

func max(a, b, c int) int {
    if a >= b && a >= c {
        return a
    }
    if b >= c {
        return b
    }
    return c
}
```

### 复杂度
- **时间复杂度**：O(n log n)
- **空间复杂度**：O(log n)

---

## 5. 最接近的三数之和

### 原理
排序后使用双指针法，结合分治思想。

### 实现
```go
import "sort"

func ThreeSumClosest(nums []int, target int) int {
    sort.Ints(nums)
    closest := nums[0] + nums[1] + nums[2]
    
    for i := 0; i < len(nums)-2; i++ {
        left, right := i+1, len(nums)-1
        
        for left < right {
            sum := nums[i] + nums[left] + nums[right]
            
            if abs(sum-target) < abs(closest-target) {
                closest = sum
            }
            
            if sum < target {
                left++
            } else if sum > target {
                right--
            } else {
                return target
            }
        }
    }
    
    return closest
}

func abs(a int) int {
    if a < 0 {
        return -a
    }
    return a
}
```

### 复杂度
- **时间复杂度**：O(n²)
- **空间复杂度**：O(1)

---

## 6. 合并K个升序链表

### 原理
分治合并，将链表两两合并。

### 实现
```go
type ListNode struct {
    Val  int
    Next *ListNode
}

func MergeKLists(lists []*ListNode) *ListNode {
    if len(lists) == 0 {
        return nil
    }
    if len(lists) == 1 {
        return lists[0]
    }
    
    mid := len(lists) / 2
    left := MergeKLists(lists[:mid])
    right := MergeKLists(lists[mid:])
    
    return mergeTwoLists(left, right)
}

func mergeTwoLists(l1, l2 *ListNode) *ListNode {
    dummy := &ListNode{}
    current := dummy
    
    for l1 != nil && l2 != nil {
        if l1.Val < l2.Val {
            current.Next = l1
            l1 = l1.Next
        } else {
            current.Next = l2
            l2 = l2.Next
        }
        current = current.Next
    }
    
    if l1 != nil {
        current.Next = l1
    }
    if l2 != nil {
        current.Next = l2
    }
    
    return dummy.Next
}
```

### 复杂度
- **时间复杂度**：O(n log k)（n为总节点数，k为链表数）
- **空间复杂度**：O(log k)

---

## 7. 数组中的第K个最大元素

### 原理
利用快速选择算法，基于分治思想。

### 实现
```go
func FindKthLargest(nums []int, k int) int {
    return quickSelect(nums, 0, len(nums)-1, len(nums)-k)
}

func quickSelect(nums []int, left, right, k int) int {
    if left == right {
        return nums[left]
    }
    
    pivotIndex := partition(nums, left, right)
    
    if pivotIndex == k {
        return nums[k]
    } else if pivotIndex < k {
        return quickSelect(nums, pivotIndex+1, right, k)
    } else {
        return quickSelect(nums, left, pivotIndex-1, k)
    }
}
```

### 复杂度
- **时间复杂度**：O(n)（平均），O(n²)（最坏）
- **空间复杂度**：O(log n)

---

## 分治算法对比

| 算法 | 时间复杂度 | 空间复杂度 | 描述 |
|------|----------|-----------|------|
| 归并排序 | O(n log n) | O(n) | 稳定排序 |
| 快速排序 | O(n log n) | O(log n) | 不稳定排序 |
| 二分查找 | O(log n) | O(1) | 有序数组查找 |
| 最大子数组 | O(n log n) | O(log n) | 分治解法 |
| 三数之和 | O(n²) | O(1) | 双指针 |
| 合并K链表 | O(n log k) | O(log k) | 分组合并 |
| 快速选择 | O(n) | O(log n) | 第K大元素 |

## 常见面试题

### 1. 分治算法和动态规划的区别？

**答案：**
- **分治**：子问题相互独立，递归求解后合并
- **动态规划**：子问题重叠，保存子问题解避免重复计算

### 2. 快速排序和归并排序的区别？

**答案：**
- **快速排序**：原地排序，不稳定，平均O(n log n)
- **归并排序**：需要额外空间，稳定，始终O(n log n)

### 3. 二分查找的前提条件是什么？

**答案：**
数组必须是有序的，且支持随机访问。

### 4. 快速选择算法的原理是什么？

**答案：**
快速选择基于快速排序的分区思想，每次分区后根据基准位置判断目标在左半部分还是右半部分，只递归处理包含目标的部分，平均时间复杂度O(n)。

### 5. 合并K个有序链表有哪些方法？

**答案：**
- **分治法**：两两合并，时间O(n log k)
- **优先队列**：维护k个指针，时间O(n log k)
- **顺序合并**：逐个合并，时间O(kn)

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
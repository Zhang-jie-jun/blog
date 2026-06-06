# 排序算法

## 排序算法简介
### 什么是排序算法？
排序算法是将一组数据按照特定顺序（如升序或降序）排列的算法。排序是计算机科学中最基础也是最重要的操作之一。

### 排序算法分类
- **比较类排序**：通过比较元素大小来决定顺序，时间复杂度通常为 O(n log n) ~ O(n²)
- **非比较类排序**：不通过比较，而是利用数据的某些特性来排序，时间复杂度可以达到 O(n)

### 算法性能指标
- **时间复杂度**：最坏、平均、最好情况下的运行时间
- **空间复杂度**：算法所需的额外空间
- **稳定性**：相等元素的相对顺序是否保持不变

---

## 1. 冒泡排序 (Bubble Sort)

### 原理
重复遍历数组，比较相邻元素，如果顺序错误则交换它们。

### 实现
```go
func BubbleSort(arr []int) {
    n := len(arr)
    for i := 0; i < n-1; i++ {
        swapped := false
        for j := 0; j < n-1-i; j++ {
            if arr[j] > arr[j+1] {
                arr[j], arr[j+1] = arr[j+1], arr[j]
                swapped = true
            }
        }
        if !swapped {
            break // 提前终止
        }
    }
}
```

### 复杂度
- **时间复杂度**：O(n²)
- **空间复杂度**：O(1)
- **稳定性**：稳定

---

## 2. 选择排序 (Selection Sort)

### 原理
每次从未排序部分选择最小（或最大）的元素放到已排序部分的末尾。

### 实现
```go
func SelectionSort(arr []int) {
    n := len(arr)
    for i := 0; i < n-1; i++ {
        minIdx := i
        for j := i+1; j < n; j++ {
            if arr[j] < arr[minIdx] {
                minIdx = j
            }
        }
        arr[i], arr[minIdx] = arr[minIdx], arr[i]
    }
}
```

### 复杂度
- **时间复杂度**：O(n²)
- **空间复杂度**：O(1)
- **稳定性**：不稳定

---

## 3. 插入排序 (Insertion Sort)

### 原理
将未排序元素逐个插入到已排序部分的正确位置。

### 实现
```go
func InsertionSort(arr []int) {
    n := len(arr)
    for i := 1; i < n; i++ {
        key := arr[i]
        j := i - 1
        for j >= 0 && arr[j] > key {
            arr[j+1] = arr[j]
            j--
        }
        arr[j+1] = key
    }
}
```

### 复杂度
- **时间复杂度**：O(n²)（最好 O(n)）
- **空间复杂度**：O(1)
- **稳定性**：稳定

---

## 4. 希尔排序 (Shell Sort)

### 原理
将数组分成多个子序列进行插入排序，逐步减小间隔直至为1。

### 实现
```go
func ShellSort(arr []int) {
    n := len(arr)
    gap := n / 2
    
    for gap > 0 {
        for i := gap; i < n; i++ {
            temp := arr[i]
            j := i
            for j >= gap && arr[j-gap] > temp {
                arr[j] = arr[j-gap]
                j -= gap
            }
            arr[j] = temp
        }
        gap /= 2
    }
}
```

### 复杂度
- **时间复杂度**：O(n log n) ~ O(n²)
- **空间复杂度**：O(1)
- **稳定性**：不稳定

---

## 5. 快速排序 (Quick Sort)

### 原理
选择一个基准元素，将数组分成两部分，左边小于基准，右边大于基准，然后递归排序。

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
- **时间复杂度**：O(n log n)（最坏 O(n²)）
- **空间复杂度**：O(log n)
- **稳定性**：不稳定

---

## 6. 归并排序 (Merge Sort)

### 原理
将数组分成两半，分别排序后合并成一个有序数组。

### 实现
```go
func MergeSort(arr []int) []int {
    n := len(arr)
    if n <= 1 {
        return arr
    }
    
    mid := n / 2
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
- **稳定性**：稳定

---

## 7. 堆排序 (Heap Sort)

### 原理
利用堆数据结构，每次提取堆顶元素放到数组末尾。

### 实现
```go
func HeapSort(arr []int) {
    n := len(arr)
    
    // 构建最大堆
    for i := n/2 - 1; i >= 0; i-- {
        heapify(arr, n, i)
    }
    
    // 逐个提取元素
    for i := n - 1; i > 0; i-- {
        arr[0], arr[i] = arr[i], arr[0]
        heapify(arr, i, 0)
    }
}

func heapify(arr []int, n, i int) {
    largest := i
    left := 2*i + 1
    right := 2*i + 2
    
    if left < n && arr[left] > arr[largest] {
        largest = left
    }
    
    if right < n && arr[right] > arr[largest] {
        largest = right
    }
    
    if largest != i {
        arr[i], arr[largest] = arr[largest], arr[i]
        heapify(arr, n, largest)
    }
}
```

### 复杂度
- **时间复杂度**：O(n log n)
- **空间复杂度**：O(1)
- **稳定性**：不稳定

---

## 8. 计数排序 (Counting Sort)

### 原理
统计每个元素出现的次数，然后按顺序输出。

### 实现
```go
func CountingSort(arr []int) []int {
    if len(arr) == 0 {
        return arr
    }
    
    // 找到最大值和最小值
    min, max := arr[0], arr[0]
    for _, num := range arr {
        if num < min {
            min = num
        }
        if num > max {
            max = num
        }
    }
    
    // 创建计数数组
    count := make([]int, max-min+1)
    for _, num := range arr {
        count[num-min]++
    }
    
    // 重建数组
    result := make([]int, 0, len(arr))
    for i, cnt := range count {
        for j := 0; j < cnt; j++ {
            result = append(result, i+min)
        }
    }
    
    return result
}
```

### 复杂度
- **时间复杂度**：O(n + k)（k 为值域大小）
- **空间复杂度**：O(k)
- **稳定性**：稳定

---

## 9. 桶排序 (Bucket Sort)

### 原理
将元素分到多个桶中，每个桶单独排序，最后合并。

### 实现
```go
func BucketSort(arr []float64) []float64 {
    if len(arr) == 0 {
        return arr
    }
    
    // 找到最大值和最小值
    min, max := arr[0], arr[0]
    for _, num := range arr {
        if num < min {
            min = num
        }
        if num > max {
            max = num
        }
    }
    
    // 创建桶
    bucketCount := 5
    buckets := make([][]float64, bucketCount)
    
    // 分配元素到桶
    for _, num := range arr {
        index := int((num - min) / (max - min + 1) * float64(bucketCount))
        buckets[index] = append(buckets[index], num)
    }
    
    // 排序每个桶并合并
    result := make([]float64, 0, len(arr))
    for _, bucket := range buckets {
        InsertionSortFloat(bucket)
        result = append(result, bucket...)
    }
    
    return result
}

func InsertionSortFloat(arr []float64) {
    n := len(arr)
    for i := 1; i < n; i++ {
        key := arr[i]
        j := i - 1
        for j >= 0 && arr[j] > key {
            arr[j+1] = arr[j]
            j--
        }
        arr[j+1] = key
    }
}
```

### 复杂度
- **时间复杂度**：O(n + k)（平均），O(n²)（最坏）
- **空间复杂度**：O(n + k)
- **稳定性**：稳定

---

## 10. 基数排序 (Radix Sort)

### 原理
按数字的每一位进行排序，从最低位到最高位。

### 实现
```go
func RadixSort(arr []int) {
    if len(arr) == 0 {
        return
    }
    
    // 找到最大值
    max := arr[0]
    for _, num := range arr {
        if num > max {
            max = num
        }
    }
    
    // 按每一位排序
    for exp := 1; max/exp > 0; exp *= 10 {
        countingSortByDigit(arr, exp)
    }
}

func countingSortByDigit(arr []int, exp int) {
    n := len(arr)
    output := make([]int, n)
    count := make([]int, 10)
    
    // 统计每个数字出现的次数
    for i := 0; i < n; i++ {
        index := (arr[i] / exp) % 10
        count[index]++
    }
    
    // 计算累积次数
    for i := 1; i < 10; i++ {
        count[i] += count[i-1]
    }
    
    // 构建输出数组
    for i := n - 1; i >= 0; i-- {
        index := (arr[i] / exp) % 10
        output[count[index]-1] = arr[i]
        count[index]--
    }
    
    // 复制回原数组
    copy(arr, output)
}
```

### 复杂度
- **时间复杂度**：O(d * (n + k))（d 为位数，k 为基数）
- **空间复杂度**：O(n + k)
- **稳定性**：稳定

---

## 排序算法对比

| 算法 | 时间复杂度(平均) | 时间复杂度(最坏) | 空间复杂度 | 稳定性 |
|------|----------------|----------------|-----------|--------|
| 冒泡排序 | O(n²) | O(n²) | O(1) | 稳定 |
| 选择排序 | O(n²) | O(n²) | O(1) | 不稳定 |
| 插入排序 | O(n²) | O(n²) | O(1) | 稳定 |
| 希尔排序 | O(n log n) | O(n²) | O(1) | 不稳定 |
| 快速排序 | O(n log n) | O(n²) | O(log n) | 不稳定 |
| 归并排序 | O(n log n) | O(n log n) | O(n) | 稳定 |
| 堆排序 | O(n log n) | O(n log n) | O(1) | 不稳定 |
| 计数排序 | O(n + k) | O(n + k) | O(k) | 稳定 |
| 桶排序 | O(n + k) | O(n²) | O(n + k) | 稳定 |
| 基数排序 | O(d(n + k)) | O(d(n + k)) | O(n + k) | 稳定 |

## 常见面试题

### 1. 快速排序的时间复杂度为什么是 O(n log n)？

**答案：**
快速排序每次将数组分成两部分，递归深度为 log n，每层处理 n 个元素，因此总时间复杂度为 O(n log n)。最坏情况（数组已有序）下，每次划分只能减少一个元素，递归深度为 n，时间复杂度退化为 O(n²)。

### 2. 为什么快速排序通常比归并排序快？

**答案：**
虽然两者时间复杂度相同，但快速排序的常数因子更小：
- 快速排序是原地排序，不需要额外空间
- 快速排序的内存访问更局部化，缓存命中率更高
- 归并排序需要额外的合并步骤

### 3. 什么是稳定排序？

**答案：**
稳定排序是指相等元素在排序后的相对顺序与排序前相同。例如，排序 [(1, "a"), (2, "b"), (1, "c")] 时，稳定排序会保持两个 1 的顺序：[(1, "a"), (1, "c"), (2, "b")]。

### 4. 什么时候使用非比较排序？

**答案：**
当数据满足以下条件时，非比较排序（如计数排序、桶排序、基数排序）可以达到线性时间：
- 数据范围较小（适合计数排序）
- 数据分布均匀（适合桶排序）
- 数据是整数且位数有限（适合基数排序）

### 5. 插入排序在什么情况下效率最高？

**答案：**
插入排序在数组基本有序时效率最高，时间复杂度接近 O(n)。因为此时每个元素只需比较很少次数就能找到正确位置。

### 6. 堆排序和快速排序的区别？

**答案：**
- **堆排序**：利用堆数据结构，每次提取最大值，时间复杂度稳定 O(n log n)，不稳定
- **快速排序**：利用分治思想，选择基准元素划分，平均 O(n log n)，最坏 O(n²)，不稳定

### 7. 如何选择合适的排序算法？

**答案：**
- 小规模数据：插入排序（简单且常数因子小）
- 中等规模数据：快速排序（平均性能最好）
- 需要稳定排序：归并排序、计数排序
- 数据范围有限：计数排序、基数排序
- 内存受限：堆排序（原地排序）

### 8. 归并排序为什么是稳定的？

**答案：**
归并排序在合并两个有序数组时，当两个元素相等时，优先取左边数组的元素，这样可以保持相等元素的相对顺序。

### 9. 希尔排序的增量序列如何选择？

**答案：**
常用的增量序列有：
- 原始序列：n/2, n/4, ..., 1
- Hibbard 序列：2^k - 1
- Sedgewick 序列：4^k + 3*2^(k-1) + 1

增量序列的选择会影响希尔排序的性能。

### 10. 基数排序为什么要从低位到高位排序？

**答案：**
从低位到高位排序可以保证低位排序的结果不会被高位排序破坏。如果从高位到低位排序，低位排序会打乱已经排好的高位顺序。

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
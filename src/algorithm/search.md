# 查找算法

## 查找算法简介
### 什么是查找算法？
查找算法是在数据集合中寻找特定元素的算法。查找是计算机科学中最基本的操作之一，广泛应用于数据库、文件系统、网络搜索等领域。

### 查找算法分类
- **顺序查找**：逐个检查元素
- **二分查找**：利用有序性快速定位
- **哈希查找**：利用哈希函数快速定位
- **树查找**：利用树结构进行高效查找

### 性能指标
- **时间复杂度**：查找所需的比较次数
- **空间复杂度**：算法所需的额外空间
- **平均查找长度**：期望的比较次数

---

## 1. 顺序查找 (Sequential Search)

### 原理
从数组的一端开始，逐个检查每个元素，直到找到目标元素或遍历结束。

### 实现
```go
func SequentialSearch(arr []int, target int) int {
    for i, v := range arr {
        if v == target {
            return i
        }
    }
    return -1
}
```

### 复杂度
- **时间复杂度**：O(n)
- **空间复杂度**：O(1)
- **适用场景**：无序数组、小规模数据

---

## 2. 二分查找 (Binary Search)

### 原理
利用数组的有序性，每次将搜索范围缩小一半。

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
- **适用场景**：有序数组、大规模数据

---

## 3. 插值查找 (Interpolation Search)

### 原理
根据目标值估算位置，适用于均匀分布的有序数组。

### 实现
```go
func InterpolationSearch(arr []int, target int) int {
    left, right := 0, len(arr)-1
    
    for left <= right && target >= arr[left] && target <= arr[right] {
        pos := left + (target-arr[left])*(right-left)/(arr[right]-arr[left])
        
        if arr[pos] == target {
            return pos
        } else if arr[pos] < target {
            left = pos + 1
        } else {
            right = pos - 1
        }
    }
    
    return -1
}
```

### 复杂度
- **时间复杂度**：O(log log n)（平均）
- **空间复杂度**：O(1)
- **适用场景**：均匀分布的有序数组

---

## 4. 哈希查找 (Hash Search)

### 原理
利用哈希函数将关键字映射到数组索引，实现O(1)查找。

### 实现
```go
type HashTable struct {
    size int
    data []*HashNode
}

type HashNode struct {
    key   int
    value int
    next  *HashNode
}

func NewHashTable(size int) *HashTable {
    return &HashTable{size: size, data: make([]*HashNode, size)}
}

func (h *HashTable) hash(key int) int {
    return key % h.size
}

func (h *HashTable) Insert(key, value int) {
    index := h.hash(key)
    node := &HashNode{key: key, value: value}
    
    if h.data[index] == nil {
        h.data[index] = node
    } else {
        current := h.data[index]
        for current.next != nil {
            current = current.next
        }
        current.next = node
    }
}

func (h *HashTable) Search(key int) (int, bool) {
    index := h.hash(key)
    current := h.data[index]
    
    for current != nil {
        if current.key == key {
            return current.value, true
        }
        current = current.next
    }
    
    return 0, false
}
```

### 复杂度
- **时间复杂度**：O(1)（平均）
- **空间复杂度**：O(n)
- **适用场景**：需要频繁查找的场景

---

## 5. 二叉搜索树查找 (BST Search)

### 原理
利用二叉搜索树的性质：左子树 < 根 < 右子树。

### 实现
```go
type TreeNode struct {
    Val   int
    Left  *TreeNode
    Right *TreeNode
}

func BSTSearch(root *TreeNode, target int) *TreeNode {
    if root == nil || root.Val == target {
        return root
    }
    
    if target < root.Val {
        return BSTSearch(root.Left, target)
    }
    return BSTSearch(root.Right, target)
}
```

### 复杂度
- **时间复杂度**：O(log n)（平均）
- **空间复杂度**：O(log n)
- **适用场景**：动态数据集合

---

## 查找算法对比

| 算法 | 时间复杂度 | 空间复杂度 | 适用场景 |
|------|----------|-----------|----------|
| 顺序查找 | O(n) | O(1) | 无序、小规模 |
| 二分查找 | O(log n) | O(1) | 有序数组 |
| 插值查找 | O(log log n) | O(1) | 均匀分布有序数组 |
| 哈希查找 | O(1) | O(n) | 频繁查找 |
| BST查找 | O(log n) | O(log n) | 动态数据 |

## 常见面试题

### 1. 二分查找的时间复杂度为什么是 O(log n)？

**答案：**
每次比较将搜索范围缩小一半，需要 log₂(n) 次比较才能找到目标或确定不存在。

### 2. 为什么二分查找需要数组有序？

**答案：**
二分查找依赖于中间元素与目标的比较来决定搜索方向。如果数组无序，无法确定目标在左半部分还是右半部分。

### 3. 哈希查找为什么会有冲突？如何解决？

**答案：**
当不同的关键字通过哈希函数得到相同的索引时就会产生冲突。解决方法：链地址法、开放地址法、再哈希法。

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
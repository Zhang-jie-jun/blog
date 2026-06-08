# 哈希表

## 哈希表简介
### 什么是哈希表？
哈希表是一种通过哈希函数将键映射到数组索引的数据结构，实现O(1)时间复杂度的插入、删除和查找操作。

### 哈希表的特点
- **快速访问**：平均O(1)的查找、插入、删除
- **哈希函数**：将键转换为数组索引
- **冲突处理**：解决不同键映射到同一索引的问题

### 常见应用场景
- 数据库索引
- 缓存系统
- 关联数组
- 去重操作

---

## 1. 哈希表实现

### 链地址法
```go
type HashNode struct {
    key   int
    value int
    next  *HashNode
}

type HashTable struct {
    size  int
    slots []*HashNode
}

func NewHashTable(size int) *HashTable {
    return &HashTable{
        size:  size,
        slots: make([]*HashNode, size),
    }
}

func (h *HashTable) hash(key int) int {
    return key % h.size
}

func (h *HashTable) Insert(key, value int) {
    index := h.hash(key)
    newNode := &HashNode{key: key, value: value}
    
    if h.slots[index] == nil {
        h.slots[index] = newNode
    } else {
        current := h.slots[index]
        for current.next != nil {
            if current.key == key {
                current.value = value
                return
            }
            current = current.next
        }
        current.next = newNode
    }
}

func (h *HashTable) Search(key int) (int, bool) {
    index := h.hash(key)
    current := h.slots[index]
    
    for current != nil {
        if current.key == key {
            return current.value, true
        }
        current = current.next
    }
    
    return 0, false
}

func (h *HashTable) Delete(key int) bool {
    index := h.hash(key)
    current := h.slots[index]
    var prev *HashNode
    
    for current != nil {
        if current.key == key {
            if prev == nil {
                h.slots[index] = current.next
            } else {
                prev.next = current.next
            }
            return true
        }
        prev = current
        current = current.next
    }
    
    return false
}
```

### 开放地址法（线性探测）
```go
type OpenAddressHashTable struct {
    size     int
    keys     []int
    values   []int
    deleted  []bool
}

func NewOpenAddressHashTable(size int) *OpenAddressHashTable {
    return &OpenAddressHashTable{
        size:    size,
        keys:    make([]int, size),
        values:  make([]int, size),
        deleted: make([]bool, size),
    }
}

func (h *OpenAddressHashTable) hash(key int) int {
    return key % h.size
}

func (h *OpenAddressHashTable) Insert(key, value int) bool {
    index := h.hash(key)
    
    for i := 0; i < h.size; i++ {
        pos := (index + i) % h.size
        
        if h.keys[pos] == 0 && !h.deleted[pos] {
            h.keys[pos] = key
            h.values[pos] = value
            return true
        }
        
        if h.keys[pos] == key {
            h.values[pos] = value
            return true
        }
    }
    
    return false // 哈希表已满
}

func (h *OpenAddressHashTable) Search(key int) (int, bool) {
    index := h.hash(key)
    
    for i := 0; i < h.size; i++ {
        pos := (index + i) % h.size
        
        if h.keys[pos] == 0 && !h.deleted[pos] {
            return 0, false
        }
        
        if h.keys[pos] == key {
            return h.values[pos], true
        }
    }
    
    return 0, false
}

func (h *OpenAddressHashTable) Delete(key int) bool {
    index := h.hash(key)
    
    for i := 0; i < h.size; i++ {
        pos := (index + i) % h.size
        
        if h.keys[pos] == 0 && !h.deleted[pos] {
            return false
        }
        
        if h.keys[pos] == key {
            h.deleted[pos] = true
            return true
        }
    }
    
    return false
}
```

### 复杂度
- **时间复杂度**：O(1)（平均），O(n)（最坏）
- **空间复杂度**：O(n)

---

## 2. 两数之和

### 问题描述
给定一个整数数组和一个目标值，找出数组中和为目标值的两个数。

### 实现
```go
func TwoSum(nums []int, target int) []int {
    hashMap := make(map[int]int)
    
    for i, num := range nums {
        complement := target - num
        
        if j, ok := hashMap[complement]; ok {
            return []int{j, i}
        }
        
        hashMap[num] = i
    }
    
    return nil
}
```

### 复杂度
- **时间复杂度**：O(n)
- **空间复杂度**：O(n)

---

## 3. 无重复字符的最长子串

### 问题描述
给定一个字符串，找出不含有重复字符的最长子串的长度。

### 实现
```go
func LengthOfLongestSubstring(s string) int {
    charMap := make(map[byte]int)
    maxLen := 0
    left := 0
    
    for right := 0; right < len(s); right++ {
        if pos, ok := charMap[s[right]]; ok && pos >= left {
            left = pos + 1
        }
        
        charMap[s[right]] = right
        
        if right-left+1 > maxLen {
            maxLen = right - left + 1
        }
    }
    
    return maxLen
}
```

### 复杂度
- **时间复杂度**：O(n)
- **空间复杂度**：O(min(m, n))（m为字符集大小）

---

## 4. 字母异位词分组

### 问题描述
给定一个字符串数组，将字母异位词组合在一起。

### 实现
```go
import "sort"

func GroupAnagrams(strs []string) [][]string {
    groups := make(map[string][]string)
    
    for _, str := range strs {
        // 将字符串排序作为key
        chars := []byte(str)
        sort.Slice(chars, func(i, j int) bool {
            return chars[i] < chars[j]
        })
        key := string(chars)
        
        groups[key] = append(groups[key], str)
    }
    
    result := make([][]string, 0, len(groups))
    for _, group := range groups {
        result = append(result, group)
    }
    
    return result
}
```

### 复杂度
- **时间复杂度**：O(n * k log k)（k为字符串最大长度）
- **空间复杂度**：O(n * k)

---

## 5. LRU缓存

### 问题描述
实现一个LRU（最近最少使用）缓存机制。

### 实现
```go
type LRUCache struct {
    capacity int
    cache    map[int]*LRUNode
    head     *LRUNode
    tail     *LRUNode
}

type LRUNode struct {
    key   int
    value int
    prev  *LRUNode
    next  *LRUNode
}

func NewLRUCache(capacity int) *LRUCache {
    return &LRUCache{
        capacity: capacity,
        cache:    make(map[int]*LRUNode),
    }
}

func (c *LRUCache) Get(key int) int {
    node, ok := c.cache[key]
    if !ok {
        return -1
    }
    
    c.moveToHead(node)
    return node.value
}

func (c *LRUCache) Put(key, value int) {
    node, ok := c.cache[key]
    
    if ok {
        node.value = value
        c.moveToHead(node)
        return
    }
    
    newNode := &LRUNode{key: key, value: value}
    c.cache[key] = newNode
    c.addToHead(newNode)
    
    if len(c.cache) > c.capacity {
        tail := c.removeTail()
        delete(c.cache, tail.key)
    }
}

func (c *LRUCache) addToHead(node *LRUNode) {
    node.prev = nil
    node.next = c.head
    
    if c.head != nil {
        c.head.prev = node
    } else {
        c.tail = node
    }
    
    c.head = node
}

func (c *LRUCache) removeNode(node *LRUNode) {
    if node.prev != nil {
        node.prev.next = node.next
    } else {
        c.head = node.next
    }
    
    if node.next != nil {
        node.next.prev = node.prev
    } else {
        c.tail = node.prev
    }
}

func (c *LRUCache) moveToHead(node *LRUNode) {
    c.removeNode(node)
    c.addToHead(node)
}

func (c *LRUCache) removeTail() *LRUNode {
    tail := c.tail
    c.removeNode(tail)
    return tail
}
```

### 复杂度
- **时间复杂度**：O(1)（所有操作）
- **空间复杂度**：O(capacity)

---

## 哈希表算法对比

| 冲突解决方法 | 优点 | 缺点 |
|-------------|------|------|
| 链地址法 | 实现简单，适合频繁插入删除 | 需要额外空间存储链表 |
| 开放地址法 | 空间利用率高，缓存友好 | 容易产生聚集 |
| 再哈希法 | 减少聚集 | 计算量大 |

## 常见面试题

### 1. 哈希表的工作原理是什么？

**答案：**
哈希表通过哈希函数将键转换为数组索引，实现O(1)的访问时间。当发生冲突时，使用链地址法或开放地址法解决。

### 2. 常见的哈希函数有哪些？

**答案：**
- 直接寻址法
- 除留余数法
- 平方取中法
- 折叠法
- 随机数法

### 3. 哈希冲突的解决方法有哪些？

**答案：**
- **链地址法**：每个槽维护一个链表
- **开放地址法**：线性探测、二次探测、双重哈希
- **再哈希法**：使用多个哈希函数

### 4. 什么是负载因子？

**答案：**
负载因子 = 元素数量 / 哈希表大小。负载因子越大，冲突概率越高。通常负载因子达到0.7~0.8时需要扩容。

### 5. LRU和LFU的区别？

**答案：**
- **LRU**（最近最少使用）：淘汰最长时间未使用的元素
- **LFU**（最不经常使用）：淘汰使用次数最少的元素

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

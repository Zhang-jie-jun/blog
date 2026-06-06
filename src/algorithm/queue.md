# 队列

## 队列简介
### 什么是队列？
队列是一种遵循先进先出（FIFO）原则的线性数据结构。队列允许在一端插入（队尾），在另一端删除（队头）。

### 队列的特点
- **先进先出**（First In First Out, FIFO）
- 队尾插入，队头删除
- 操作时间复杂度为O(1)

### 常见应用场景
- 任务调度
- 消息队列
- BFS算法
- 缓存机制

---

## 1. 队列的实现

### 基于数组的实现
```go
type Queue struct {
    elements []int
    front    int
    rear     int
    size     int
    capacity int
}

func NewQueue(capacity int) *Queue {
    return &Queue{
        elements: make([]int, capacity),
        front:    0,
        rear:     0,
        size:     0,
        capacity: capacity,
    }
}

func (q *Queue) Enqueue(val int) bool {
    if q.IsFull() {
        return false
    }
    
    q.elements[q.rear] = val
    q.rear = (q.rear + 1) % q.capacity
    q.size++
    return true
}

func (q *Queue) Dequeue() (int, bool) {
    if q.IsEmpty() {
        return 0, false
    }
    
    val := q.elements[q.front]
    q.front = (q.front + 1) % q.capacity
    q.size--
    return val, true
}

func (q *Queue) Peek() (int, bool) {
    if q.IsEmpty() {
        return 0, false
    }
    return q.elements[q.front], true
}

func (q *Queue) IsEmpty() bool {
    return q.size == 0
}

func (q *Queue) IsFull() bool {
    return q.size == q.capacity
}

func (q *Queue) Size() int {
    return q.size
}
```

### 基于链表的实现
```go
type QueueNode struct {
    Val  int
    Next *QueueNode
}

type LinkedListQueue struct {
    front *QueueNode
    rear  *QueueNode
    size  int
}

func NewLinkedListQueue() *LinkedListQueue {
    return &LinkedListQueue{front: nil, rear: nil, size: 0}
}

func (q *LinkedListQueue) Enqueue(val int) {
    newNode := &QueueNode{Val: val, Next: nil}
    
    if q.IsEmpty() {
        q.front = newNode
        q.rear = newNode
    } else {
        q.rear.Next = newNode
        q.rear = newNode
    }
    
    q.size++
}

func (q *LinkedListQueue) Dequeue() (int, bool) {
    if q.IsEmpty() {
        return 0, false
    }
    
    val := q.front.Val
    q.front = q.front.Next
    
    if q.front == nil {
        q.rear = nil
    }
    
    q.size--
    return val, true
}

func (q *LinkedListQueue) Peek() (int, bool) {
    if q.IsEmpty() {
        return 0, false
    }
    return q.front.Val, true
}

func (q *LinkedListQueue) IsEmpty() bool {
    return q.front == nil
}

func (q *LinkedListQueue) Size() int {
    return q.size
}
```

### 复杂度
- **时间复杂度**：O(1)（所有操作）
- **空间复杂度**：O(n)

---

## 2. 循环队列

### 原理
使用数组实现的队列，通过取模运算实现循环。

### 实现
```go
type CircularQueue struct {
    elements []int
    front    int
    rear     int
    size     int
}

func NewCircularQueue(k int) *CircularQueue {
    return &CircularQueue{
        elements: make([]int, k),
        front:    0,
        rear:     0,
        size:     0,
    }
}

func (q *CircularQueue) EnQueue(value int) bool {
    if q.IsFull() {
        return false
    }
    
    q.elements[q.rear] = value
    q.rear = (q.rear + 1) % len(q.elements)
    q.size++
    return true
}

func (q *CircularQueue) DeQueue() bool {
    if q.IsEmpty() {
        return false
    }
    
    q.front = (q.front + 1) % len(q.elements)
    q.size--
    return true
}

func (q *CircularQueue) Front() int {
    if q.IsEmpty() {
        return -1
    }
    return q.elements[q.front]
}

func (q *CircularQueue) Rear() int {
    if q.IsEmpty() {
        return -1
    }
    return q.elements[(q.rear-1+len(q.elements))%len(q.elements)]
}

func (q *CircularQueue) IsEmpty() bool {
    return q.size == 0
}

func (q *CircularQueue) IsFull() bool {
    return q.size == len(q.elements)
}
```

---

## 3. 优先队列

### 原理
优先队列中的元素按照优先级顺序出队，而不是插入顺序。

### 实现（基于堆）
```go
import "container/heap"

type PriorityQueue []*Item

type Item struct {
    value    string
    priority int
    index    int
}

func (pq PriorityQueue) Len() int { return len(pq) }

func (pq PriorityQueue) Less(i, j int) bool {
    return pq[i].priority > pq[j].priority
}

func (pq PriorityQueue) Swap(i, j int) {
    pq[i], pq[j] = pq[j], pq[i]
    pq[i].index = i
    pq[j].index = j
}

func (pq *PriorityQueue) Push(x interface{}) {
    n := len(*pq)
    item := x.(*Item)
    item.index = n
    *pq = append(*pq, item)
}

func (pq *PriorityQueue) Pop() interface{} {
    old := *pq
    n := len(old)
    item := old[n-1]
    item.index = -1
    *pq = old[0 : n-1]
    return item
}

func (pq *PriorityQueue) Update(item *Item, value string, priority int) {
    item.value = value
    item.priority = priority
    heap.Fix(pq, item.index)
}
```

---

## 4. 滑动窗口最大值

### 问题描述
给定一个数组和滑动窗口大小，找出所有滑动窗口中的最大值。

### 实现（使用双端队列）
```go
func MaxSlidingWindow(nums []int, k int) []int {
    result := []int{}
    deque := []int{}
    
    for i, num := range nums {
        // 移除超出窗口范围的元素
        for len(deque) > 0 && deque[0] < i-k+1 {
            deque = deque[1:]
        }
        
        // 移除比当前元素小的元素
        for len(deque) > 0 && nums[deque[len(deque)-1]] <= num {
            deque = deque[:len(deque)-1]
        }
        
        deque = append(deque, i)
        
        // 当窗口形成时，记录最大值
        if i >= k-1 {
            result = append(result, nums[deque[0]])
        }
    }
    
    return result
}
```

### 复杂度
- **时间复杂度**：O(n)
- **空间复杂度**：O(k)

---

## 5. 用队列实现栈

### 实现
```go
type MyStack struct {
    queue1 *LinkedListQueue
    queue2 *LinkedListQueue
}

func NewMyStack() *MyStack {
    return &MyStack{
        queue1: NewLinkedListQueue(),
        queue2: NewLinkedListQueue(),
    }
}

func (s *MyStack) Push(x int) {
    s.queue2.Enqueue(x)
    
    for !s.queue1.IsEmpty() {
        val, _ := s.queue1.Dequeue()
        s.queue2.Enqueue(val)
    }
    
    s.queue1, s.queue2 = s.queue2, s.queue1
}

func (s *MyStack) Pop() int {
    val, _ := s.queue1.Dequeue()
    return val
}

func (s *MyStack) Top() int {
    val, _ := s.queue1.Peek()
    return val
}

func (s *MyStack) Empty() bool {
    return s.queue1.IsEmpty()
}
```

### 复杂度
- **时间复杂度**：O(n)（Push），O(1)（Pop、Top）
- **空间复杂度**：O(n)

---

## 队列算法对比

| 队列类型 | Enqueue | Dequeue | 特点 |
|---------|---------|---------|------|
| 普通队列 | O(1) | O(1) | 基本FIFO |
| 循环队列 | O(1) | O(1) | 空间利用率高 |
| 优先队列 | O(log n) | O(log n) | 按优先级出队 |
| 双端队列 | O(1) | O(1) | 两端都可操作 |

## 常见面试题

### 1. 队列和栈的区别？

**答案：**
- **队列**：先进先出（FIFO），队尾插入，队头删除
- **栈**：后进先出（LIFO），栈顶插入和删除

### 2. 如何用两个队列实现一个栈？

**答案：**
使用两个队列，push时将元素加入queue2，然后将queue1的所有元素移到queue2，交换两个队列的角色。

### 3. 什么是双端队列？

**答案：**
双端队列（Deque）允许在两端进行插入和删除操作，可以作为栈或队列使用。

### 4. 滑动窗口最大值问题为什么使用双端队列？

**答案：**
双端队列可以维护窗口内的候选最大值，保持队列中的元素索引对应的数值递减，从而在O(1)时间内获取窗口最大值。

### 5. 优先队列的应用场景有哪些？

**答案：**
- 任务调度（按优先级执行）
- Dijkstra算法（找最短路径）
- Huffman编码（构建最优二叉树）

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
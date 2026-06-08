# 链表

## 链表简介
### 什么是链表？
链表是一种线性数据结构，元素通过指针链接在一起。每个节点包含数据和指向下一个节点的指针。

### 链表的特点
- **非连续存储**：元素在内存中不一定连续
- **动态大小**：可以灵活地添加或删除元素
- **顺序访问**：只能从头节点开始顺序访问

### 常见链表类型
- **单链表**：每个节点只有一个指向下一个节点的指针
- **双链表**：每个节点有两个指针，分别指向前一个和后一个节点
- **循环链表**：最后一个节点指向第一个节点

### 常见应用场景
- 实现栈和队列
- 动态数组
- 哈希表的冲突解决
- 图的邻接表表示

---

## 1. 单链表实现

### 节点结构
```go
type ListNode struct {
    Val  int
    Next *ListNode
}

type LinkedList struct {
    head *ListNode
    size int
}

func NewLinkedList() *LinkedList {
    return &LinkedList{head: nil, size: 0}
}
```

### 基本操作
```go
func (l *LinkedList) AddFirst(val int) {
    newNode := &ListNode{Val: val, Next: l.head}
    l.head = newNode
    l.size++
}

func (l *LinkedList) AddLast(val int) {
    newNode := &ListNode{Val: val, Next: nil}
    
    if l.head == nil {
        l.head = newNode
    } else {
        current := l.head
        for current.Next != nil {
            current = current.Next
        }
        current.Next = newNode
    }
    
    l.size++
}

func (l *LinkedList) Insert(index, val int) bool {
    if index < 0 || index > l.size {
        return false
    }
    
    if index == 0 {
        l.AddFirst(val)
        return true
    }
    
    newNode := &ListNode{Val: val}
    current := l.head
    
    for i := 0; i < index-1; i++ {
        current = current.Next
    }
    
    newNode.Next = current.Next
    current.Next = newNode
    l.size++
    
    return true
}

func (l *LinkedList) RemoveFirst() (int, bool) {
    if l.head == nil {
        return 0, false
    }
    
    val := l.head.Val
    l.head = l.head.Next
    l.size--
    
    return val, true
}

func (l *LinkedList) RemoveLast() (int, bool) {
    if l.head == nil {
        return 0, false
    }
    
    if l.head.Next == nil {
        return l.RemoveFirst()
    }
    
    current := l.head
    for current.Next.Next != nil {
        current = current.Next
    }
    
    val := current.Next.Val
    current.Next = nil
    l.size--
    
    return val, true
}

func (l *LinkedList) Get(index int) (int, bool) {
    if index < 0 || index >= l.size {
        return 0, false
    }
    
    current := l.head
    for i := 0; i < index; i++ {
        current = current.Next
    }
    
    return current.Val, true
}

func (l *LinkedList) Size() int {
    return l.size
}

func (l *LinkedList) IsEmpty() bool {
    return l.head == nil
}
```

---

## 2. 反转链表

### 问题描述
反转一个单链表。

### 实现（迭代）
```go
func ReverseList(head *ListNode) *ListNode {
    var prev *ListNode
    current := head
    
    for current != nil {
        next := current.Next
        current.Next = prev
        prev = current
        current = next
    }
    
    return prev
}
```

### 实现（递归）
```go
func ReverseListRecursive(head *ListNode) *ListNode {
    if head == nil || head.Next == nil {
        return head
    }
    
    newHead := ReverseListRecursive(head.Next)
    head.Next.Next = head
    head.Next = nil
    
    return newHead
}
```

### 复杂度
- **时间复杂度**：O(n)
- **空间复杂度**：O(1)（迭代），O(n)（递归）

---

## 3. 合并两个有序链表

### 问题描述
将两个升序链表合并为一个新的升序链表。

### 实现
```go
func MergeTwoLists(l1, l2 *ListNode) *ListNode {
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
- **时间复杂度**：O(n + m)
- **空间复杂度**：O(1)

---

## 4. 链表中倒数第k个节点

### 问题描述
找出链表中倒数第k个节点。

### 实现（双指针）
```go
func GetKthFromEnd(head *ListNode, k int) *ListNode {
    fast := head
    slow := head
    
    // 快指针先走k步
    for i := 0; i < k; i++ {
        if fast == nil {
            return nil
        }
        fast = fast.Next
    }
    
    // 快慢指针一起走
    for fast != nil {
        fast = fast.Next
        slow = slow.Next
    }
    
    return slow
}
```

### 复杂度
- **时间复杂度**：O(n)
- **空间复杂度**：O(1)

---

## 5. 环形链表

### 问题描述
判断链表中是否有环。

### 实现（快慢指针）
```go
func HasCycle(head *ListNode) bool {
    if head == nil || head.Next == nil {
        return false
    }
    
    slow := head
    fast := head.Next
    
    for slow != fast {
        if fast == nil || fast.Next == nil {
            return false
        }
        slow = slow.Next
        fast = fast.Next.Next
    }
    
    return true
}
```

### 复杂度
- **时间复杂度**：O(n)
- **空间复杂度**：O(1)

---

## 6. 相交链表

### 问题描述
找到两个单链表相交的起始节点。

### 实现
```go
func GetIntersectionNode(headA, headB *ListNode) *ListNode {
    if headA == nil || headB == nil {
        return nil
    }
    
    a, b := headA, headB
    
    for a != b {
        if a == nil {
            a = headB
        } else {
            a = a.Next
        }
        
        if b == nil {
            b = headA
        } else {
            b = b.Next
        }
    }
    
    return a
}
```

### 复杂度
- **时间复杂度**：O(n + m)
- **空间复杂度**：O(1)

---

## 7. 删除链表的倒数第N个节点

### 问题描述
删除链表的倒数第N个节点。

### 实现
```go
func RemoveNthFromEnd(head *ListNode, n int) *ListNode {
    dummy := &ListNode{Next: head}
    fast := dummy
    slow := dummy
    
    // 快指针先走n+1步
    for i := 0; i <= n; i++ {
        fast = fast.Next
    }
    
    // 快慢指针一起走
    for fast != nil {
        fast = fast.Next
        slow = slow.Next
    }
    
    // 删除节点
    slow.Next = slow.Next.Next
    
    return dummy.Next
}
```

### 复杂度
- **时间复杂度**：O(n)
- **空间复杂度**：O(1)

---

## 8. 双链表实现

### 节点结构
```go
type DoublyListNode struct {
    Val  int
    Prev *DoublyListNode
    Next *DoublyListNode
}

type DoublyLinkedList struct {
    head *DoublyListNode
    tail *DoublyListNode
    size int
}

func NewDoublyLinkedList() *DoublyLinkedList {
    return &DoublyLinkedList{head: nil, tail: nil, size: 0}
}
```

### 基本操作
```go
func (d *DoublyLinkedList) AddFirst(val int) {
    newNode := &DoublyListNode{Val: val, Prev: nil, Next: d.head}
    
    if d.head == nil {
        d.tail = newNode
    } else {
        d.head.Prev = newNode
    }
    
    d.head = newNode
    d.size++
}

func (d *DoublyLinkedList) AddLast(val int) {
    newNode := &DoublyListNode{Val: val, Prev: d.tail, Next: nil}
    
    if d.tail == nil {
        d.head = newNode
    } else {
        d.tail.Next = newNode
    }
    
    d.tail = newNode
    d.size++
}

func (d *DoublyLinkedList) RemoveFirst() (int, bool) {
    if d.head == nil {
        return 0, false
    }
    
    val := d.head.Val
    d.head = d.head.Next
    
    if d.head == nil {
        d.tail = nil
    } else {
        d.head.Prev = nil
    }
    
    d.size--
    return val, true
}

func (d *DoublyLinkedList) RemoveLast() (int, bool) {
    if d.tail == nil {
        return 0, false
    }
    
    val := d.tail.Val
    d.tail = d.tail.Prev
    
    if d.tail == nil {
        d.head = nil
    } else {
        d.tail.Next = nil
    }
    
    d.size--
    return val, true
}
```

---

## 链表算法对比

| 操作 | 单链表 | 双链表 |
|------|-------|-------|
| 头部插入 | O(1) | O(1) |
| 尾部插入 | O(n) | O(1) |
| 头部删除 | O(1) | O(1) |
| 尾部删除 | O(n) | O(1) |
| 访问第k个元素 | O(k) | O(min(k, n-k)) |

## 常见面试题

### 1. 单链表和双链表的区别？

**答案：**
- **单链表**：每个节点只有一个Next指针，只能单向遍历
- **双链表**：每个节点有Prev和Next指针，可以双向遍历，尾部操作更高效

### 2. 如何判断链表是否有环？

**答案：**
使用快慢指针法，快指针每次走两步，慢指针每次走一步。如果相遇则有环。

### 3. 如何找到环形链表的入口节点？

**答案：**
先通过快慢指针找到相遇点，然后将一个指针移到链表头部，两个指针以相同速度移动，相遇点即为入口。

### 4. 如何合并k个有序链表？

**答案：**
方法一：分治合并，两两合并链表。
方法二：使用优先队列，每次取最小的节点。

### 5. 链表和数组的区别？

**答案：**

| 特性 | 数组 | 链表 |
|------|------|------|
| 存储 | 连续 | 非连续 |
| 随机访问 | O(1) | O(n) |
| 插入删除 | O(n) | O(1) |
| 空间 | 固定 | 动态 |

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

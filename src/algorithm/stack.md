# 栈

## 栈简介
### 什么是栈？
栈是一种遵循后进先出（LIFO）原则的线性数据结构。栈只允许在一端进行插入和删除操作，这一端称为栈顶。

### 栈的特点
- **后进先出**（Last In First Out, LIFO）
- 只允许在栈顶进行操作
- 操作时间复杂度为O(1)

### 常见应用场景
- 函数调用栈
- 表达式求值
- 括号匹配
- 浏览器历史记录

---

## 1. 栈的实现

### 基于数组的实现
```go
type Stack struct {
    elements []int
}

func NewStack() *Stack {
    return &Stack{elements: []int{}}
}

func (s *Stack) Push(val int) {
    s.elements = append(s.elements, val)
}

func (s *Stack) Pop() (int, bool) {
    if s.IsEmpty() {
        return 0, false
    }
    
    top := s.elements[len(s.elements)-1]
    s.elements = s.elements[:len(s.elements)-1]
    return top, true
}

func (s *Stack) Peek() (int, bool) {
    if s.IsEmpty() {
        return 0, false
    }
    return s.elements[len(s.elements)-1], true
}

func (s *Stack) IsEmpty() bool {
    return len(s.elements) == 0
}

func (s *Stack) Size() int {
    return len(s.elements)
}
```

### 基于链表的实现
```go
type StackNode struct {
    Val  int
    Next *StackNode
}

type LinkedListStack struct {
    top *StackNode
    size int
}

func NewLinkedListStack() *LinkedListStack {
    return &LinkedListStack{top: nil, size: 0}
}

func (s *LinkedListStack) Push(val int) {
    newNode := &StackNode{Val: val, Next: s.top}
    s.top = newNode
    s.size++
}

func (s *LinkedListStack) Pop() (int, bool) {
    if s.IsEmpty() {
        return 0, false
    }
    
    top := s.top
    s.top = s.top.Next
    s.size--
    
    return top.Val, true
}

func (s *LinkedListStack) Peek() (int, bool) {
    if s.IsEmpty() {
        return 0, false
    }
    return s.top.Val, true
}

func (s *LinkedListStack) IsEmpty() bool {
    return s.top == nil
}

func (s *LinkedListStack) Size() int {
    return s.size
}
```

### 复杂度
- **时间复杂度**：O(1)（所有操作）
- **空间复杂度**：O(n)

---

## 2. 有效的括号

### 问题描述
给定一个只包含 '()'、'{}'、'[]' 的字符串，判断字符串是否有效。

### 实现
```go
func IsValid(s string) bool {
    stack := NewStack()
    
    for _, char := range s {
        if char == '(' || char == '{' || char == '[' {
            stack.Push(int(char))
        } else {
            if stack.IsEmpty() {
                return false
            }
            
            top, _ := stack.Pop()
            if (char == ')' && top != int('(')) ||
               (char == '}' && top != int('{')) ||
               (char == ']' && top != int('[')) {
                return false
            }
        }
    }
    
    return stack.IsEmpty()
}
```

### 复杂度
- **时间复杂度**：O(n)
- **空间复杂度**：O(n)

---

## 3. 最小栈

### 问题描述
设计一个支持 push、pop、top 操作，并能在常数时间内检索到最小元素的栈。

### 实现
```go
type MinStack struct {
    stack    []int
    minStack []int
}

func NewMinStack() *MinStack {
    return &MinStack{
        stack:    []int{},
        minStack: []int{},
    }
}

func (m *MinStack) Push(val int) {
    m.stack = append(m.stack, val)
    
    if len(m.minStack) == 0 || val <= m.minStack[len(m.minStack)-1] {
        m.minStack = append(m.minStack, val)
    }
}

func (m *MinStack) Pop() {
    if len(m.stack) == 0 {
        return
    }
    
    top := m.stack[len(m.stack)-1]
    m.stack = m.stack[:len(m.stack)-1]
    
    if top == m.minStack[len(m.minStack)-1] {
        m.minStack = m.minStack[:len(m.minStack)-1]
    }
}

func (m *MinStack) Top() int {
    return m.stack[len(m.stack)-1]
}

func (m *MinStack) GetMin() int {
    return m.minStack[len(m.minStack)-1]
}
```

### 复杂度
- **时间复杂度**：O(1)（所有操作）
- **空间复杂度**：O(n)

---

## 4. 逆波兰表达式求值

### 问题描述
根据逆波兰表示法，求表达式的值。

### 实现
```go
import "strconv"

func EvalRPN(tokens []string) int {
    stack := NewStack()
    
    for _, token := range tokens {
        if token == "+" || token == "-" || token == "*" || token == "/" {
            b, _ := stack.Pop()
            a, _ := stack.Pop()
            
            var result int
            switch token {
            case "+":
                result = a + b
            case "-":
                result = a - b
            case "*":
                result = a * b
            case "/":
                result = a / b
            }
            
            stack.Push(result)
        } else {
            num, _ := strconv.Atoi(token)
            stack.Push(num)
        }
    }
    
    result, _ := stack.Pop()
    return result
}
```

### 复杂度
- **时间复杂度**：O(n)
- **空间复杂度**：O(n)

---

## 5. 用栈实现队列

### 实现
```go
type MyQueue struct {
    inStack  *Stack
    outStack *Stack
}

func NewMyQueue() *MyQueue {
    return &MyQueue{
        inStack:  NewStack(),
        outStack: NewStack(),
    }
}

func (q *MyQueue) Push(x int) {
    q.inStack.Push(x)
}

func (q *MyQueue) Pop() int {
    q.move()
    val, _ := q.outStack.Pop()
    return val
}

func (q *MyQueue) Peek() int {
    q.move()
    val, _ := q.outStack.Peek()
    return val
}

func (q *MyQueue) Empty() bool {
    return q.inStack.IsEmpty() && q.outStack.IsEmpty()
}

func (q *MyQueue) move() {
    if q.outStack.IsEmpty() {
        for !q.inStack.IsEmpty() {
            val, _ := q.inStack.Pop()
            q.outStack.Push(val)
        }
    }
}
```

### 复杂度
- **时间复杂度**：O(1)（均摊）
- **空间复杂度**：O(n)

---

## 栈算法对比

| 操作 | 数组实现 | 链表实现 |
|------|---------|---------|
| Push | O(1) | O(1) |
| Pop | O(1) | O(1) |
| Peek | O(1) | O(1) |
| IsEmpty | O(1) | O(1) |

## 常见面试题

### 1. 栈和队列的区别？

**答案：**
- **栈**：后进先出（LIFO），只允许在一端操作
- **队列**：先进先出（FIFO），允许在两端操作

### 2. 如何用两个栈实现一个队列？

**答案：**
使用两个栈，一个用于入队，一个用于出队。出队时如果出队栈为空，将入队栈的所有元素弹出并压入出队栈。

### 3. 什么是单调栈？

**答案：**
单调栈是一种特殊的栈，栈内元素保持单调递增或单调递减。常用于解决 Next Greater Element 等问题。

### 4. 如何找到数组中每个元素右边第一个比它大的元素？

**答案：**
使用单调栈：

```go
func NextGreaterElement(nums []int) []int {
    result := make([]int, len(nums))
    stack := []int{}
    
    for i := len(nums) - 1; i >= 0; i-- {
        for len(stack) > 0 && stack[len(stack)-1] <= nums[i] {
            stack = stack[:len(stack)-1]
        }
        
        if len(stack) == 0 {
            result[i] = -1
        } else {
            result[i] = stack[len(stack)-1]
        }
        
        stack = append(stack, nums[i])
    }
    
    return result
}
```

### 5. 如何实现浏览器的前进后退功能？

**答案：**
使用两个栈，一个保存前进历史，一个保存后退历史。

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

# context详解

## context简介
### context是什么？
context 是 Go 语言中用于在 goroutine 之间传递请求范围的数据、取消信号和超时时间的机制。它提供了一种优雅的方式来管理多个 goroutine 的生命周期。

### context的作用
- **取消信号传递**：当一个请求被取消或超时，所有相关的 goroutine 应该优雅地退出
- **请求范围数据传递**：在请求处理链中传递请求 ID、认证信息等
- **超时控制**：限制操作的执行时间

### context接口定义
```go
type Context interface {
    Deadline() (deadline time.Time, ok bool)
    Done() <-chan struct{}
    Err() error
    Value(key interface{}) interface{}
}
```

## context的实现原理
### 基础context类型
#### emptyCtx
emptyCtx 是一个空的 context，没有超时、取消和值传递功能：

```go
type emptyCtx int

func (*emptyCtx) Deadline() (deadline time.Time, ok bool) {
    return
}

func (*emptyCtx) Done() <-chan struct{} {
    return nil
}

func (*emptyCtx) Err() error {
    return nil
}

func (*emptyCtx) Value(key interface{}) interface{} {
    return nil
}
```

#### Background和TODO
```go
var (
    background = new(emptyCtx)
    todo       = new(emptyCtx)
)

func Background() Context {
    return background
}

func TODO() Context {
    return todo
}
```

### 派生context类型
#### cancelCtx
支持取消功能的 context：

```go
type cancelCtx struct {
    Context
    mu       sync.Mutex
    done     chan struct{}
    children map[canceler]struct{}
    err      error
}

type canceler interface {
    cancel(removeFromParent bool, err error)
    Done() <-chan struct{}
}
```

#### timerCtx
支持超时功能的 context：

```go
type timerCtx struct {
    cancelCtx
    timer *time.Timer
    deadline time.Time
}
```

#### valueCtx
支持值传递的 context：

```go
type valueCtx struct {
    Context
    key, val interface{}
}
```

### context创建过程
#### WithCancel
```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc) {
    c := newCancelCtx(parent)
    propagateCancel(parent, &c)
    return &c, func() { c.cancel(true, Canceled) }
}
```

#### WithDeadline
```go
func WithDeadline(parent Context, d time.Time) (Context, CancelFunc) {
    if cur, ok := parent.Deadline(); ok && cur.Before(d) {
        return WithCancel(parent)
    }
    c := &timerCtx{
        cancelCtx: newCancelCtx(parent),
        deadline:  d,
    }
    propagateCancel(parent, c)
    dur := time.Until(d)
    if dur <= 0 {
        c.cancel(true, DeadlineExceeded)
        return c, func() { c.cancel(false, Canceled) }
    }
    c.mu.Lock()
    defer c.mu.Unlock()
    if c.err == nil {
        c.timer = time.AfterFunc(dur, func() {
            c.cancel(true, DeadlineExceeded)
        })
    }
    return c, func() { c.cancel(true, Canceled) }
}
```

#### WithTimeout
```go
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc) {
    return WithDeadline(parent, time.Now().Add(timeout))
}
```

#### WithValue
```go
func WithValue(parent Context, key, val interface{}) Context {
    if key == nil {
        panic("nil key")
    }
    if !reflectlite.TypeOf(key).Comparable() {
        panic("key is not comparable")
    }
    return &valueCtx{parent, key, val}
}
```

### 取消机制
```go
func (c *cancelCtx) cancel(removeFromParent bool, err error) {
    c.mu.Lock()
    if c.err != nil {
        c.mu.Unlock()
        return
    }
    c.err = err
    close(c.done)
    
    // 取消所有子context
    for child := range c.children {
        child.cancel(false, err)
    }
    c.children = nil
    c.mu.Unlock()
    
    if removeFromParent {
        removeChild(c.Context, c)
    }
}
```

## context使用模式
### 基础使用示例
```go
func main() {
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()
    
    go func(ctx context.Context) {
        for {
            select {
            case <-ctx.Done():
                fmt.Println("goroutine canceled")
                return
            default:
                fmt.Println("working...")
                time.Sleep(time.Second)
            }
        }
    }(ctx)
    
    time.Sleep(3 * time.Second)
    cancel()
    time.Sleep(time.Second)
}
```

### 超时控制示例
```go
func fetchData(ctx context.Context) (string, error) {
    ctx, cancel := context.WithTimeout(ctx, 2*time.Second)
    defer cancel()
    
    ch := make(chan string)
    go func() {
        time.Sleep(3 * time.Second)
        ch <- "data"
    }()
    
    select {
    case data := <-ch:
        return data, nil
    case <-ctx.Done():
        return "", ctx.Err()
    }
}
```

### 值传递示例
```go
func handler(ctx context.Context) {
    if reqID, ok := ctx.Value("requestID").(string); ok {
        fmt.Printf("Request ID: %s\n", reqID)
    }
}

func main() {
    ctx := context.WithValue(context.Background(), "requestID", "12345")
    handler(ctx)
}
```

## 常见面试题
### 1. context的作用是什么？

**答案：**
context 主要用于：
- 在 goroutine 之间传递取消信号和超时信息
- 传递请求范围的数据（如请求ID、认证信息）
- 管理多个 goroutine 的生命周期

### 2. Background 和 TODO 的区别？

**答案：**
- `context.Background()`：用于 main 函数、初始化和测试，作为根 context
- `context.TODO()`：当不确定使用哪个 context 时使用，通常是临时的占位符

### 3. 为什么不能传递 nil context？

**答案：**
传递 nil context 会导致无法正确处理取消信号和超时，可能造成 goroutine 泄漏。Go 官方建议始终传递非 nil 的 context。

### 4. WithCancel、WithDeadline、WithTimeout 的区别？

**答案：**
- `WithCancel`：创建可手动取消的 context
- `WithDeadline`：创建在指定时间点自动取消的 context
- `WithTimeout`：创建在指定时长后自动取消的 context（是 WithDeadline 的便捷封装）

### 5. context 的值传递是如何实现的？

**答案：**
通过 valueCtx 链表实现，每个 valueCtx 包含一个 key-value 对和指向父 context 的指针。查找时向上遍历链表直到找到对应的 key 或到达链尾。

```go
func (c *valueCtx) Value(key interface{}) interface{} {
    if c.key == key {
        return c.val
    }
    return c.Context.Value(key)
}
```

### 6. 取消 context 后会发生什么？

**答案：**
- context 的 Done channel 会被关闭
- 所有子 context 也会被递归取消
- 通过 Err() 方法可以获取取消原因

### 7. context 是线程安全的吗？

**答案：**
是的，context 的实现使用了互斥锁来保证并发安全。

### 8. 如何正确传递 context？

**答案：**
- 作为函数的第一个参数传递
- 参数名统一为 ctx
- 不要存储 context，只在函数间传递

```go
func doSomething(ctx context.Context, arg string) error {
    // 使用 ctx 进行取消判断
    select {
    case <-ctx.Done():
        return ctx.Err()
    default:
    }
    // ...
}
```

### 9. context 可以传递哪些类型的值？

**答案：**
key 必须是可比较的类型（支持 == 和 !=），value 可以是任意类型。通常用于传递请求范围的数据，如：
- 请求 ID
- 用户认证信息
- 请求截止时间

### 10. 如果父 context 被取消，子 context 会怎样？

**答案：**
会被自动取消。context 形成树形结构，父 context 取消时会递归取消所有子 context。

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

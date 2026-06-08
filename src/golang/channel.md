# channel底层原理

## channel简介
### channel是什么？
channel 是 Go 语言中用于 goroutine 之间通信的一种机制，可以看作是 goroutine 之间的管道。它提供了一种安全的方式，让一个 goroutine 向另一个 goroutine 发送数据，实现并发同步和数据传递。

在 Go 语言中，channel 是一种类型，使用 `chan` 关键字定义，格式为 `chan T`，其中 `T` 是 channel 中传递的数据类型。

### channel声明
```go
var ch chan int // 声明一个传递int类型的channel，未分配空间
```

### channel初始化
```go
ch1 := make(chan int)         // 创建一个无缓冲channel
ch2 := make(chan int, 10)     // 创建一个容量为10的有缓冲channel
ch3 := make(chan<- int, 5)    // 创建一个只能发送的channel
ch4 := make(<-chan int, 5)    // 创建一个只能接收的channel
```

`make` 函数会调用 `runtime.makechan()` 来初始化 channel。

_源码位于go/src/runtime/chan.go中_

## channel的实现原理
Go 语言的 channel 底层是通过 `hchan` 结构体实现的，它包含了队列、锁、发送接收等待队列等核心组件。

### channel数据结构
`hchan` 是 channel 的核心数据结构：

```go
type hchan struct {
    qcount   uint           // 队列中当前元素个数
    dataqsiz uint           // 环形队列的大小（缓冲区容量）
    buf      unsafe.Pointer // 指向环形缓冲区的指针
    elemsize uint16         // 每个元素的大小
    closed   uint32         // channel是否关闭的标志
    elemtype *_type         // 元素类型信息
    sendx    uint           // 发送操作的索引位置
    recvx    uint           // 接收操作的索引位置
    recvq    waitq          // 等待接收的goroutine队列
    sendq    waitq          // 等待发送的goroutine队列

    lock mutex              // 互斥锁，保护channel的并发访问
}
```

`waitq` 是等待队列的结构：

```go
type waitq struct {
    first *sudog
    last  *sudog
}
```

`sudog` 是对 goroutine 的封装，用于表示等待在 channel 上的 goroutine。

### channel内存模型
channel 的内存布局如下：

```
┌─────────────────────────────────────────────────────────────┐
│                        hchan                               │
├─────────────┬─────────────┬────────────────────────────────┤
│  qcount     │  dataqsiz   │  buf (指向环形缓冲区)          │
├─────────────┼─────────────┼────────────────────────────────┤
│  elemsize   │  closed     │  elemtype                      │
├─────────────┼─────────────┼────────────────────────────────┤
│    sendx    │    recvx    │                                │
├─────────────┴─────────────┴────────────────────────────────┤
│                    recvq (接收等待队列)                    │
├─────────────────────────────────────────────────────────────┤
│                    sendq (发送等待队列)                    │
├─────────────────────────────────────────────────────────────┤
│                       lock                                 │
└─────────────────────────────────────────────────────────────┘
```

环形缓冲区的工作方式：
- `sendx` 表示下一个发送位置
- `recvx` 表示下一个接收位置
- 当 `sendx == recvx` 且 `qcount == 0` 时，缓冲区为空
- 当 `sendx == recvx` 且 `qcount == dataqsiz` 时，缓冲区满

### channel创建过程
`makechan` 函数负责创建 channel：

```go
func makechan(t *chantype, size int) *hchan {
    elem := t.elem

    // 检查元素类型是否合法
    if elem.size >= 1<<16 {
        throw("makechan: invalid channel element type")
    }
    // 计算所需内存大小
    mem, overflow := math.MulUintptr(elem.size, uintptr(size))
    if overflow || mem > maxAlloc-hchanSize || size < 0 {
        panic(plainError("makechan: size out of range"))
    }

    var c *hchan
    switch {
    case mem == 0:
        // 无缓冲channel或size为0，不需要缓冲区
        c = (*hchan)(mallocgc(hchanSize, nil, true))
        c.buf = c.raceaddr()
    case elem.ptrdata == 0:
        // 元素不包含指针，分配一块连续内存
        c = (*hchan)(mallocgc(hchanSize+mem, nil, true))
        c.buf = add(unsafe.Pointer(c), hchanSize)
    default:
        // 元素包含指针，分别分配hchan和缓冲区
        c = new(hchan)
        c.buf = mallocgc(mem, elem, true)
    }

    c.elemsize = uint16(elem.size)
    c.elemtype = elem
    c.dataqsiz = uint(size)

    return c
}
```

创建 channel 时会根据是否需要缓冲区、元素是否包含指针来选择不同的内存分配策略。

## channel操作原理
### 发送操作 (ch <- value)
发送操作的核心逻辑：

```go
func chansend(c *hchan, ep unsafe.Pointer, block bool, callerpc uintptr) bool {
    // 如果channel为nil且非阻塞，直接返回false
    if c == nil {
        if !block {
            return false
        }
        gopark(nil, nil, waitReasonChanSendNilChan, traceEvGoStop, 2)
        throw("unreachable")
    }

    // 快速路径：非阻塞发送且channel未关闭且缓冲区未满
    if !block && c.closed == 0 && full(c) {
        return false
    }

    // 获取锁
    lock(&c.lock)

    // 如果channel已关闭，panic
    if c.closed != 0 {
        unlock(&c.lock)
        panic(plainError("send on closed channel"))
    }

    // 如果有等待接收的goroutine，直接发送给它
    if sg := c.recvq.dequeue(); sg != nil {
        send(c, sg, ep, func() { unlock(&c.lock) }, 3)
        return true
    }

    // 如果缓冲区未满，放入缓冲区
    if c.qcount < c.dataqsiz {
        qp := chanbuf(c, c.sendx)
        typedmemmove(c.elemtype, qp, ep)
        c.sendx++
        if c.sendx == c.dataqsiz {
            c.sendx = 0
        }
        c.qcount++
        unlock(&c.lock)
        return true
    }

    // 缓冲区已满，非阻塞模式返回false
    if !block {
        unlock(&c.lock)
        return false
    }

    // 阻塞模式：将当前goroutine加入发送等待队列
    gp := getg()
    mysg := acquireSudog()
    mysg.releasetime = 0
    mysg.elem = ep
    mysg.waitlink = nil
    mysg.g = gp
    mysg.isSelect = false
    gp.waiting = mysg
    gp.param = nil
    
    c.sendq.enqueue(mysg)
    
    // 挂起当前goroutine
    gopark(chanparkcommit, unsafe.Pointer(&c.lock), waitReasonChanSend, traceEvGoBlockSend, 2)

    // 唤醒后清理
    mysg = gp.waiting
    gp.waiting = nil
    gp.activeStackChans = false
    closed := !mysg.success
    gp.param = nil
    
    releaseSudog(mysg)
    if closed {
        panic(plainError("send on closed channel"))
    }
    return true
}
```

发送操作的流程：
1. 检查 channel 是否为 nil
2. 如果有等待接收的 goroutine，直接传递数据
3. 如果缓冲区未满，放入缓冲区
4. 否则阻塞当前 goroutine，加入发送等待队列

### 接收操作 (<- ch)
接收操作的核心逻辑：

```go
func chanrecv(c *hchan, ep unsafe.Pointer, block bool) (selected, received bool) {
    // 如果channel为nil且非阻塞，直接返回
    if c == nil {
        if !block {
            return
        }
        gopark(nil, nil, waitReasonChanReceiveNilChan, traceEvGoStop, 2)
        throw("unreachable")
    }

    // 快速路径：非阻塞接收且channel未关闭且缓冲区为空
    if !block && empty(c) && c.closed == 0 {
        return
    }

    // 获取锁
    lock(&c.lock)

    // 如果channel已关闭且缓冲区为空，返回零值
    if c.closed != 0 && c.qcount == 0 {
        unlock(&c.lock)
        if ep != nil {
            typedmemclr(c.elemtype, ep)
        }
        return true, false
    }

    // 如果有等待发送的goroutine，直接接收
    if sg := c.sendq.dequeue(); sg != nil {
        recv(c, sg, ep, func() { unlock(&c.lock) }, 3)
        return true, true
    }

    // 如果缓冲区有数据，从缓冲区取
    if c.qcount > 0 {
        qp := chanbuf(c, c.recvx)
        if ep != nil {
            typedmemmove(c.elemtype, ep, qp)
        }
        typedmemclr(c.elemtype, qp)
        c.recvx++
        if c.recvx == c.dataqsiz {
            c.recvx = 0
        }
        c.qcount--
        unlock(&c.lock)
        return true, true
    }

    // 缓冲区为空，非阻塞模式返回
    if !block {
        unlock(&c.lock)
        return
    }

    // 阻塞模式：将当前goroutine加入接收等待队列
    gp := getg()
    mysg := acquireSudog()
    mysg.releasetime = 0
    mysg.elem = ep
    mysg.waitlink = nil
    gp.waiting = mysg
    gp.param = nil
    
    c.recvq.enqueue(mysg)
    
    // 挂起当前goroutine
    gopark(chanparkcommit, unsafe.Pointer(&c.lock), waitReasonChanReceive, traceEvGoBlockRecv, 2)

    // 唤醒后清理
    mysg = gp.waiting
    gp.waiting = nil
    gp.activeStackChans = false
    closed := !mysg.success
    gp.param = nil
    
    releaseSudog(mysg)
    return true, !closed
}
```

接收操作的流程：
1. 检查 channel 是否为 nil
2. 如果 channel 已关闭且缓冲区为空，返回零值
3. 如果有等待发送的 goroutine，直接接收数据
4. 如果缓冲区有数据，从缓冲区取
5. 否则阻塞当前 goroutine，加入接收等待队列

### 关闭操作 (close(ch))
关闭 channel 的核心逻辑：

```go
func closechan(c *hchan) {
    // 如果channel为nil，panic
    if c == nil {
        panic(plainError("close of nil channel"))
    }

    // 获取锁
    lock(&c.lock)
    
    // 如果channel已关闭，panic
    if c.closed != 0 {
        unlock(&c.lock)
        panic(plainError("close of closed channel"))
    }

    // 标记channel为关闭
    c.closed = 1

    // 唤醒所有等待接收的goroutine
    var glist gList
    for {
        sg := c.recvq.dequeue()
        if sg == nil {
            break
        }
        if sg.elem != nil {
            typedmemclr(c.elemtype, sg.elem)
        }
        sg.success = false
        glist.push(sg)
    }

    // 唤醒所有等待发送的goroutine（它们会panic）
    for {
        sg := c.sendq.dequeue()
        if sg == nil {
            break
        }
        sg.elem = nil
        sg.success = false
        glist.push(sg)
    }
    
    unlock(&c.lock)

    // 唤醒所有等待的goroutine
    for !glist.empty() {
        sg := glist.pop()
        sg.releasetime = cputicks()
        goready(sg.g, 3)
    }
}
```

关闭 channel 时会：
1. 标记 channel 为关闭状态
2. 唤醒所有等待接收的 goroutine（它们会收到零值）
3. 唤醒所有等待发送的 goroutine（它们会 panic）

## channel类型
### 无缓冲channel
无缓冲 channel 没有缓冲区，发送和接收操作必须同步进行：

```go
ch := make(chan int) // 无缓冲channel

go func() {
    ch <- 1 // 阻塞直到有接收者
}()

x := <-ch // 阻塞直到有发送者
```

### 有缓冲channel
有缓冲 channel 有一个缓冲区，可以存储多个元素：

```go
ch := make(chan int, 3) // 容量为3的有缓冲channel

ch <- 1 // 立即返回，放入缓冲区
ch <- 2 // 立即返回
ch <- 3 // 立即返回
// ch <- 4 // 阻塞，等待缓冲区有空间

x := <-ch // 从缓冲区取数据
```

### 单向channel
单向 channel 只能进行发送或接收操作：

```go
// 只能发送的channel
var sendCh chan<- int

// 只能接收的channel  
var recvCh <-chan int

// 转换示例
ch := make(chan int, 10)
sendCh = ch // 双向channel可以隐式转换为单向发送channel
recvCh = ch // 双向channel可以隐式转换为单向接收channel
```

## channel的并发特性
### 线程安全
channel 是线程安全的，内部通过互斥锁 `lock` 保证并发访问的安全性。

### 选择语句 (select)
`select` 语句可以同时等待多个 channel 操作：

```go
select {
case <-ch1:
    // 处理ch1的数据
case ch2 <- value:
    // 向ch2发送数据
case <-time.After(time.Second):
    // 超时处理
default:
    // 所有channel都不可用时执行
}
```

### 遍历channel
可以使用 `for range` 遍历 channel，直到 channel 关闭：

```go
ch := make(chan int, 3)
ch <- 1
ch <- 2
ch <- 3
close(ch)

for v := range ch {
    fmt.Println(v) // 输出：1 2 3
}
```

## 代码示例
**示例1: 无缓冲channel实现goroutine同步**

```go
package main

import "fmt"

func worker(id int, done chan bool) {
    fmt.Printf("Worker %d starting\n", id)
    // 模拟工作
    for i := 0; i < 3; i++ {
        fmt.Printf("Worker %d: %d\n", id, i)
    }
    done <- true // 通知完成
}

func main() {
    done := make(chan bool, 1)
    go worker(1, done)
    
    <-done // 等待worker完成
    fmt.Println("Worker finished")
}
```

输出：
```txt
Worker 1 starting
Worker 1: 0
Worker 1: 1
Worker 1: 2
Worker finished
```

**示例2: 有缓冲channel实现生产者-消费者模式**

```go
package main

import (
    "fmt"
    "time"
)

func producer(ch chan<- int) {
    for i := 0; i < 5; i++ {
        ch <- i
        fmt.Printf("Produced: %d\n", i)
    }
    close(ch)
}

func consumer(ch <-chan int) {
    for v := range ch {
        fmt.Printf("Consumed: %d\n", v)
        time.Sleep(time.Millisecond * 100)
    }
}

func main() {
    ch := make(chan int, 2) // 缓冲区大小为2
    
    go producer(ch)
    consumer(ch)
}
```

输出：
```txt
Produced: 0
Produced: 1
Produced: 2
Consumed: 0
Produced: 3
Consumed: 1
Produced: 4
Consumed: 2
Consumed: 3
Consumed: 4
```

**示例3: select语句处理多个channel**

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ch1 := make(chan string)
    ch2 := make(chan string)

    go func() {
        time.Sleep(time.Second * 1)
        ch1 <- "from ch1"
    }()

    go func() {
        time.Sleep(time.Second * 2)
        ch2 <- "from ch2"
    }()

    for i := 0; i < 2; i++ {
        select {
        case msg1 := <-ch1:
            fmt.Println("Received:", msg1)
        case msg2 := <-ch2:
            fmt.Println("Received:", msg2)
        }
    }
}
```

输出：
```txt
Received: from ch1
Received: from ch2
```

## 常见面试题
### 1. channel 的发送和接收在什么情况下会阻塞？

**答案：**
- **无缓冲 channel**：发送操作会阻塞，直到有接收者；接收操作会阻塞，直到有发送者。
- **有缓冲 channel**：发送操作在缓冲区满时阻塞；接收操作在缓冲区空时阻塞。

### 2. 向 nil channel 发送或接收会发生什么？

**答案：**
向 nil channel 发送或接收数据会永久阻塞（deadlock），goroutine 将无法继续执行。

```go
var ch chan int // nil channel
ch <- 1         // 永久阻塞
x := <-ch       // 永久阻塞
```

### 3. 向已关闭的 channel 发送数据会发生什么？

**答案：**
向已关闭的 channel 发送数据会导致 panic。

```go
ch := make(chan int, 1)
close(ch)
ch <- 1 // panic: send on closed channel
```

### 4. 从已关闭的 channel 接收数据会发生什么？

**答案：**
从已关闭的 channel 接收数据会立即返回，如果缓冲区还有数据则返回数据，否则返回该类型的零值。可以使用第二个返回值判断是否成功接收到数据。

```go
ch := make(chan int, 2)
ch <- 1
close(ch)

x, ok := <-ch // x=1, ok=true
y, ok := <-ch // y=0, ok=false（缓冲区已空）
```

### 5. 如何优雅地关闭 channel？

**答案：**
通常由发送方负责关闭 channel，接收方通过 `for range` 或检查第二个返回值来判断 channel 是否关闭。

```go
// 发送方关闭 channel
func producer(ch chan<- int) {
    for i := 0; i < 5; i++ {
        ch <- i
    }
    close(ch) // 发送完毕，关闭 channel
}

// 接收方处理关闭的 channel
func consumer(ch <-chan int) {
    for v := range ch { // 自动检测 channel 是否关闭
        fmt.Println(v)
    }
    fmt.Println("channel closed")
}
```

### 6. channel 是线程安全的吗？

**答案：**
是的，channel 是线程安全的。Go runtime 内部使用互斥锁保护 channel 的并发访问，多个 goroutine 可以安全地向同一个 channel 发送或接收数据。

### 7. 无缓冲 channel 和有缓冲 channel 的区别？

**答案：**
| 特性 | 无缓冲 channel | 有缓冲 channel |
|------|---------------|---------------|
| 同步方式 | 同步通信 | 异步通信 |
| 发送阻塞 | 必须等待接收者 | 缓冲区满时阻塞 |
| 接收阻塞 | 必须等待发送者 | 缓冲区空时阻塞 |
| 适用场景 | goroutine 间同步 | 生产消费解耦 |

### 8. 单向 channel 有什么作用？

**答案：**
单向 channel 用于限制 channel 的使用方式，提高代码的安全性和可读性。

- `chan<- T`：只能发送的 channel
- `<-chan T`：只能接收的 channel

```go
func sendOnly(ch chan<- int) {
    ch <- 1 // 只能发送
    // <-ch // 编译错误：cannot receive from send-only channel
}

func receiveOnly(ch <-chan int) {
    <-ch // 只能接收
    // ch <- 1 // 编译错误：cannot send to receive-only channel
}
```

### 9. select 语句中多个 case 同时就绪会怎样？

**答案：**
select 语句会随机选择一个就绪的 case 执行，保证公平性。

```go
ch1 := make(chan int)
ch2 := make(chan int)

go func() { ch1 <- 1 }()
go func() { ch2 <- 2 }()

select {
case <-ch1:
    fmt.Println("ch1")
case <-ch2:
    fmt.Println("ch2")
}
// 输出可能是 "ch1" 或 "ch2"，随机选择
```

### 10. 如何实现一个定时任务？

**答案：**
可以使用 `time.After` 和 select 语句实现定时任务。

```go
func main() {
    ticker := time.Tick(time.Second)
    for {
        select {
        case <-ticker:
            fmt.Println("tick")
        case <-time.After(5 * time.Second):
            fmt.Println("timeout")
            return
        }
    }
}
```

### 11. 什么是 channel 的死锁？常见场景有哪些？

**答案：**
当 goroutine 等待永远不会发生的操作时就会发生死锁。

**常见场景：**
1. 单 goroutine 向无缓冲 channel 发送数据后立即接收
2. 两个 goroutine 互相等待对方发送数据
3. goroutine 等待 nil channel 的操作

```go
// 场景1：单 goroutine 死锁
func main() {
    ch := make(chan int)
    ch <- 1  // 阻塞，等待接收者
    <-ch     // 永远不会执行到
}

// 场景2：互相等待死锁
func main() {
    ch1 := make(chan int)
    ch2 := make(chan int)

    go func() {
        ch1 <- 1
        <-ch2 // 等待 ch2
    }()

    go func() {
        ch2 <- 2
        <-ch1 // 等待 ch1
    }()
}
```

### 12. 如何使用 channel 实现信号量？

**答案：**
可以使用有缓冲 channel 实现信号量，限制并发数量。

```go
func main() {
    sem := make(chan struct{}, 3) // 最多3个并发

    for i := 0; i < 10; i++ {
        sem <- struct{}{} // 获取信号量
        go func(id int) {
            defer func() { <-sem }() // 释放信号量
            fmt.Printf("Worker %d working\n", id)
            time.Sleep(time.Second)
        }(i)
    }

    // 等待所有 goroutine 完成
    for i := 0; i < 3; i++ {
        sem <- struct{}{}
    }
}
```

### 13. 如何使用 channel 实现超时控制？

**答案：**
使用 `time.After` 和 select 语句实现超时控制。

```go
func doWork() error {
    ch := make(chan struct{})
    
    go func() {
        // 模拟耗时操作
        time.Sleep(2 * time.Second)
        close(ch)
    }()

    select {
    case <-ch:
        return nil
    case <-time.After(1 * time.Second):
        return fmt.Errorf("timeout")
    }
}
```

### 14. channel 和锁的区别？

**答案：**
| 特性 | channel | 锁 |
|------|---------|----|
| 通信方式 | 通过共享内存传递数据 | 通过互斥访问共享内存 |
| 同步机制 | 发送/接收阻塞 | 获取/释放锁阻塞 |
| 适用场景 | goroutine 间通信 | 保护共享资源 |
| 编程范式 | CSP（通信顺序进程） | 传统并发控制 |

### 15. 关闭 channel 后，之前发送的数据还能接收吗？

**答案：**
可以。关闭 channel 后，缓冲区中已有的数据仍然可以被接收，直到缓冲区为空。

```go
ch := make(chan int, 3)
ch <- 1
ch <- 2
ch <- 3
close(ch)

fmt.Println(<-ch) // 1
fmt.Println(<-ch) // 2
fmt.Println(<-ch) // 3
fmt.Println(<-ch) // 0（零值）
```

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

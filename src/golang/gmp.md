# GMP模型

## GMP简介

### 什么是GMP

GMP是Go语言的并发调度模型，由Goroutine、Machine、Processor三个核心组件组成，实现了高效的用户态线程调度。

### GMP架构图

```
┌─────────────────────────────────────────────────────────────────────┐
│                        OS Scheduler                               │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────────┐    ┌─────────────────┐                        │
│  │   Machine (M0)  │    │   Machine (M1)  │  ...                   │
│  │  ┌───────────┐  │    │  ┌───────────┐  │                        │
│  │  │ Processor │  │    │  │ Processor │  │                        │
│  │  │   (P0)    │  │    │   (P1)      │  │                        │
│  │  │  LocalQ   │  │    │   LocalQ    │  │                        │
│  │  │ [G0,G1,G2]│  │    │   [G3,G4]   │  │                        │
│  │  └─────┬─────┘  │    │   └────┬────┘  │                        │
│  └────────┼────────┘    └────────┼────────┘                        │
│           │                      │                                 │
│           ▼                      ▼                                 │
│  ┌─────────────────────────────────────────────┐                   │
│  │           Global Queue (GlobalQ)           │                   │
│  │            [G5, G6, G7, G8, G9]            │                   │
│  └─────────────────────────────────────────────┘                   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## GMP数据结构

### G（Goroutine）

```go
type g struct {
    stack       stack      // 栈信息
    stackguard0 uintptr    // 栈溢出检查边界
    stackguard1 uintptr    // 栈溢出检查边界
    _panic      *_panic    // 恐慌信息
    _defer      *_defer    // defer链表
    m           *m         // 当前绑定的M
    sched       gobuf      // 调度保存的上下文
    status      uint32     // 状态：idle, runnable, running, etc.
    // ...
}
```

### M（Machine）

```go
type m struct {
    g0          *g         // goroutine 0，用于执行调度和系统调用
    gsignal     *g         // 信号处理goroutine
    curg        *g         // 当前执行的goroutine
    p           *p         // 当前绑定的P
    nextp       *p         // 下一个要绑定的P
    // ...
}
```

### P（Processor）

```go
type p struct {
    id          int
    status      uint32     // idle, running, etc.
    m           *m         // 当前绑定的M
    runq        [256]guintptr // 本地运行队列（固定大小）
    runqhead    uint32     // 队列头指针
    runqtail    uint32     // 队列尾指针
    gcAssistTime int64     // GC辅助时间
    // ...
}
```

---

## 调度机制

### 调度循环

```go
// 源码位于 go/src/runtime/proc.go

func schedule() {
    // 无限循环调度
    for {
        // 1. 从本地队列获取G
        g := runqget(_g_.m.p.ptr())
        
        // 2. 如果本地队列为空，尝试从全局队列获取
        if g == nil {
            g = globrunqget(_g_.m.p.ptr(), 1)
        }
        
        // 3. 如果全局队列也为空，尝试工作窃取
        if g == nil {
            g = findrunnable()
        }
        
        // 4. 执行G
        execute(g, false)
    }
}
```

### 工作窃取算法

```go
// 工作窃取：从其他P的本地队列窃取G
func findrunnable() *g {
    // 尝试从其他P窃取工作
    for i := 0; i < int(gomaxprocs); i++ {
        p := allp[(sched.pidle + i) % gomaxprocs]
        if p == nil || p.status != _Pidle {
            continue
        }
        
        // 尝试窃取一半的G
        if g := runqsteal(_g_.m.p.ptr(), p); g != nil {
            return g
        }
    }
    
    // 从全局队列获取
    if g := globrunqget(_g_.m.p.ptr(), 0); g != nil {
        return g
    }
    
    return nil
}
```

---

## Goroutine创建与调度

### 创建Goroutine

```go
// 创建goroutine的底层函数
func newproc(siz int32, fn *funcval) {
    // 获取当前G的栈指针
    gp := malg(siz)
    
    // 设置G的执行函数
    gp.sched.pc = funcPC(fn)
    gp.sched.sp = gp.stack.hi
    
    // 将G加入调度队列
    runqput(_g_.m.p.ptr(), gp)
    
    // 如果有空闲的P，唤醒一个M
    if atomic.Load(&sched.npidle) != 0 && atomic.Load(&sched.nmspinning) == 0 {
        startm(nil, true)
    }
}
```

### 执行Goroutine

```go
func execute(gp *g, inheritTime bool) {
    // 切换到目标G的上下文
    casgstatus(gp, _Grunnable, _Grunning)
    gp.m = _g_.m
    
    // 保存当前G的上下文
    save(&_g_.sched)
    
    // 切换到新G
    gogo(&gp.sched)
}
```

---

## 阻塞处理

### Goroutine阻塞

```go
// 当G阻塞时的处理逻辑
func park_m(gp *g) {
    // 保存G的上下文
    save(&gp.sched)
    
    // 将G从运行队列移除
    runqremove(_g_.m.p.ptr(), gp)
    
    // 设置G状态为waiting
    casgstatus(gp, _Grunning, _Gwaiting)
    
    // M与P分离
    dropm()
    
    // 调度其他G
    schedule()
}
```

### 网络IO处理

```go
// 使用netpoller处理网络IO
func netpoll(block bool) *g {
    // 检查是否有就绪的IO事件
    if block {
        // 阻塞等待IO事件
        gopark(netpollblock, nil, waitReasonIOWait, traceEvGoBlockNet, 1)
    }
    
    // 获取就绪的G
    gp := netpollgrab()
    return gp
}
```

---

## 抢占调度

### 协作式调度（Go 1.14之前）

```go
// 协作式调度：需要主动让出CPU
func sysmon() {
    // 监控长时间运行的G
    for {
        // 检查是否有G运行超过10ms
        if gp != nil && gp.status == _Grunning {
            if t := gp.sched.timestamp; t != 0 && now - t > 10*1e6 {
                // 发送抢占信号
                preemptone(gp)
            }
        }
        // ...
    }
}
```

### 基于信号的抢占（Go 1.14+）

```go
// 基于信号的抢占
func sigPreemptHandler(sig uint32, info *siginfo, ctx unsafe.Pointer) {
    // 获取当前运行的G
    gp := getg()
    
    // 设置抢占标志
    gp.preempt = true
    
    // 在函数调用返回时触发调度
}
```

---

## 常见问题与陷阱

### 陷阱1：GOMAXPROCS设置不当

```go
// 错误：设置GOMAXPROCS为1，无法利用多核
func main() {
    runtime.GOMAXPROCS(1) // 只使用1个CPU核心
    
    // 大量并发任务，无法并行执行
    for i := 0; i < 100; i++ {
        go heavyTask(i)
    }
}
```

### 陷阱2：Goroutine泄漏

```go
// Goroutine泄漏示例
func leakyGoroutine() {
    ch := make(chan int)
    
    go func() {
        for {
            select {
            case val := <-ch:
                process(val)
            }
        }
    }()
    
    // ch从未关闭，goroutine永远不会退出
}
```

### 陷阱3：死锁

```go
// 死锁示例
func deadlock() {
    var wg sync.WaitGroup
    wg.Add(1)
    
    go func() {
        // 忘记调用wg.Done()
        process()
    }()
    
    wg.Wait() // 永远等待
}
```

---

## 面试题

### 基础题

1. **GMP分别代表什么？**
   - 答案：G代表Goroutine，M代表Machine，P代表Processor。

2. **Goroutine和线程的区别？**
   - 答案：Goroutine是用户态线程，创建成本低、调度开销小，可以创建成千上万；线程是内核态，创建成本高、数量有限。

3. **什么是工作窃取？**
   - 答案：当一个P的本地队列为空时，它会从其他P的本地队列或全局队列中窃取Goroutine来执行，实现负载均衡。

4. **GOMAXPROCS的作用是什么？**
   - 答案：设置可同时执行用户级代码的最大CPU核心数，默认等于CPU核心数。

5. **P的作用是什么？**
   - 答案：P是处理器，维护本地运行队列，提供Goroutine运行所需的上下文环境。

6. **M为什么需要绑定P？**
   - 答案：M需要绑定P才能执行Goroutine，P提供了运行环境和本地队列。

7. **什么是G0？**
   - 答案：G0是每个M都有的特殊Goroutine，用于执行调度和系统调用。

8. **Go的调度策略是什么？**
   - 答案：Go采用协作式调度+抢占式调度，Go 1.14之后引入了基于信号的抢占。

### 进阶题

9. **Goroutine的状态有哪些？**
   - 答案：idle（空闲）、runnable（可运行）、running（运行中）、waiting（等待）、dead（死亡）。

10. **如何查看当前Goroutine数量？**
    - 答案：使用runtime.NumGoroutine()函数。

11. **Goroutine的栈是如何增长的？**
    - 答案：Goroutine的栈默认大小为2KB，可以动态扩展，最大可达1GB。

12. **什么是调度器的工作窃取算法？**
    - 答案：当一个P的本地队列为空时，它会尝试从其他P的本地队列窃取一半的Goroutine，实现负载均衡。

13. **Go 1.14之前为什么会出现Goroutine饥饿？**
    - 答案：Go 1.14之前使用协作式调度，Goroutine需要主动让出CPU，如果一个Goroutine执行长时间计算而不调用函数，会一直占用CPU。

### 复杂场景题

14. **在高并发场景下，如何优化Goroutine调度性能？**
    - 答案：
      - 合理设置GOMAXPROCS
      - 避免创建过多Goroutine
      - 使用对象池复用资源
      - 避免长时间阻塞操作
      - 使用并发原语（channel、sync包）协调Goroutine

15. **如何排查Goroutine泄漏问题？**
    - 答案：
      - 使用runtime.NumGoroutine()监控Goroutine数量
      - 使用pprof分析Goroutine栈
      - 检查channel是否正确关闭
      - 检查WaitGroup是否正确使用
      - 检查select是否有default分支

16. **Goroutine和channel如何配合使用？**
    - 答案：
      - 使用channel进行Goroutine间通信
      - 使用channel控制并发数量
      - 使用channel传递数据和信号
      - 使用select处理多个channel

17. **什么是并发安全？如何保证并发安全？**
    - 答案：
      - 并发安全指多个Goroutine同时访问共享资源时不会产生数据竞争
      - 使用sync.Mutex、sync.RWMutex保护共享资源
      - 使用channel进行同步
      - 使用原子操作（sync/atomic包）

18. **在实际项目中，如何设计高效的并发模型？**
    - 答案：
      - 使用生产者-消费者模式
      - 使用worker pool模式
      - 使用context控制Goroutine生命周期
      - 使用errgroup管理一组Goroutine
      - 合理使用channel进行通信

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
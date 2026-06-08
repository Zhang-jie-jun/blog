# C++ 并发编程

## 并发编程简介

C++11 引入了标准并发库，支持多线程编程、同步原语和原子操作。

### 并发概念

| 概念 | 描述 |
|------|------|
| **进程** | 程序的执行实例，有独立的内存空间 |
| **线程** | 进程内的执行单元，共享进程内存 |
| **并发** | 多个任务同时进行（宏观） |
| **并行** | 多个任务真正同时执行（微观） |

---

## 1. 线程管理

### 创建线程

```cpp
#include <thread>
#include <iostream>

void hello() {
    std::cout << "Hello from thread!" << std::endl;
}

int main() {
    // 创建线程
    std::thread t(hello);
    
    // 等待线程完成
    t.join();
    
    return 0;
}
```

### 传递参数

```cpp
void printMessage(const std::string& msg) {
    std::cout << msg << std::endl;
}

int main() {
    std::string message = "Hello World";
    std::thread t(printMessage, std::ref(message));
    t.join();
    return 0;
}
```

### 线程分离

```cpp
void daemon() {
    // 后台任务
}

int main() {
    std::thread t(daemon);
    t.detach();  // 分离线程，不再管理
    return 0;
}
```

---

## 2. 互斥锁

### 基本用法

```cpp
#include <mutex>
#include <iostream>

std::mutex mtx;
int counter = 0;

void increment() {
    for (int i = 0; i < 100000; ++i) {
        mtx.lock();
        counter++;
        mtx.unlock();
    }
}

int main() {
    std::thread t1(increment);
    std::thread t2(increment);
    
    t1.join();
    t2.join();
    
    std::cout << "Counter: " << counter << std::endl;
    return 0;
}
```

### RAII 锁

```cpp
void safeIncrement() {
    for (int i = 0; i < 100000; ++i) {
        std::lock_guard<std::mutex> lock(mtx);
        counter++;
        // 自动释放锁
    }
}
```

### 递归互斥锁

```cpp
#include <recursive_mutex>

std::recursive_mutex rmtx;

void recursiveFunction(int depth) {
    std::lock_guard<std::recursive_mutex> lock(rmtx);
    
    if (depth > 0) {
        recursiveFunction(depth - 1);
    }
}
```

---

## 3. 条件变量

### 生产者-消费者模式

```cpp
#include <condition_variable>
#include <queue>

std::mutex mtx;
std::condition_variable cv;
std::queue<int> q;

void producer() {
    for (int i = 0; i < 10; ++i) {
        std::lock_guard<std::mutex> lock(mtx);
        q.push(i);
        std::cout << "Produced: " << i << std::endl;
        cv.notify_one();
    }
}

void consumer() {
    while (true) {
        std::unique_lock<std::mutex> lock(mtx);
        cv.wait(lock, []{ return !q.empty(); });
        
        int val = q.front();
        q.pop();
        std::cout << "Consumed: " << val << std::endl;
        
        if (val == 9) break;
    }
}

int main() {
    std::thread t1(producer);
    std::thread t2(consumer);
    
    t1.join();
    t2.join();
    return 0;
}
```

---

## 4. 原子操作

### 基本原子类型

```cpp
#include <atomic>

std::atomic<int> atomic_counter(0);

void atomicIncrement() {
    for (int i = 0; i < 100000; ++i) {
        atomic_counter.fetch_add(1, std::memory_order_relaxed);
    }
}

int main() {
    std::thread t1(atomicIncrement);
    std::thread t2(atomicIncrement);
    
    t1.join();
    t2.join();
    
    std::cout << "Atomic Counter: " << atomic_counter << std::endl;
    return 0;
}
```

### 内存顺序

```cpp
// memory_order_relaxed：无顺序保证
// memory_order_acquire：读取时获得同步
// memory_order_release：写入时释放同步
// memory_order_seq_cst：顺序一致性
```

---

## 5. 线程池

### 简单线程池实现

```cpp
#include <thread>
#include <queue>
#include <functional>

class ThreadPool {
private:
    std::vector<std::thread> workers;
    std::queue<std::function<void()>> tasks;
    std::mutex mtx;
    std::condition_variable cv;
    bool stop;
    
public:
    ThreadPool(size_t numThreads) : stop(false) {
        for (size_t i = 0; i < numThreads; ++i) {
            workers.emplace_back([this] {
                while (true) {
                    std::function<void()> task;
                    
                    {
                        std::unique_lock<std::mutex> lock(this->mtx);
                        this->cv.wait(lock, [this] {
                            return this->stop || !this->tasks.empty();
                        });
                        
                        if (this->stop && this->tasks.empty()) return;
                        
                        task = std::move(this->tasks.front());
                        this->tasks.pop();
                    }
                    
                    task();
                }
            });
        }
    }
    
    ~ThreadPool() {
        {
            std::lock_guard<std::mutex> lock(mtx);
            stop = true;
        }
        cv.notify_all();
        
        for (auto& worker : workers) {
            worker.join();
        }
    }
    
    template <typename F>
    void enqueue(F&& f) {
        {
            std::lock_guard<std::mutex> lock(mtx);
            tasks.emplace(std::forward<F>(f));
        }
        cv.notify_one();
    }
};
```

---

## 6. 异步操作

### std::async

```cpp
#include <future>

int calculate() {
    // 耗时计算
    return 42;
}

int main() {
    // 异步执行
    std::future<int> result = std::async(std::launch::async, calculate);
    
    // 做其他事情
    std::cout << "Waiting..." << std::endl;
    
    // 获取结果
    std::cout << "Result: " << result.get() << std::endl;
    return 0;
}
```

---

## 常见面试题

### 1. 互斥锁和原子操作的区别？

**答案：**
- **互斥锁**：用于保护临界区，适合复杂操作
- **原子操作**：硬件级别的原子性，适合简单操作，性能更好

### 2. 死锁的条件是什么？如何避免？

**答案：**
死锁条件：互斥、持有并等待、不可剥夺、循环等待。
避免方法：按顺序加锁、使用超时机制、使用 RAII。

### 3. 条件变量为什么需要搭配互斥锁？

**答案：**
条件变量的 wait 操作需要原子地释放锁并阻塞线程，避免竞态条件。

### 4. std::async 的两种启动策略？

**答案：**
- `std::launch::async`：异步执行
- `std::launch::deferred`：延迟执行，直到调用 get()

### 5. 线程池的优点？

**答案：**
- 减少线程创建销毁的开销
- 控制并发数
- 便于任务管理和调度

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

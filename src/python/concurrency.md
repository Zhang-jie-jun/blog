# Python 并发编程

## 并发编程简介

Python 提供了多种并发编程方式，包括多线程、多进程和协程。

---

## 1. 多线程

### threading 模块

```python
import threading
import time

def worker(name):
    print(f"Worker {name} started")
    time.sleep(2)
    print(f"Worker {name} finished")

# 创建线程
threads = []
for i in range(3):
    t = threading.Thread(target=worker, args=(f"Thread-{i}",))
    threads.append(t)
    t.start()

# 等待所有线程完成
for t in threads:
    t.join()
```

### 线程同步

```python
import threading

counter = 0
lock = threading.Lock()

def increment():
    global counter
    for _ in range(100000):
        with lock:
            counter += 1

t1 = threading.Thread(target=increment)
t2 = threading.Thread(target=increment)
t1.start()
t2.start()
t1.join()
t2.join()
print(counter)  # 200000
```

---

## 2. 多进程

### multiprocessing 模块

```python
import multiprocessing
import os

def worker(name):
    print(f"Worker {name} started, PID: {os.getpid()}")
    return f"Result from {name}"

if __name__ == "__main__":
    processes = []
    for i in range(3):
        p = multiprocessing.Process(target=worker, args=(f"Process-{i}",))
        processes.append(p)
        p.start()
    
    for p in processes:
        p.join()
```

### 进程池

```python
from multiprocessing import Pool

def square(x):
    return x ** 2

if __name__ == "__main__":
    with Pool(4) as pool:
        results = pool.map(square, [1, 2, 3, 4, 5])
        print(results)  # [1, 4, 9, 16, 25]
```

---

## 3. 协程

### asyncio 模块

```python
import asyncio

async def say_hello(name):
    print(f"Hello, {name}")
    await asyncio.sleep(1)
    print(f"Goodbye, {name}")

async def main():
    # 创建任务
    task1 = asyncio.create_task(say_hello("Alice"))
    task2 = asyncio.create_task(say_hello("Bob"))
    
    # 等待任务完成
    await task1
    await task2

# 运行事件循环
asyncio.run(main())
```

### 异步IO

```python
import asyncio
import aiohttp

async def fetch(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.text()

async def main():
    urls = [
        "https://api.example.com/data1",
        "https://api.example.com/data2",
        "https://api.example.com/data3"
    ]
    
    tasks = [fetch(url) for url in urls]
    results = await asyncio.gather(*tasks)
    
    for result in results:
        print(result[:100])

asyncio.run(main())
```

---

## 4. GIL 全局解释器锁

### GIL 的影响

```python
# CPU 密集型任务：多线程不提速
import threading

def cpu_bound():
    result = 0
    for i in range(10**7):
        result += i
    return result

# 多线程执行
threads = []
for _ in range(4):
    t = threading.Thread(target=cpu_bound)
    threads.append(t)
    t.start()

for t in threads:
    t.join()
```

### 使用多进程绕过 GIL

```python
from multiprocessing import Pool

def cpu_bound():
    result = 0
    for i in range(10**7):
        result += i
    return result

if __name__ == "__main__":
    with Pool(4) as pool:
        results = pool.map(cpu_bound, [1, 1, 1, 1])
```

---

## 5. 线程安全的数据结构

```python
from collections import deque
import threading

class ThreadSafeQueue:
    def __init__(self):
        self.queue = deque()
        self.lock = threading.Lock()
    
    def enqueue(self, item):
        with self.lock:
            self.queue.append(item)
    
    def dequeue(self):
        with self.lock:
            if self.queue:
                return self.queue.popleft()
            return None
```

---

## 常见面试题

### 1. GIL 是什么？

**答案：**
GIL（Global Interpreter Lock）是 Python 解释器中的互斥锁，确保同一时刻只有一个线程执行 Python 字节码。

### 2. 多线程和多进程的区别？

**答案：**
| 特性 | 多线程 | 多进程 |
|------|--------|--------|
| 内存共享 | 共享 | 不共享 |
| GIL 影响 | 受影响 | 不受影响 |
| 开销 | 低 | 高 |
| 适用场景 | IO 密集型 | CPU 密集型 |

### 3. 协程是什么？

**答案：**
协程是用户态的轻量级线程，由程序员控制调度，支持异步编程。

### 4. async/await 的作用？

**答案：**
async 定义异步函数，await 暂停执行等待异步操作完成，实现非阻塞 IO。

### 5. 什么时候使用多进程？

**答案：**
当任务是 CPU 密集型且需要利用多核 CPU 时，使用多进程。

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
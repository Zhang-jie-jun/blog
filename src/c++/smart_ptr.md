# C++ 智能指针

## 智能指针简介

智能指针是 C++11 引入的自动内存管理工具，通过 RAII 机制自动释放动态分配的内存，避免内存泄漏。

### 智能指针类型

| 类型 | 所有权 | 特点 |
|------|--------|------|
| **unique_ptr** | 独占 | 不可拷贝，可移动 |
| **shared_ptr** | 共享 | 引用计数，可拷贝 |
| **weak_ptr** | 弱引用 | 不增加引用计数 |
| **auto_ptr** | 独占 | 已废弃，不推荐使用 |

---

## 1. std::unique_ptr

### 基本用法

```cpp
#include <memory>

// 创建 unique_ptr
std::unique_ptr<int> p1(new int(42));

// 使用 make_unique（C++14）
std::unique_ptr<int> p2 = std::make_unique<int>(100);

// 访问对象
std::cout << *p1 << std::endl;
std::cout << p1->get() << std::endl;

// 所有权转移
std::unique_ptr<int> p3 = std::move(p1);
// p1 现在为空

// 重置
p3.reset(new int(200));

// 检查是否为空
if (p3) {
    std::cout << *p3 << std::endl;
}
```

### 自定义删除器

```cpp
// 使用 lambda 作为删除器
std::unique_ptr<FILE, decltype(&fclose)> file(fopen("test.txt", "r"), &fclose);

// 自定义删除器
auto deleter = [](int* p) {
    std::cout << "Deleting: " << *p << std::endl;
    delete p;
};
std::unique_ptr<int, decltype(deleter)> p(new int(42), deleter);
```

---

## 2. std::shared_ptr

### 基本用法

```cpp
#include <memory>

// 创建 shared_ptr
std::shared_ptr<int> p1 = std::make_shared<int>(42);

// 引用计数
std::cout << "Count: " << p1.use_count() << std::endl;  // 1

// 拷贝共享
std::shared_ptr<int> p2 = p1;
std::cout << "Count: " << p1.use_count() << std::endl;  // 2

// 作用域示例
{
    std::shared_ptr<int> p3 = p1;
    std::cout << "Count: " << p1.use_count() << std::endl;  // 3
}
std::cout << "Count: " << p1.use_count() << std::endl;  // 2
```

### 循环引用问题

```cpp
// 错误：循环引用导致内存泄漏
struct Node {
    std::shared_ptr<Node> next;
};

auto node1 = std::make_shared<Node>();
auto node2 = std::make_shared<Node>();
node1->next = node2;
node2->next = node1;  // 循环引用
```

---

## 3. std::weak_ptr

### 解决循环引用

```cpp
struct Node {
    std::weak_ptr<Node> next;  // 使用 weak_ptr
};

auto node1 = std::make_shared<Node>();
auto node2 = std::make_shared<Node>();
node1->next = node2;
node2->next = node1;  // 不会增加引用计数
```

### 基本用法

```cpp
std::shared_ptr<int> sp = std::make_shared<int>(42);
std::weak_ptr<int> wp = sp;

// 检查引用是否有效
if (wp.expired()) {
    std::cout << "Expired" << std::endl;
}

// 获取 shared_ptr
if (auto locked = wp.lock()) {
    std::cout << *locked << std::endl;
}
```

---

## 4. 智能指针比较

### 性能对比

```cpp
// unique_ptr 性能测试
void test_unique_ptr() {
    std::unique_ptr<int> p;
    for (int i = 0; i < 1000000; ++i) {
        p = std::make_unique<int>(i);
    }
}

// shared_ptr 性能测试
void test_shared_ptr() {
    std::shared_ptr<int> p;
    for (int i = 0; i < 1000000; ++i) {
        p = std::make_shared<int>(i);
    }
}
```

### 使用场景选择

| 场景 | 推荐智能指针 |
|------|------------|
| 独占所有权 | unique_ptr |
| 共享所有权 | shared_ptr |
| 解决循环引用 | weak_ptr |
| Pimpl 惯用法 | unique_ptr |
| 缓存场景 | weak_ptr |

---

## 5. 常见陷阱

### 陷阱1：裸指针与智能指针混用

```cpp
int* raw = new int(42);
std::unique_ptr<int> p(raw);
delete raw;  // 错误：重复释放
```

### 陷阱2：多个智能指针管理同一对象

```cpp
int* raw = new int(42);
std::unique_ptr<int> p1(raw);
std::unique_ptr<int> p2(raw);  // 错误：double free
```

### 陷阱3：循环引用

```cpp
struct A {
    std::shared_ptr<B> b;
};

struct B {
    std::shared_ptr<A> a;
};

auto a = std::make_shared<A>();
auto b = std::make_shared<B>();
a->b = b;
b->a = a;  // 循环引用，内存泄漏
```

---

## 常见面试题

### 1. unique_ptr 和 shared_ptr 的区别？

**答案：**
- **unique_ptr**：独占所有权，不可拷贝，移动语义，性能更好
- **shared_ptr**：共享所有权，引用计数，可拷贝，有额外开销

### 2. 为什么需要 weak_ptr？

**答案：**
weak_ptr 用于解决 shared_ptr 的循环引用问题，它不增加引用计数，当所有 shared_ptr 都销毁后，weak_ptr 自动失效。

### 3. make_shared 和直接 new 的区别？

**答案：**
- **make_shared**：一次分配内存（对象+控制块），更高效，异常安全
- **new**：两次分配内存（先对象，再控制块）

### 4. 什么是引用计数？

**答案：**
引用计数是一种内存管理技术，每个对象维护一个计数器，记录有多少指针指向它。当计数器变为0时，自动释放对象。

### 5. 智能指针的线程安全性？

**答案：**
- shared_ptr 的引用计数操作是原子的
- 多个线程可以同时读写不同的 shared_ptr
- 同一个 shared_ptr 的读写需要同步保护

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
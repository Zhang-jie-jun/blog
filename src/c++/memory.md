# C++ 内存管理

## 内存管理简介

C++ 的内存管理是其核心特性之一，提供了手动和自动两种内存管理方式。理解内存管理对于编写高效、安全的 C++ 代码至关重要。

### 内存区域划分

| 区域 | 用途 | 生命周期 |
|------|------|---------|
| **栈（Stack）** | 局部变量、函数参数 | 自动分配和释放 |
| **堆（Heap）** | 动态分配的对象 | 手动管理 |
| **全局/静态存储区** | 全局变量、静态变量 | 程序整个运行期间 |
| **常量存储区** | 字符串常量、const 变量 | 程序整个运行期间 |

---

## 1. 栈内存

栈内存是自动管理的，用于存储局部变量和函数参数。

### 特点
- **自动分配和释放**：函数调用时分配，函数返回时释放
- **大小有限**：通常为几 MB
- **效率高**：分配和释放只需移动栈指针

### 示例
```cpp
void func() {
    int x = 10;        // 栈上分配
    int arr[100];      // 栈上分配数组
    std::string str;   // 对象在栈上，但其内部数据可能在堆上
}
```

---

## 2. 堆内存

堆内存需要手动分配和释放，使用 `new` 和 `delete` 操作符。

### 基本操作

```cpp
// 分配单个对象
int* p = new int;
*p = 42;
delete p;

// 分配数组
int* arr = new int[10];
arr[0] = 1;
delete[] arr;

// 分配并初始化
int* q = new int(100);
delete q;

// 动态分配对象
class MyClass {
public:
    MyClass(int x) : val(x) {}
private:
    int val;
};

MyClass* obj = new MyClass(42);
delete obj;
```

### 常见问题

#### 内存泄漏
```cpp
void leak() {
    int* p = new int(10);
    // 忘记 delete，导致内存泄漏
}
```

#### 双重释放
```cpp
void doubleFree() {
    int* p = new int(10);
    delete p;
    delete p;  // 未定义行为
}
```

#### 野指针
```cpp
void wildPointer() {
    int* p = new int(10);
    delete p;
    *p = 20;  // 未定义行为，p 是野指针
}
```

---

## 3. 智能指针

C++11 引入了智能指针，实现自动内存管理，避免内存泄漏。

### std::unique_ptr

独占所有权的智能指针，同一时间只能有一个 unique_ptr 指向对象。

```cpp
#include <memory>

void uniquePtrExample() {
    std::unique_ptr<int> p(new int(42));
    std::cout << *p << std::endl;
    
    // 所有权转移
    std::unique_ptr<int> q = std::move(p);
    
    // p 现在为空
    if (p) {
        std::cout << "p is not null" << std::endl;
    } else {
        std::cout << "p is null" << std::endl;
    }
    
    // 自动释放，无需手动 delete
}
```

### std::shared_ptr

共享所有权的智能指针，使用引用计数管理对象生命周期。

```cpp
void sharedPtrExample() {
    std::shared_ptr<int> p = std::make_shared<int>(42);
    std::cout << "count: " << p.use_count() << std::endl;  // 1
    
    {
        std::shared_ptr<int> q = p;
        std::cout << "count: " << p.use_count() << std::endl;  // 2
    }
    
    std::cout << "count: " << p.use_count() << std::endl;  // 1
    // 离开作用域自动释放
}
```

### std::weak_ptr

弱引用智能指针，不增加引用计数，用于解决 shared_ptr 的循环引用问题。

```cpp
struct Node {
    std::shared_ptr<Node> next;
    std::weak_ptr<Node> prev;  // 使用 weak_ptr 避免循环引用
};

void weakPtrExample() {
    auto node1 = std::make_shared<Node>();
    auto node2 = std::make_shared<Node>();
    
    node1->next = node2;
    node2->prev = node1;  // 不会增加 node1 的引用计数
}
```

---

## 4. 内存池

内存池是一种预先分配一块大内存，然后在其上进行小块内存分配的技术，减少内存碎片和分配开销。

### 简单内存池实现

```cpp
class MemoryPool {
private:
    char* pool;
    char* current;
    size_t totalSize;
    
public:
    MemoryPool(size_t size) : totalSize(size) {
        pool = new char[size];
        current = pool;
    }
    
    ~MemoryPool() {
        delete[] pool;
    }
    
    void* allocate(size_t size) {
        if (current + size > pool + totalSize) {
            return nullptr;
        }
        
        void* ptr = current;
        current += size;
        return ptr;
    }
    
    void deallocate(void* ptr) {
        // 简单实现，不实际释放
    }
};
```

---

## 5. RAII 原则

RAII（Resource Acquisition Is Initialization）是 C++ 的核心编程原则，利用对象的生命周期管理资源。

### 示例：文件管理

```cpp
class FileHandler {
private:
    FILE* file;
    
public:
    FileHandler(const char* filename, const char* mode) {
        file = fopen(filename, mode);
        if (!file) {
            throw std::runtime_error("Failed to open file");
        }
    }
    
    ~FileHandler() {
        if (file) {
            fclose(file);
        }
    }
    
    // 禁止拷贝
    FileHandler(const FileHandler&) = delete;
    FileHandler& operator=(const FileHandler&) = delete;
    
    FILE* get() const { return file; }
};

void useFile() {
    FileHandler fh("example.txt", "w");
    // 文件自动在作用域结束时关闭
}
```

---

## 常见面试题

### 1. 栈和堆的区别？

**答案：**
| 特性 | 栈 | 堆 |
|------|----|----|
| 分配方式 | 自动 | 手动 |
| 大小 | 固定且较小 | 动态且较大 |
| 效率 | 高 | 较低 |
| 生命周期 | 函数调用期间 | 手动控制 |
| 内存碎片 | 无 | 可能产生 |

### 2. 智能指针有哪些种类？区别是什么？

**答案：**
- **unique_ptr**：独占所有权，不可拷贝，可移动
- **shared_ptr**：共享所有权，使用引用计数
- **weak_ptr**：弱引用，不增加引用计数，解决循环引用

### 3. 什么是 RAII？

**答案：**
RAII（Resource Acquisition Is Initialization）是一种利用对象生命周期管理资源的技术。资源在构造函数中获取，在析构函数中释放，确保资源在任何情况下都能正确释放。

### 4. 如何避免内存泄漏？

**答案：**
- 使用智能指针代替裸指针
- 遵循 RAII 原则
- 使用内存池减少碎片
- 使用工具检测（如 Valgrind、AddressSanitizer）

### 5. 什么是内存碎片？如何减少？

**答案：**
内存碎片是指堆内存中存在大量不可用的小块内存。减少方法：
- 使用内存池
- 按大小分类分配
- 使用定制分配器

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
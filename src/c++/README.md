# C++

&emsp;&emsp;C++ 是一种高级通用编程语言，由 Bjarne Stroustrup 于 1985 年在贝尔实验室创建。它是 C 语言的扩展，支持面向对象、泛型、函数式等多种编程范式。C++ 以其高性能、灵活性和广泛的应用领域而闻名，是系统编程、游戏开发、嵌入式系统等领域的首选语言。

**语言特色：**
+ 高性能：接近汇编语言的执行效率
+ 多范式支持：面向对象、泛型、过程式、函数式
+ 内存控制：手动内存管理，提供细粒度控制
+ 广泛应用：系统级编程、游戏引擎、高性能计算
  
## 核心特性

### 面向对象编程
C++ 支持封装、继承和多态三大特性，通过类和对象实现代码的组织和复用。

### 泛型编程
通过模板（Template）实现类型无关的通用代码，是 STL（标准模板库）的基础。

### 内存管理
提供手动内存管理（new/delete）和智能指针（unique_ptr、shared_ptr）等多种内存管理方式。

### 高性能
零运行时开销（Zero-overhead principle），编译器优化能力强，适合高性能场景。

## 目录

| 文档 | 内容简介 |
|------|---------|
| [内存管理](memory.md) | 堆内存分配、栈内存、智能指针、内存池 |
| [面向对象](oop.md) | 类与对象、继承、多态、虚函数 |
| [模板编程](template.md) | 函数模板、类模板、特化、SFINAE |
| [STL详解](stl.md) | 容器、算法、迭代器、函数对象 |
| [智能指针](smart_ptr.md) | unique_ptr、shared_ptr、weak_ptr、auto_ptr |
| [Boost库](boost.md) | Boost库介绍、常用组件、使用场景 |
| [并发编程](concurrency.md) | 线程、互斥锁、条件变量、原子操作 |
| [设计模式](design_pattern.md) | 常用设计模式的C++实现 |

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
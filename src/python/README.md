# Python

&emsp;&emsp;Python 是一种高级、通用、解释型编程语言，由 Guido van Rossum 于 1991 年发布。Python 以其简洁的语法、强大的库生态和广泛的应用领域而闻名，是数据科学、机器学习、Web 开发等领域的首选语言。

**语言特色：**
+ 简洁优雅：代码可读性强，语法简洁
+ 动态类型：无需显式声明变量类型
+ 面向对象：支持面向对象编程
+ 丰富的库：拥有庞大的标准库和第三方库
+ 跨平台：支持多种操作系统
  
## 核心特性

### 动态类型系统
Python 是动态类型语言，变量的类型在运行时确定，提供了更大的灵活性。

### 自动内存管理
Python 使用引用计数和垃圾回收机制自动管理内存，开发者无需手动分配和释放内存。

### 面向对象
Python 支持面向对象编程，所有东西都是对象，支持封装、继承和多态。

### 函数式编程
Python 支持函数式编程特性，如 lambda 表达式、高阶函数等。

## 目录

| 文档 | 内容简介 |
|------|---------|
| [基础语法](basic.md) | 数据类型、运算符、流程控制、异常处理 |
| [面向对象](oop.md) | 类与对象、继承、多态、魔法方法 |
| [装饰器](decorator.md) | 装饰器原理、应用场景、常用装饰器 |
| [生成器与迭代器](generator.md) | 迭代器协议、生成器、yield 关键字 |
| [并发编程](concurrency.md) | 多线程、多进程、协程、异步IO |
| [内存管理](memory.md) | 引用计数、垃圾回收、内存优化 |
| [标准库](stdlib.md) | 常用标准库介绍和使用示例 |
| [第三方库](thirdparty.md) | NumPy、Pandas、TensorFlow、Django等 |
| [设计模式](design_pattern.md) | 常用设计模式的Python实现 |
| [元编程](metaprogramming.md) | 元类、反射、动态代码生成 |

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
# golang

&emsp;&emsp;Go是一个开源的编程语言，它能让构造简单、可靠且高效的软件变得容易。Go是从2007年末由Robert Griesemer, Rob Pike, Ken Thompson主持开发，后来还加入了Ian Lance Taylor, Russ Cox等人，并最终于2009年11月开源，在2012年早些时候发布了Go 1稳定版本。现在Go的开发已经是完全开放的，并且拥有一个活跃的社区。
  
**语言特色：**
+ 简洁、快速、安全
+ 并行、有趣、开源
+ 内存管理、数组安全、编译迅速  

## 核心特性

### 并发模型
Go 语言内置了轻量级线程（Goroutine）和通道（Channel），提供了简洁而强大的并发编程模型。

### 内存管理
Go 采用自动垃圾回收机制，开发者无需手动管理内存，减少了内存泄漏和野指针问题。

### 类型系统
静态类型语言，支持接口、泛型等特性，提供了类型安全的同时保持了灵活性。

### 编译速度
Go 的编译速度非常快，支持交叉编译，可以轻松构建跨平台应用。

## 目录

| 文档 | 内容简介 |
|------|---------|
| [slice底层原理](slice.md) | 切片的底层实现、扩容机制、内存布局 |
| [map底层原理](map.md) | 哈希表实现、冲突解决、扩容策略 |
| [并发与channel](channel.md) | 通道原理、并发模式、生产者消费者模式 |
| [反射机制](reflect.md) | 反射原理、类型获取、值操作 |
| [context详解](context.md) | 上下文传递、取消机制、超时控制 |
| [GMP模型](gmp.md) | Goroutine、M、P调度模型、工作窃取 |
| [垃圾回收机制(GC)](gc.md) | 三色标记、写屏障、STW优化 |
| [module管理](module.md) | Go Module、依赖管理、版本控制 |
| [单元测试](ut.md) | testing框架、gomock、gomonkey、sqlmock、GoConvey、Wire依赖注入 |

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
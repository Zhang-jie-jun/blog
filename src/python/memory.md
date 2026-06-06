# Python 内存管理

## 内存管理简介

Python 使用引用计数和垃圾回收机制自动管理内存，开发者无需手动分配和释放内存。

---

## 1. 引用计数

### 基本原理

```python
import sys

# 创建对象，引用计数为1
a = [1, 2, 3]
print(sys.getrefcount(a))  # 2（getrefcount 会临时增加引用）

# 增加引用
b = a
print(sys.getrefcount(a))  # 3

# 减少引用
del b
print(sys.getrefcount(a))  # 2

# 引用为0时自动释放
del a
```

### 引用计数的问题

```python
# 循环引用问题
class Node:
    def __init__(self):
        self.next = None

# 创建循环引用
node1 = Node()
node2 = Node()
node1.next = node2
node2.next = node1

# 删除引用，但循环引用导致引用计数不为0
del node1
del node2
# 内存泄漏（需要垃圾回收器处理）
```

---

## 2. 垃圾回收

### 触发条件

```python
import gc

# 手动触发垃圾回收
gc.collect()

# 查看垃圾回收统计
print(gc.get_stats())

# 设置垃圾回收阈值
gc.set_threshold(700, 10, 10)
```

### 弱引用

```python
import weakref

class MyClass:
    def __del__(self):
        print("Object destroyed")

obj = MyClass()
weak = weakref.ref(obj)

print(weak())  # <__main__.MyClass object at ...>

del obj
print(weak())  # None（对象已被销毁）
```

---

## 3. 内存优化

### 列表 vs 生成器

```python
# 列表：立即生成所有元素
numbers = [x for x in range(1000000)]  # 占用大量内存

# 生成器：按需生成
numbers_gen = (x for x in range(1000000))  # 几乎不占用内存
```

### __slots__

```python
# 默认方式：使用字典存储属性
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

# 使用 __slots__ 减少内存
class PersonOptimized:
    __slots__ = ['name', 'age']
    def __init__(self, name, age):
        self.name = name
        self.age = age
```

### 内存视图

```python
import array

# 创建数组
arr = array.array('i', [1, 2, 3, 4, 5])

# 创建内存视图（共享底层数据）
view = memoryview(arr)
view[0] = 100
print(arr)  # array('i', [100, 2, 3, 4, 5])
```

---

## 4. 内存调试

### tracemalloc

```python
import tracemalloc

# 开始追踪
tracemalloc.start()

# 执行代码
data = [i for i in range(10000)]

# 获取内存快照
snapshot = tracemalloc.take_snapshot()
top_stats = snapshot.statistics('lineno')

print("[Top 10 memory usage]")
for stat in top_stats[:10]:
    print(stat)
```

### objgraph

```python
# 需要安装：pip install objgraph
import objgraph

# 找出内存泄漏
objgraph.show_most_common_types(limit=10)

# 追踪对象引用
objgraph.show_backrefs([obj], filename='backrefs.png')
```

---

## 常见面试题

### 1. Python 的内存管理机制是什么？

**答案：**
Python 使用引用计数为主，垃圾回收为辅的内存管理机制。

### 2. 什么是循环引用？如何解决？

**答案：**
循环引用是指两个或多个对象相互引用，导致引用计数无法归零。Python 的垃圾回收器会定期检测并清理循环引用。

### 3. __slots__ 的作用是什么？

**答案：**
`__slots__` 限制类实例的属性，使用固定大小的数组代替字典，减少内存占用。

### 4. 弱引用的用途？

**答案：**
弱引用不增加引用计数，用于打破循环引用，常用于缓存场景。

### 5. 如何检测内存泄漏？

**答案：**
使用 `tracemalloc` 模块或 `objgraph` 库检测内存使用情况。

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
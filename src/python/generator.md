# Python 生成器与迭代器

## 生成器与迭代器简介

迭代器和生成器是 Python 中处理序列数据的重要工具，支持惰性求值，节省内存。

---

## 1. 迭代器

### 迭代器协议

```python
class MyIterator:
    def __init__(self, data):
        self.data = data
        self.index = 0
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.index >= len(self.data):
            raise StopIteration
        value = self.data[self.index]
        self.index += 1
        return value

# 使用迭代器
nums = MyIterator([1, 2, 3])
for num in nums:
    print(num)
```

### 内置迭代器

```python
# 列表迭代
for item in [1, 2, 3]:
    print(item)

# 字符串迭代
for char in "hello":
    print(char)

# 文件迭代
with open("example.txt") as f:
    for line in f:
        print(line)
```

---

## 2. 生成器

### 生成器函数

```python
def countdown(n):
    while n > 0:
        yield n
        n -= 1

# 使用生成器
for num in countdown(5):
    print(num)
# 输出: 5 4 3 2 1
```

### 生成器表达式

```python
# 生成器表达式
gen = (x ** 2 for x in range(5))
for num in gen:
    print(num)
# 输出: 0 1 4 9 16

# 对比列表推导式
lst = [x ** 2 for x in range(5)]  # 立即生成列表
```

### 无限生成器

```python
def fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

# 取前10个斐波那契数
fib = fibonacci()
for _ in range(10):
    print(next(fib))
```

---

## 3. yield from

```python
def gen1():
    yield 1
    yield 2

def gen2():
    yield from gen1()  # 委托给另一个生成器
    yield 3
    yield 4

for num in gen2():
    print(num)
# 输出: 1 2 3 4
```

---

## 4. 迭代工具函数

### itertools 模块

```python
import itertools

# 无限迭代器
count = itertools.count(1, 2)  # 1, 3, 5, 7...
cycle = itertools.cycle([1, 2, 3])  # 1, 2, 3, 1, 2, 3...
repeat = itertools.repeat(5, 3)  # 5, 5, 5

# 有限迭代器
chain = itertools.chain([1, 2], [3, 4])  # 1, 2, 3, 4
groupby = itertools.groupby("AABBC")  # 按连续相同元素分组

# 组合迭代器
permutations = itertools.permutations([1, 2, 3], 2)  # 排列
combinations = itertools.combinations([1, 2, 3], 2)  # 组合
```

---

## 5. 实战应用

### 处理大文件

```python
def read_large_file(filepath):
    """逐行读取大文件"""
    with open(filepath, 'r') as f:
        for line in f:
            yield line.strip()

# 使用生成器处理大文件
for line in read_large_file("large_file.txt"):
    process(line)
```

### 管道处理

```python
def filter_even(numbers):
    for num in numbers:
        if num % 2 == 0:
            yield num

def square(numbers):
    for num in numbers:
        yield num ** 2

def sum_values(numbers):
    total = 0
    for num in numbers:
        total += num
    return total

# 管道处理
data = [1, 2, 3, 4, 5, 6]
result = sum_values(square(filter_even(data)))
print(result)  # 56
```

---

## 常见面试题

### 1. 迭代器和生成器的区别？

**答案：**
- **迭代器**：实现了 `__iter__` 和 `__next__` 方法的对象
- **生成器**：使用 yield 关键字的函数，自动实现迭代器协议

### 2. 什么是惰性求值？

**答案：**
惰性求值是指在需要时才计算值，而不是提前计算所有值。生成器支持惰性求值，节省内存。

### 3. yield 和 return 的区别？

**答案：**
- `return`：返回值并终止函数
- `yield`：返回值但暂停函数，下次调用继续执行

### 4. 如何创建无限序列？

**答案：**
使用生成器函数配合 while True 循环，例如：

```python
def infinite_sequence():
    num = 0
    while True:
        yield num
        num += 1
```

### 5. itertools 模块有哪些常用函数？

**答案：**
- `count()`：无限计数器
- `cycle()`：无限循环序列
- `chain()`：连接多个迭代器
- `permutations()`：排列
- `combinations()`：组合

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
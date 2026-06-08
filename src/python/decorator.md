# Python 装饰器

## 装饰器简介

装饰器是 Python 中一种强大的功能，用于修改或增强函数和方法的行为。

---

## 1. 装饰器基础

### 简单装饰器

```python
def simple_decorator(func):
    def wrapper():
        print("Before function call")
        func()
        print("After function call")
    return wrapper

@simple_decorator
def greet():
    print("Hello, World!")

greet()
# 输出:
# Before function call
# Hello, World!
# After function call
```

### 带参数的装饰器

```python
def decorator_with_args(arg1, arg2):
    def decorator(func):
        def wrapper(*args, **kwargs):
            print(f"Decorator args: {arg1}, {arg2}")
            return func(*args, **kwargs)
        return wrapper
    return decorator

@decorator_with_args("hello", "world")
def add(a, b):
    return a + b

result = add(3, 5)
# 输出: Decorator args: hello, world
# result = 8
```

---

## 2. 常用内置装饰器

### @staticmethod

```python
class Math:
    @staticmethod
    def add(a, b):
        return a + b

print(Math.add(3, 5))  # 8（无需创建实例）
```

### @classmethod

```python
class Person:
    species = "Homo sapiens"
    
    @classmethod
    def get_species(cls):
        return cls.species

print(Person.get_species())  # Homo sapiens
```

### @property

```python
class Circle:
    def __init__(self, radius):
        self._radius = radius
    
    @property
    def radius(self):
        return self._radius
    
    @radius.setter
    def radius(self, value):
        if value < 0:
            raise ValueError("Radius cannot be negative")
        self._radius = value
```

---

## 3. 实用装饰器示例

### 日志装饰器

```python
import logging

def log_decorator(func):
    logging.basicConfig(level=logging.INFO)
    
    def wrapper(*args, **kwargs):
        logging.info(f"Calling {func.__name__} with args={args}, kwargs={kwargs}")
        result = func(*args, **kwargs)
        logging.info(f"{func.__name__} returned {result}")
        return result
    return wrapper

@log_decorator
def multiply(a, b):
    return a * b

multiply(3, 4)
# 输出:
# INFO:root:Calling multiply with args=(3, 4), kwargs={}
# INFO:root:multiply returned 12
```

### 计时装饰器

```python
import time

def timer_decorator(func):
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} took {end - start:.4f} seconds")
        return result
    return wrapper

@timer_decorator
def slow_function():
    time.sleep(1)
    return "Done"

slow_function()
# 输出: slow_function took 1.0005 seconds
```

### 缓存装饰器

```python
def memoize(func):
    cache = {}
    
    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
    return wrapper

@memoize
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

print(fibonacci(10))  # 55（只计算一次）
```

---

## 4. 装饰器类

```python
class DecoratorClass:
    def __init__(self, func):
        self.func = func
    
    def __call__(self, *args, **kwargs):
        print("Before")
        result = self.func(*args, **kwargs)
        print("After")
        return result

@DecoratorClass
def hello(name):
    return f"Hello, {name}!"

print(hello("Alice"))
# 输出:
# Before
# Hello, Alice!
# After
```

---

## 5. 保留函数元信息

```python
from functools import wraps

def decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        """Wrapper docstring"""
        return func(*args, **kwargs)
    return wrapper

@decorator
def original():
    """Original docstring"""
    pass

print(original.__name__)    # original（不是 wrapper）
print(original.__doc__)     # Original docstring（不是 Wrapper docstring）
```

---

## 常见面试题

### 1. 装饰器的作用是什么？

**答案：**
装饰器用于修改或增强函数和方法的行为，在不修改原函数代码的情况下添加额外功能。

### 2. 装饰器是如何实现的？

**答案：**
装饰器本质上是一个接受函数作为参数并返回新函数的函数。使用 @ 语法糖应用装饰器。

### 3. 如何编写带参数的装饰器？

**答案：**
需要三层嵌套：最外层接受装饰器参数，中间层接受被装饰函数，最内层是包装函数。

### 4. @staticmethod 和 @classmethod 的区别？

**答案：**
- `@staticmethod`：不需要访问类或实例
- `@classmethod`：需要访问类，可以通过 cls 参数

### 5. 装饰器会影响函数的元信息吗？

**答案：**
默认会影响。使用 `functools.wraps` 装饰器可以保留原函数的元信息。

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

# Python 基础语法

## 基础语法简介

Python 的语法简洁优雅，注重可读性。本章节介绍 Python 的基本语法元素。

---

## 1. 数据类型

### 基本类型

```python
# 整数
age = 25
print(type(age))  # <class 'int'>

# 浮点数
pi = 3.14159
print(type(pi))   # <class 'float'>

# 布尔值
is_active = True
print(type(is_active))  # <class 'bool'>

# 字符串
name = "Hello World"
print(type(name))  # <class 'str'>

# None
result = None
print(type(result))  # <class 'NoneType'>
```

### 容器类型

```python
# 列表（可变）
fruits = ['apple', 'banana', 'cherry']
fruits.append('date')
print(fruits)  # ['apple', 'banana', 'cherry', 'date']

# 元组（不可变）
point = (3, 4)
print(point[0])  # 3

# 字典
person = {'name': 'Alice', 'age': 30}
print(person['name'])  # Alice

# 集合
unique_numbers = {1, 2, 3, 3, 4}
print(unique_numbers)  # {1, 2, 3, 4}
```

---

## 2. 运算符

### 算术运算符

```python
a = 10
b = 3

print(a + b)   # 13
print(a - b)   # 7
print(a * b)   # 30
print(a / b)   # 3.333...
print(a // b)  # 3（整数除法）
print(a % b)   # 1（取模）
print(a ** b)  # 1000（幂运算）
```

### 比较运算符

```python
x = 5
y = 3

print(x == y)  # False
print(x != y)  # True
print(x > y)   # True
print(x < y)   # False
print(x >= y)  # True
print(x <= y)  # False
```

### 逻辑运算符

```python
p = True
q = False

print(p and q)  # False
print(p or q)   # True
print(not p)    # False
```

---

## 3. 流程控制

### 条件语句

```python
score = 85

if score >= 90:
    grade = 'A'
elif score >= 80:
    grade = 'B'
elif score >= 70:
    grade = 'C'
else:
    grade = 'D'

print(f"Grade: {grade}")
```

### 循环语句

```python
# for 循环
for i in range(5):
    print(i)

# 遍历列表
fruits = ['apple', 'banana', 'cherry']
for fruit in fruits:
    print(fruit)

# while 循环
count = 0
while count < 5:
    print(count)
    count += 1

# break 和 continue
for i in range(10):
    if i == 3:
        continue  # 跳过当前迭代
    if i == 7:
        break     # 终止循环
    print(i)
```

---

## 4. 函数

### 定义和调用

```python
def greet(name):
    """返回问候语"""
    return f"Hello, {name}!"

print(greet("Alice"))  # Hello, Alice!
```

### 参数传递

```python
def add(a, b=0):
    return a + b

print(add(3, 5))  # 8
print(add(3))     # 3（使用默认参数）

# 可变参数
def sum_all(*args):
    total = 0
    for num in args:
        total += num
    return total

print(sum_all(1, 2, 3, 4))  # 10

# 关键字参数
def introduce(**kwargs):
    for key, value in kwargs.items():
        print(f"{key}: {value}")

introduce(name="Bob", age=25, city="New York")
```

### Lambda 表达式

```python
add = lambda x, y: x + y
print(add(3, 5))  # 8

# 排序时使用
students = [('Alice', 25), ('Bob', 20), ('Charlie', 30)]
students.sort(key=lambda x: x[1])
print(students)  # [('Bob', 20), ('Alice', 25), ('Charlie', 30)]
```

---

## 5. 异常处理

### try-except 语句

```python
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero")
except Exception as e:
    print(f"An error occurred: {e}")
else:
    print(f"Result: {result}")
finally:
    print("This always executes")
```

### 自定义异常

```python
class InvalidAgeError(Exception):
    pass

def check_age(age):
    if age < 0:
        raise InvalidAgeError("Age cannot be negative")
    print(f"Valid age: {age}")

try:
    check_age(-5)
except InvalidAgeError as e:
    print(e)  # Age cannot be negative
```

---

## 6. 文件操作

### 读取文件

```python
# 方式一：手动关闭
file = open('example.txt', 'r')
content = file.read()
file.close()

# 方式二：with 语句（自动关闭）
with open('example.txt', 'r') as f:
    content = f.read()

# 逐行读取
with open('example.txt', 'r') as f:
    for line in f:
        print(line.strip())
```

### 写入文件

```python
# 覆盖写入
with open('output.txt', 'w') as f:
    f.write("Hello, World!")

# 追加写入
with open('output.txt', 'a') as f:
    f.write("\nAppended text")
```

---

## 常见面试题

### 1. Python 中的可变和不可变类型有哪些？

**答案：**
- **不可变类型**：int、float、str、tuple、frozenset
- **可变类型**：list、dict、set

### 2. == 和 is 的区别？

**答案：**
- `==` 比较值是否相等
- `is` 比较对象是否是同一个实例（比较内存地址）

### 3. Python 中的 pass 关键字有什么作用？

**答案：**
pass 是一个空操作语句，用于占位，保持语法完整。

### 4. 什么是列表推导式？

**答案：**
列表推导式是一种简洁的创建列表的方式：
```python
squares = [x**2 for x in range(10)]
```

### 5. Python 中的缩进有什么作用？

**答案：**
Python 使用缩进来表示代码块，替代了其他语言的花括号。

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

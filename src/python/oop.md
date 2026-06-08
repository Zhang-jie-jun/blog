# Python 面向对象

## 面向对象简介

Python 是一种面向对象的编程语言，一切皆对象。本章节介绍 Python 的面向对象编程特性。

---

## 1. 类与对象

### 定义类

```python
class Person:
    # 类属性
    species = "Homo sapiens"
    
    # 构造方法
    def __init__(self, name, age):
        # 实例属性
        self.name = name
        self.age = age
    
    # 实例方法
    def introduce(self):
        return f"My name is {self.name}, I'm {self.age} years old."
    
    # 类方法
    @classmethod
    def get_species(cls):
        return cls.species
    
    # 静态方法
    @staticmethod
    def is_adult(age):
        return age >= 18

# 创建对象
person = Person("Alice", 25)
print(person.introduce())  # My name is Alice, I'm 25 years old.
```

### 访问控制

```python
class Car:
    def __init__(self):
        self.__engine = "V8"  # 私有属性
        self._color = "red"   # 保护属性
    
    def get_engine(self):
        return self.__engine

car = Car()
print(car._color)        # red（可以访问，但不推荐）
# print(car.__engine)    # 错误：AttributeError
print(car.get_engine())  # V8
```

---

## 2. 继承

### 单继承

```python
class Animal:
    def __init__(self, name):
        self.name = name
    
    def speak(self):
        pass

class Dog(Animal):
    def speak(self):
        return f"{self.name} says Woof!"

class Cat(Animal):
    def speak(self):
        return f"{self.name} says Meow!"

dog = Dog("Buddy")
cat = Cat("Whiskers")
print(dog.speak())  # Buddy says Woof!
print(cat.speak())  # Whiskers says Meow!
```

### 多重继承

```python
class Flyable:
    def fly(self):
        return "I can fly!"

class Swimmable:
    def swim(self):
        return "I can swim!"

class Duck(Flyable, Swimmable):
    pass

duck = Duck()
print(duck.fly())   # I can fly!
print(duck.swim())  # I can swim!
```

---

## 3. 多态

### 方法重写

```python
class Shape:
    def area(self):
        pass

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius
    
    def area(self):
        return 3.14159 * self.radius ** 2

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    def area(self):
        return self.width * self.height

# 多态：统一接口，不同实现
shapes = [Circle(5), Rectangle(4, 6)]
for shape in shapes:
    print(f"Area: {shape.area()}")
```

---

## 4. 魔法方法

### 常用魔法方法

```python
class Vector:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    # 加法
    def __add__(self, other):
        return Vector(self.x + other.x, self.y + other.y)
    
    # 减法
    def __sub__(self, other):
        return Vector(self.x - other.x, self.y - other.y)
    
    # 字符串表示
    def __str__(self):
        return f"Vector({self.x}, {self.y})"
    
    # 长度
    def __len__(self):
        return int((self.x ** 2 + self.y ** 2) ** 0.5)
    
    # 索引访问
    def __getitem__(self, key):
        if key == 0:
            return self.x
        elif key == 1:
            return self.y
        raise IndexError("Invalid index")

v1 = Vector(3, 4)
v2 = Vector(1, 2)
print(v1 + v2)  # Vector(4, 6)
print(len(v1))  # 5
print(v1[0])    # 3
```

### 上下文管理器

```python
class FileHandler:
    def __init__(self, filename, mode):
        self.filename = filename
        self.mode = mode
    
    def __enter__(self):
        self.file = open(self.filename, self.mode)
        return self.file
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.file.close()

# 使用 with 语句
with FileHandler("example.txt", "r") as f:
    content = f.read()
```

---

## 5. 属性装饰器

### property 装饰器

```python
class Person:
    def __init__(self, name):
        self._name = name
    
    @property
    def name(self):
        return self._name
    
    @name.setter
    def name(self, value):
        if not isinstance(value, str):
            raise TypeError("Name must be a string")
        self._name = value

person = Person("Alice")
print(person.name)    # Alice
person.name = "Bob"   # 设置值
# person.name = 123   # TypeError
```

---

## 常见面试题

### 1. __init__ 和 __new__ 的区别？

**答案：**
- `__new__`：创建对象时调用，返回对象实例
- `__init__`：初始化对象时调用，设置对象属性

### 2. 什么是鸭子类型？

**答案：**
鸭子类型是一种动态类型的风格，关注对象的行为而非类型。只要对象实现了特定方法，就可以使用。

### 3. Python 支持多重继承吗？

**答案：**
是的，Python 支持多重继承。使用 `super()` 函数调用父类方法。

### 4. 什么是魔法方法？

**答案：**
魔法方法是 Python 中以双下划线开头和结尾的方法，如 `__init__`、`__str__`、`__add__` 等，用于实现运算符重载和特殊行为。

### 5. @property 装饰器的作用？

**答案：**
@property 装饰器将方法转换为属性，允许通过属性访问语法调用方法，同时可以控制属性的读取和设置。

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
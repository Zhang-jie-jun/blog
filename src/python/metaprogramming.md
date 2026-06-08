# Python 元编程

## 元编程简介

元编程是指编写能够操作代码的代码，在 Python 中可以通过元类、反射等技术实现。

---

## 1. 元类

### 什么是元类

```python
# type 是所有类的元类
class MyClass:
    pass

print(type(MyClass))  # <class 'type'>
print(type(int))      # <class 'type'>
```

### 自定义元类

```python
class MyMeta(type):
    def __new__(cls, name, bases, attrs):
        # 修改类属性
        attrs['author'] = 'John'
        return super().__new__(cls, name, bases, attrs)

class MyClass(metaclass=MyMeta):
    pass

print(MyClass.author)  # John
```

### 元类应用：注册模式

```python
class RegistryMeta(type):
    registry = {}
    
    def __new__(cls, name, bases, attrs):
        new_class = super().__new__(cls, name, bases, attrs)
        cls.registry[name] = new_class
        return new_class

class BasePlugin(metaclass=RegistryMeta):
    pass

class PluginA(BasePlugin):
    pass

class PluginB(BasePlugin):
    pass

print(RegistryMeta.registry)
# {'BasePlugin': <class '__main__.BasePlugin'>, 
#  'PluginA': <class '__main__.PluginA'>, 
#  'PluginB': <class '__main__.PluginB'>}
```

---

## 2. 反射

### 获取对象信息

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
    
    def greet(self):
        return f"Hello, {self.name}"

person = Person("Alice", 30)

# 获取属性
print(getattr(person, 'name'))  # Alice

# 设置属性
setattr(person, 'age', 31)
print(person.age)  # 31

# 检查属性是否存在
print(hasattr(person, 'greet'))  # True

# 调用方法
method = getattr(person, 'greet')
print(method())  # Hello, Alice
```

### 动态导入模块

```python
import importlib

# 动态导入模块
math_module = importlib.import_module('math')
print(math_module.sqrt(16))  # 4.0

# 动态导入类
module = importlib.import_module('my_module')
MyClass = getattr(module, 'MyClass')
obj = MyClass()
```

---

## 3. 动态代码生成

### exec 和 eval

```python
# exec：执行代码块
code = """
def hello(name):
    return f"Hello, {name}!"
"""
exec(code)
print(hello("World"))  # Hello, World!

# eval：计算表达式
result = eval("2 + 3 * 4")
print(result)  # 14
```

### compile 函数

```python
# 编译代码
code = compile("print('Hello')", '<string>', 'exec')
exec(code)  # Hello

# 编译表达式
expr = compile("x + y", '<string>', 'eval')
x, y = 3, 5
result = eval(expr)
print(result)  # 8
```

---

## 4. 装饰器进阶

### 类装饰器

```python
class Decorator:
    def __init__(self, func):
        self.func = func
    
    def __call__(self, *args, **kwargs):
        print("Before")
        result = self.func(*args, **kwargs)
        print("After")
        return result

@Decorator
def greet(name):
    return f"Hello, {name}!"

print(greet("Alice"))
# Before
# Hello, Alice!
# After
```

### 装饰器工厂

```python
def decorator_factory(arg):
    def decorator(func):
        def wrapper(*args, **kwargs):
            print(f"Decorator arg: {arg}")
            return func(*args, **kwargs)
        return wrapper
    return decorator

@decorator_factory("hello")
def func():
    print("Function")

func()
# Decorator arg: hello
# Function
```

---

## 5. 描述符

### 描述符协议

```python
class Descriptor:
    def __get__(self, instance, owner):
        return self.value
    
    def __set__(self, instance, value):
        self.value = value
    
    def __delete__(self, instance):
        del self.value

class MyClass:
    attr = Descriptor()

obj = MyClass()
obj.attr = 42
print(obj.attr)  # 42
```

### 应用：类型检查

```python
class Typed:
    def __init__(self, type_):
        self.type = type_
    
    def __get__(self, instance, owner):
        return instance.__dict__.get(self.name)
    
    def __set__(self, instance, value):
        if not isinstance(value, self.type):
            raise TypeError(f"Expected {self.type}")
        instance.__dict__[self.name] = value
    
    def __set_name__(self, owner, name):
        self.name = name

class Person:
    name = Typed(str)
    age = Typed(int)

person = Person()
person.name = "Alice"
person.age = 30
# person.age = "30"  # TypeError: Expected <class 'int'>
```

---

## 常见面试题

### 1. 什么是元类？

**答案：**
元类是创建类的类，type 是 Python 中最基础的元类。

### 2. 反射是什么？

**答案：**
反射是指在运行时检查和修改对象的属性和方法。

### 3. exec、eval、compile 的区别？

**答案：**
- `exec`：执行代码块，无返回值
- `eval`：计算表达式，返回结果
- `compile`：编译代码为代码对象

### 4. 描述符协议是什么？

**答案：**
描述符协议包含 `__get__`、`__set__`、`__delete__` 方法，用于控制属性访问。

### 5. 元编程的应用场景？

**答案：**
- ORM 框架
- 依赖注入
- 代码生成
- 装饰器和中间件

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
# Python 设计模式

## 设计模式简介

设计模式是解决特定问题的成熟方案，分为创建型、结构型和行为型三大类。

---

## 1. 创建型模式

### 单例模式

```python
class Singleton:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

# 使用
s1 = Singleton()
s2 = Singleton()
print(s1 is s2)  # True
```

### 工厂模式

```python
class Product:
    def use(self):
        pass

class ConcreteProductA(Product):
    def use(self):
        print("Using Product A")

class ConcreteProductB(Product):
    def use(self):
        print("Using Product B")

class Factory:
    def create_product(self, type_):
        if type_ == 'A':
            return ConcreteProductA()
        elif type_ == 'B':
            return ConcreteProductB()

# 使用
factory = Factory()
product = factory.create_product('A')
product.use()  # Using Product A
```

### 建造者模式

```python
class Product:
    def __init__(self):
        self.parts = []
    
    def add_part(self, part):
        self.parts.append(part)
    
    def show(self):
        print(f"Product parts: {self.parts}")

class Builder:
    def build_part_a(self):
        pass
    
    def build_part_b(self):
        pass
    
    def get_product(self):
        pass

class ConcreteBuilder(Builder):
    def __init__(self):
        self.product = Product()
    
    def build_part_a(self):
        self.product.add_part("Part A")
    
    def build_part_b(self):
        self.product.add_part("Part B")
    
    def get_product(self):
        return self.product

class Director:
    def construct(self, builder):
        builder.build_part_a()
        builder.build_part_b()

# 使用
builder = ConcreteBuilder()
director = Director()
director.construct(builder)
product = builder.get_product()
product.show()  # Product parts: ['Part A', 'Part B']
```

---

## 2. 结构型模式

### 适配器模式

```python
class Target:
    def request(self):
        pass

class Adaptee:
    def specific_request(self):
        return "Specific request"

class Adapter(Target):
    def __init__(self, adaptee):
        self.adaptee = adaptee
    
    def request(self):
        return self.adaptee.specific_request()

# 使用
adaptee = Adaptee()
adapter = Adapter(adaptee)
print(adapter.request())  # Specific request
```

### 装饰器模式

```python
class Component:
    def operation(self):
        pass

class ConcreteComponent(Component):
    def operation(self):
        return "Base operation"

class Decorator(Component):
    def __init__(self, component):
        self.component = component
    
    def operation(self):
        return self.component.operation()

class ConcreteDecorator(Decorator):
    def operation(self):
        return f"Decorated: {self.component.operation()}"

# 使用
component = ConcreteComponent()
decorated = ConcreteDecorator(component)
print(decorated.operation())  # Decorated: Base operation
```

### 代理模式

```python
class Subject:
    def request(self):
        pass

class RealSubject(Subject):
    def request(self):
        return "Real request"

class Proxy(Subject):
    def __init__(self):
        self.real_subject = None
    
    def request(self):
        if not self.real_subject:
            self.real_subject = RealSubject()
        return self.real_subject.request()

# 使用
proxy = Proxy()
print(proxy.request())  # Real request
```

---

## 3. 行为型模式

### 观察者模式

```python
class Observer:
    def update(self, message):
        pass

class Subject:
    def __init__(self):
        self.observers = []
    
    def attach(self, observer):
        self.observers.append(observer)
    
    def detach(self, observer):
        self.observers.remove(observer)
    
    def notify(self, message):
        for observer in self.observers:
            observer.update(message)

class ConcreteObserver(Observer):
    def __init__(self, name):
        self.name = name
    
    def update(self, message):
        print(f"{self.name} received: {message}")

# 使用
subject = Subject()
observer1 = ConcreteObserver("Observer 1")
observer2 = ConcreteObserver("Observer 2")

subject.attach(observer1)
subject.attach(observer2)
subject.notify("Hello World")
# Observer 1 received: Hello World
# Observer 2 received: Hello World
```

### 策略模式

```python
class Strategy:
    def execute(self, a, b):
        pass

class AddStrategy(Strategy):
    def execute(self, a, b):
        return a + b

class MultiplyStrategy(Strategy):
    def execute(self, a, b):
        return a * b

class Context:
    def __init__(self, strategy):
        self.strategy = strategy
    
    def set_strategy(self, strategy):
        self.strategy = strategy
    
    def compute(self, a, b):
        return self.strategy.execute(a, b)

# 使用
context = Context(AddStrategy())
print(context.compute(3, 5))  # 8

context.set_strategy(MultiplyStrategy())
print(context.compute(3, 5))  # 15
```

### 模板方法模式

```python
class AbstractClass:
    def template_method(self):
        self.step1()
        self.step2()
        self.step3()
    
    def step1(self):
        print("Step 1")
    
    def step2(self):
        pass
    
    def step3(self):
        print("Step 3")

class ConcreteClass(AbstractClass):
    def step2(self):
        print("Concrete Step 2")

# 使用
obj = ConcreteClass()
obj.template_method()
# Step 1
# Concrete Step 2
# Step 3
```

---

## 常见面试题

### 1. 什么是设计模式？

**答案：**
设计模式是解决特定问题的成熟、可复用的解决方案。

### 2. 单例模式的实现方式？

**答案：**
使用 `__new__` 方法控制实例创建，确保只有一个实例。

### 3. 装饰器模式和 Python 装饰器的区别？

**答案：**
设计模式中的装饰器模式是一种架构模式，而 Python 装饰器是语言特性，用于增强函数行为。

### 4. 观察者模式的应用场景？

**答案：**
GUI 事件处理、消息订阅系统、发布-订阅模式。

### 5. 策略模式的优点？

**答案：**
运行时切换算法，符合开闭原则，降低耦合。

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

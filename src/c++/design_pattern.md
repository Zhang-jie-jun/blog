# C++ 设计模式

## 设计模式简介

设计模式是解决特定问题的成熟方案，分为创建型、结构型和行为型三大类。

### 设计模式分类

| 类别 | 描述 | 模式 |
|------|------|------|
| **创建型** | 对象创建机制 | 单例、工厂、抽象工厂、建造者、原型 |
| **结构型** | 类和对象的组合 | 适配器、桥接、组合、装饰器、外观、享元、代理 |
| **行为型** | 对象间的通信 | 责任链、命令、解释器、迭代器、中介者、备忘录、观察者、状态、策略、模板方法、访问者 |

---

## 1. 创建型模式

### 单例模式

```cpp
class Singleton {
private:
    static Singleton* instance;
    Singleton() {}
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;
    
public:
    static Singleton* getInstance() {
        if (!instance) {
            instance = new Singleton();
        }
        return instance;
    }
};

Singleton* Singleton::instance = nullptr;
```

### 工厂模式

```cpp
class Product {
public:
    virtual void use() = 0;
    virtual ~Product() = default;
};

class ConcreteProductA : public Product {
public:
    void use() override {
        std::cout << "Using Product A" << std::endl;
    }
};

class ConcreteProductB : public Product {
public:
    void use() override {
        std::cout << "Using Product B" << std::endl;
    }
};

class Factory {
public:
    Product* createProduct(const std::string& type) {
        if (type == "A") {
            return new ConcreteProductA();
        } else if (type == "B") {
            return new ConcreteProductB();
        }
        return nullptr;
    }
};
```

### 抽象工厂模式

```cpp
class AbstractFactory {
public:
    virtual ProductA* createProductA() = 0;
    virtual ProductB* createProductB() = 0;
    virtual ~AbstractFactory() = default;
};

class ConcreteFactory1 : public AbstractFactory {
public:
    ProductA* createProductA() override { return new ProductA1(); }
    ProductB* createProductB() override { return new ProductB1(); }
};

class ConcreteFactory2 : public AbstractFactory {
public:
    ProductA* createProductA() override { return new ProductA2(); }
    ProductB* createProductB() override { return new ProductB2(); }
};
```

---

## 2. 结构型模式

### 适配器模式

```cpp
class Target {
public:
    virtual void request() = 0;
};

class Adaptee {
public:
    void specificRequest() {
        std::cout << "Specific request" << std::endl;
    }
};

class Adapter : public Target {
private:
    Adaptee* adaptee;
public:
    Adapter(Adaptee* a) : adaptee(a) {}
    void request() override {
        adaptee->specificRequest();
    }
};
```

### 装饰器模式

```cpp
class Component {
public:
    virtual void operation() = 0;
    virtual ~Component() = default;
};

class ConcreteComponent : public Component {
public:
    void operation() override {
        std::cout << "Base operation" << std::endl;
    }
};

class Decorator : public Component {
protected:
    Component* component;
public:
    Decorator(Component* c) : component(c) {}
    void operation() override {
        component->operation();
    }
};

class ConcreteDecorator : public Decorator {
public:
    ConcreteDecorator(Component* c) : Decorator(c) {}
    void operation() override {
        Decorator::operation();
        std::cout << "Added behavior" << std::endl;
    }
};
```

### 代理模式

```cpp
class Subject {
public:
    virtual void doSomething() = 0;
    virtual ~Subject() = default;
};

class RealSubject : public Subject {
public:
    void doSomething() override {
        std::cout << "Real work" << std::endl;
    }
};

class Proxy : public Subject {
private:
    RealSubject* realSubject;
public:
    void doSomething() override {
        if (!realSubject) {
            realSubject = new RealSubject();
        }
        // 可以添加额外逻辑
        realSubject->doSomething();
    }
};
```

---

## 3. 行为型模式

### 观察者模式

```cpp
#include <vector>

class Observer {
public:
    virtual void update(int state) = 0;
    virtual ~Observer() = default;
};

class Subject {
private:
    std::vector<Observer*> observers;
    int state;
public:
    void attach(Observer* o) { observers.push_back(o); }
    void detach(Observer* o) { /* 移除观察者 */ }
    void setState(int s) {
        state = s;
        notify();
    }
    void notify() {
        for (auto o : observers) {
            o->update(state);
        }
    }
};

class ConcreteObserver : public Observer {
public:
    void update(int state) override {
        std::cout << "State changed to: " << state << std::endl;
    }
};
```

### 策略模式

```cpp
class Strategy {
public:
    virtual int execute(int a, int b) = 0;
    virtual ~Strategy() = default;
};

class AddStrategy : public Strategy {
public:
    int execute(int a, int b) override { return a + b; }
};

class MultiplyStrategy : public Strategy {
public:
    int execute(int a, int b) override { return a * b; }
};

class Context {
private:
    Strategy* strategy;
public:
    Context(Strategy* s) : strategy(s) {}
    void setStrategy(Strategy* s) { strategy = s; }
    int compute(int a, int b) {
        return strategy->execute(a, b);
    }
};
```

### 模板方法模式

```cpp
class AbstractClass {
public:
    void templateMethod() {
        step1();
        step2();
        step3();
    }
    virtual ~AbstractClass() = default;
protected:
    virtual void step1() { std::cout << "Step 1" << std::endl; }
    virtual void step2() = 0;
    virtual void step3() { std::cout << "Step 3" << std::endl; }
};

class ConcreteClass : public AbstractClass {
protected:
    void step2() override {
        std::cout << "Concrete Step 2" << std::endl;
    }
};
```

---

## 常见面试题

### 1. 什么是设计模式？

**答案：**
设计模式是解决特定问题的成熟、可复用的解决方案，是经验的总结。

### 2. 单例模式的实现方式？

**答案：**
- 懒汉式（延迟初始化）
- 饿汉式（提前初始化）
- 双重检查锁定
- 静态局部变量（C++11 线程安全）

### 3. 工厂模式和抽象工厂模式的区别？

**答案：**
- **工厂模式**：创建单一产品
- **抽象工厂模式**：创建产品族

### 4. 装饰器模式和适配器模式的区别？

**答案：**
- **装饰器**：增强对象功能，保持接口不变
- **适配器**：转换接口，适配不同接口

### 5. 观察者模式的应用场景？

**答案：**
- GUI 事件处理
- 消息队列
- 发布-订阅系统

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

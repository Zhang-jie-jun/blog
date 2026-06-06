# C++ 面向对象编程

## 面向对象简介

C++ 是一种支持面向对象编程的语言，通过类和对象实现封装、继承和多态三大特性。

### 三大特性

| 特性 | 描述 |
|------|------|
| **封装** | 将数据和操作数据的方法绑定在一起，隐藏内部实现 |
| **继承** | 从已有类派生出新类，实现代码复用 |
| **多态** | 同一接口有多种实现方式，提高代码灵活性 |

---

## 1. 类与对象

### 类的定义

```cpp
class Person {
private:
    // 私有成员：只能在类内部访问
    std::string name;
    int age;
    
protected:
    // 保护成员：类内部和子类可访问
    std::string address;
    
public:
    // 构造函数
    Person(std::string n, int a) : name(n), age(a) {}
    
    // 析构函数
    ~Person() {}
    
    // 成员函数
    void setName(std::string n) { name = n; }
    std::string getName() const { return name; }
    
    virtual void introduce() const {
        std::cout << "My name is " << name << ", " << age << " years old." << std::endl;
    }
};
```

### 对象的创建和使用

```cpp
// 创建对象
Person p("Alice", 25);

// 访问成员
std::cout << p.getName() << std::endl;

// 调用成员函数
p.introduce();

// 动态创建对象
Person* pp = new Person("Bob", 30);
pp->introduce();
delete pp;
```

---

## 2. 继承

继承允许一个类（派生类）从另一个类（基类）获取属性和方法。

### 继承类型

```cpp
// 公有继承
class Student : public Person {
private:
    std::string school;
    
public:
    Student(std::string n, int a, std::string s) 
        : Person(n, a), school(s) {}
    
    void introduce() const override {
        std::cout << "I'm a student at " << school << std::endl;
        Person::introduce();
    }
};

// 保护继承
class Teacher : protected Person {
    // 基类的 public 成员变为 protected
};

// 私有继承
class Staff : private Person {
    // 基类的 public 和 protected 成员变为 private
};
```

### 继承中的构造函数

```cpp
class Base {
public:
    Base(int x) { std::cout << "Base constructor: " << x << std::endl; }
};

class Derived : public Base {
public:
    Derived(int x, int y) : Base(x) {
        std::cout << "Derived constructor: " << y << std::endl;
    }
};

// 创建 Derived 对象时，先调用 Base 构造函数
Derived d(10, 20);
// 输出:
// Base constructor: 10
// Derived constructor: 20
```

---

## 3. 多态

多态允许不同类的对象通过相同的接口调用不同的实现。

### 虚函数

```cpp
class Shape {
public:
    virtual double area() const = 0;  // 纯虚函数
    virtual ~Shape() = default;
};

class Circle : public Shape {
private:
    double radius;
public:
    Circle(double r) : radius(r) {}
    double area() const override {
        return 3.14159 * radius * radius;
    }
};

class Rectangle : public Shape {
private:
    double width, height;
public:
    Rectangle(double w, double h) : width(w), height(h) {}
    double area() const override {
        return width * height;
    }
};

void printArea(const Shape& shape) {
    std::cout << "Area: " << shape.area() << std::endl;
}

// 使用
Circle c(5);
Rectangle r(4, 6);
printArea(c);   // 调用 Circle::area()
printArea(r);   // 调用 Rectangle::area()
```

### 虚析构函数

```cpp
class Base {
public:
    virtual ~Base() { std::cout << "Base destructor" << std::endl; }
};

class Derived : public Base {
public:
    ~Derived() override { std::cout << "Derived destructor" << std::endl; }
};

// 如果 Base 的析构函数不是虚函数，delete pb 只会调用 Base 的析构函数
Base* pb = new Derived();
delete pb;  // 正确调用 Derived 和 Base 的析构函数
```

---

## 4. 静态成员

静态成员属于类而非对象，所有对象共享同一份数据。

```cpp
class Counter {
private:
    static int count;  // 静态成员变量
public:
    Counter() { count++; }
    ~Counter() { count--; }
    
    static int getCount() {  // 静态成员函数
        return count;
    }
};

// 静态成员变量必须在类外初始化
int Counter::count = 0;

// 使用
Counter c1;
Counter c2;
std::cout << Counter::getCount() << std::endl;  // 输出: 2
```

---

## 5. 友元

友元允许外部函数或类访问私有成员。

```cpp
class MyClass {
private:
    int value;
    
public:
    MyClass(int v) : value(v) {}
    
    // 友元函数
    friend std::ostream& operator<<(std::ostream& os, const MyClass& obj);
    
    // 友元类
    friend class FriendClass;
};

std::ostream& operator<<(std::ostream& os, const MyClass& obj) {
    os << obj.value;  // 可以访问私有成员
    return os;
}

class FriendClass {
public:
    void printValue(const MyClass& obj) {
        std::cout << obj.value << std::endl;  // 可以访问私有成员
    }
};
```

---

## 常见面试题

### 1. 构造函数和析构函数可以是虚函数吗？

**答案：**
- **构造函数**：不可以是虚函数。因为在构造函数执行时，对象的类型还未完全确定。
- **析构函数**：通常应该是虚函数，特别是当类有虚函数时，确保正确调用派生类的析构函数。

### 2. 什么是纯虚函数？

**答案：**
纯虚函数是在基类中声明但没有实现的虚函数，形式为 `virtual void func() = 0;`。包含纯虚函数的类是抽象类，不能实例化。

### 3. override 和 final 关键字的作用？

**答案：**
- **override**：显式声明函数覆盖基类的虚函数，编译器会检查是否正确覆盖。
- **final**：禁止函数被进一步覆盖，或禁止类被继承。

### 4. 公有继承、保护继承、私有继承的区别？

**答案：**
| 基类成员 | 公有继承 | 保护继承 | 私有继承 |
|---------|---------|---------|---------|
| public | public | protected | private |
| protected | protected | protected | private |
| private | 不可访问 | 不可访问 | 不可访问 |

### 5. 什么是菱形继承？如何解决？

**答案：**
菱形继承是指一个派生类同时继承两个基类，而这两个基类又继承自同一个基类。使用虚继承（virtual inheritance）解决。

```cpp
class Base {};
class Left : virtual public Base {};
class Right : virtual public Base {};
class Derived : public Left, public Right {};
```

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
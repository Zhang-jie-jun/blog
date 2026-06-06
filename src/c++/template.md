# C++ 模板编程

## 模板简介

模板是 C++ 的泛型编程基础，允许编写类型无关的代码，提高代码复用性。

### 模板类型

| 类型 | 描述 |
|------|------|
| **函数模板** | 定义通用函数，支持多种类型 |
| **类模板** | 定义通用类，支持多种类型 |
| **变量模板** | C++14 引入，定义通用变量 |

---

## 1. 函数模板

### 基本语法

```cpp
template <typename T>
T add(T a, T b) {
    return a + b;
}

// 使用
int result1 = add<int>(3, 5);
double result2 = add<double>(2.5, 3.7);
// 自动类型推导
int result3 = add(3, 5);
```

### 多参数模板

```cpp
template <typename T, typename U>
auto multiply(T a, U b) -> decltype(a * b) {
    return a * b;
}

int x = multiply(3, 4);           // 12
double y = multiply(2.5, 3);      // 7.5
```

---

## 2. 类模板

### 基本语法

```cpp
template <typename T>
class Stack {
private:
    std::vector<T> elements;
    
public:
    void push(T value) {
        elements.push_back(value);
    }
    
    T pop() {
        T value = elements.back();
        elements.pop_back();
        return value;
    }
    
    bool isEmpty() const {
        return elements.empty();
    }
};

// 使用
Stack<int> intStack;
intStack.push(10);
intStack.push(20);
std::cout << intStack.pop() << std::endl;  // 20
```

### 模板特化

```cpp
// 通用模板
template <typename T>
struct TypeTraits {
    static constexpr bool isPointer = false;
};

// 指针特化
template <typename T>
struct TypeTraits<T*> {
    static constexpr bool isPointer = true;
};

// 使用
TypeTraits<int>::isPointer;      // false
TypeTraits<int*>::isPointer;     // true
```

### 偏特化

```cpp
// 通用模板
template <typename T, typename U>
struct Pair {
    T first;
    U second;
};

// 偏特化：第一个参数为指针
template <typename T, typename U>
struct Pair<T*, U*> {
    T* first;
    U* second;
};
```

---

## 3. 非类型模板参数

模板参数可以是常量表达式。

```cpp
template <typename T, size_t N>
class Array {
private:
    T data[N];
    
public:
    T& operator[](size_t index) {
        return data[index];
    }
    
    size_t size() const {
        return N;
    }
};

// 使用
Array<int, 10> arr;
arr[0] = 42;
std::cout << arr.size() << std::endl;  // 10
```

---

## 4. SFINAE

SFINAE（Substitution Failure Is Not An Error）是模板编程中的重要概念，当模板参数替换失败时，编译器不会报错，而是尝试其他重载。

### enable_if 的使用

```cpp
#include <type_traits>

// 仅对整数类型启用
template <typename T>
typename std::enable_if<std::is_integral<T>::value, T>::type
add(T a, T b) {
    return a + b;
}

// 仅对浮点类型启用
template <typename T>
typename std::enable_if<std::is_floating_point<T>::value, T>::type
add(T a, T b) {
    return a + b;
}
```

### constexpr if（C++17）

```cpp
template <typename T>
auto process(T value) {
    if constexpr (std::is_integral_v<T>) {
        return value * 2;
    } else if constexpr (std::is_floating_point_v<T>) {
        return value * 1.5;
    } else {
        return value;
    }
}
```

---

## 5. 可变参数模板

### 基本语法

```cpp
// 递归终止
void print() {
    std::cout << std::endl;
}

// 可变参数模板
template <typename T, typename... Args>
void print(T first, Args... args) {
    std::cout << first << " ";
    print(args...);
}

// 使用
print(1, "hello", 3.14, 'A');  // 1 hello 3.14 A
```

### 折叠表达式（C++17）

```cpp
template <typename... Args>
auto sum(Args... args) {
    return (args + ...);  // 左折叠：((arg1 + arg2) + arg3) + ...
}

int result = sum(1, 2, 3, 4);  // 10
```

---

## 6. 模板元编程

模板元编程是在编译期执行的编程方式。

### 编译期计算

```cpp
template <int N>
struct Factorial {
    static constexpr int value = N * Factorial<N-1>::value;
};

template <>
struct Factorial<0> {
    static constexpr int value = 1;
};

// 在编译期计算
constexpr int fact5 = Factorial<5>::value;  // 120
```

### 类型计算

```cpp
template <typename... Ts>
struct TypeList {};

template <typename T, typename List>
struct Contains;

template <typename T, typename... Ts>
struct Contains<T, TypeList<Ts...>> {
    static constexpr bool value = (std::is_same_v<T, Ts> || ...);
};

// 使用
using MyTypes = TypeList<int, double, std::string>;
Contains<int, MyTypes>::value;      // true
Contains<char, MyTypes>::value;     // false
```

---

## 常见面试题

### 1. 模板特化和偏特化的区别？

**答案：**
- **特化**：为特定类型提供完全不同的实现
- **偏特化**：为模板参数的一部分提供特定实现

### 2. 什么是 SFINAE？

**答案：**
SFINAE（Substitution Failure Is Not An Error）是指当模板参数替换失败时，编译器不会报错，而是尝试其他重载版本。

### 3. 可变参数模板如何工作？

**答案：**
可变参数模板使用 `...` 语法接受任意数量的参数，通过递归展开参数包。

### 4. 模板和宏的区别？

**答案：**
| 特性 | 模板 | 宏 |
|------|------|----|
| 类型安全 | 是 | 否 |
| 编译检查 | 有 | 无 |
| 代码膨胀 | 可能 | 可能 |
| 调试难度 | 较低 | 较高 |

### 5. 什么是 constexpr if？

**答案：**
C++17 引入的 constexpr if 允许在编译期根据条件选择执行路径，未选中的分支不会被实例化。

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
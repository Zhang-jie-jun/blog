# C++ STL 详解

## STL 简介

STL（Standard Template Library）是 C++ 标准库的核心部分，提供了通用的数据结构和算法。

### STL 六大组件

| 组件 | 描述 |
|------|------|
| **容器（Containers）** | 数据存储结构 |
| **算法（Algorithms）** | 通用算法 |
| **迭代器（Iterators）** | 遍历容器的接口 |
| **函数对象（Functors）** | 可调用对象 |
| **适配器（Adapters）** | 转换接口 |
| **分配器（Allocators）** | 内存分配 |

---

## 1. 容器

### 序列容器

```cpp
#include <vector>
#include <list>
#include <deque>
#include <array>
#include <string>

// vector：动态数组
std::vector<int> vec = {1, 2, 3, 4, 5};
vec.push_back(6);
vec.pop_back();

// list：双向链表
std::list<int> lst = {1, 2, 3};
lst.push_front(0);
lst.remove(2);

// deque：双端队列
std::deque<int> dq;
dq.push_front(1);
dq.push_back(2);

// array：固定大小数组
std::array<int, 5> arr = {1, 2, 3, 4, 5};

// string：字符串
std::string str = "hello";
str += " world";
```

### 关联容器

```cpp
#include <set>
#include <map>
#include <unordered_set>
#include <unordered_map>

// set：有序集合
std::set<int> s = {3, 1, 4, 1, 5};  // {1, 3, 4, 5}
s.insert(2);
s.erase(3);

// map：有序键值对
std::map<std::string, int> m;
m["apple"] = 5;
m["banana"] = 3;

// unordered_set：哈希集合
std::unordered_set<int> us = {1, 2, 3};

// unordered_map：哈希表
std::unordered_map<std::string, int> um;
um["key"] = 42;
```

### 容器适配器

```cpp
#include <stack>
#include <queue>
#include <priority_queue>

// stack：栈
std::stack<int> st;
st.push(1);
st.push(2);
st.top();   // 2
st.pop();

// queue：队列
std::queue<int> q;
q.push(1);
q.push(2);
q.front();  // 1
q.pop();

// priority_queue：优先队列（最大堆）
std::priority_queue<int> pq;
pq.push(3);
pq.push(1);
pq.push(2);
pq.top();   // 3
```

---

## 2. 迭代器

### 迭代器类型

```cpp
#include <vector>

std::vector<int> vec = {1, 2, 3, 4, 5};

// 正向迭代器
for (std::vector<int>::iterator it = vec.begin(); it != vec.end(); ++it) {
    std::cout << *it << " ";
}

// 常量迭代器
for (std::vector<int>::const_iterator it = vec.cbegin(); it != vec.cend(); ++it) {
    std::cout << *it << " ";
}

// 反向迭代器
for (std::vector<int>::reverse_iterator it = vec.rbegin(); it != vec.rend(); ++it) {
    std::cout << *it << " ";
}
```

### 迭代器失效

```cpp
std::vector<int> vec = {1, 2, 3, 4, 5};

auto it = vec.begin();
vec.insert(it + 2, 10);  // 可能使迭代器失效

// 正确做法：使用返回的迭代器
it = vec.insert(it + 2, 10);
```

---

## 3. 算法

### 常用算法

```cpp
#include <algorithm>
#include <vector>

std::vector<int> vec = {3, 1, 4, 1, 5, 9, 2, 6};

// 排序
std::sort(vec.begin(), vec.end());

// 查找
auto it = std::find(vec.begin(), vec.end(), 5);

// 二分查找
bool found = std::binary_search(vec.begin(), vec.end(), 3);

// 统计
int count = std::count(vec.begin(), vec.end(), 1);

// 最大值/最小值
int max_val = *std::max_element(vec.begin(), vec.end());

// 反转
std::reverse(vec.begin(), vec.end());

// 复制
std::vector<int> dest(10);
std::copy(vec.begin(), vec.end(), dest.begin());

// 变换
std::transform(vec.begin(), vec.end(), vec.begin(), [](int x) { return x * 2; });
```

### Lambda 表达式

```cpp
// 排序：按绝对值排序
std::sort(vec.begin(), vec.end(), [](int a, int b) {
    return std::abs(a) < std::abs(b);
});

// 查找：查找第一个偶数
auto even_it = std::find_if(vec.begin(), vec.end(), [](int x) {
    return x % 2 == 0;
});
```

---

## 4. 函数对象

```cpp
#include <functional>

// 预定义函数对象
std::plus<int> add;
int result = add(3, 5);  // 8

std::greater<int> greater_than;
bool is_greater = greater_than(5, 3);  // true

// 使用函数对象作为算法参数
std::sort(vec.begin(), vec.end(), std::greater<int>());

// 绑定器
auto add5 = std::bind(std::plus<int>(), std::placeholders::_1, 5);
int x = add5(3);  // 8

// 包装器
std::function<int(int, int)> func = std::plus<int>();
int y = func(4, 6);  // 10
```

---

## 5. 智能指针

```cpp
#include <memory>

// unique_ptr：独占所有权
std::unique_ptr<int> uptr = std::make_unique<int>(42);

// shared_ptr：共享所有权
std::shared_ptr<int> sptr = std::make_shared<int>(100);

// weak_ptr：弱引用
std::weak_ptr<int> wptr = sptr;
if (auto lock = wptr.lock()) {
    std::cout << *lock << std::endl;
}
```

---

## 常见面试题

### 1. vector 和 list 的区别？

**答案：**
| 特性 | vector | list |
|------|--------|------|
| 存储方式 | 连续数组 | 双向链表 |
| 随机访问 | O(1) | O(n) |
| 插入删除 | O(n) | O(1) |
| 内存碎片 | 可能 | 较小 |

### 2. map 和 unordered_map 的区别？

**答案：**
| 特性 | map | unordered_map |
|------|-----|--------------|
| 底层实现 | 红黑树 | 哈希表 |
| 有序性 | 有序 | 无序 |
| 查找时间 | O(log n) | O(1) 平均 |
| 插入时间 | O(log n) | O(1) 平均 |

### 3. 什么是迭代器失效？

**答案：**
当容器的结构发生改变（如插入、删除元素）时，指向容器元素的迭代器可能失效，使用失效的迭代器会导致未定义行为。

### 4. STL 算法的复杂度？

**答案：**
- `sort`：平均 O(n log n)
- `find`：O(n)
- `binary_search`：O(log n)
- `count`：O(n)

### 5. 什么是 RAII？在 STL 中如何体现？

**答案：**
RAII（Resource Acquisition Is Initialization）是利用对象生命周期管理资源的技术。STL 容器在析构时自动释放内存，智能指针自动管理动态内存，都是 RAII 的体现。

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

# C++ Boost 库

## Boost 简介

Boost 是一个广泛使用的 C++ 库集合，包含了许多高质量的通用库，填补了标准库的空白。

### Boost 的特点

| 特性 | 描述 |
|------|------|
| **高质量** | 经过充分测试，工业级质量 |
| **跨平台** | 支持多种编译器和平台 |
| **标准化** | 多个库已被纳入 C++ 标准 |
| **模块化** | 按需使用，不增加不必要的依赖 |

---

## 1. 常用组件

### 智能指针

```cpp
#include <boost/shared_ptr.hpp>
#include <boost/scoped_ptr.hpp>
#include <boost/weak_ptr.hpp>

// shared_ptr
boost::shared_ptr<int> p1(new int(42));
boost::shared_ptr<int> p2 = p1;

// scoped_ptr（独占所有权，不可转移）
boost::scoped_ptr<int> p3(new int(100));

// weak_ptr
boost::weak_ptr<int> wp = p1;
```

### 容器

```cpp
#include <boost/array.hpp>
#include <boost/ptr_container/ptr_vector.hpp>

// boost::array（类似 std::array）
boost::array<int, 5> arr = {1, 2, 3, 4, 5};

// ptr_vector（存储指针的容器）
boost::ptr_vector<int> pv;
pv.push_back(new int(10));
pv.push_back(new int(20));
```

### 字符串处理

```cpp
#include <boost/algorithm/string.hpp>
#include <string>

std::string str = "  Hello World!  ";

// 去除首尾空格
boost::trim(str);

// 转换为小写
boost::to_lower(str);

// 分割字符串
std::vector<std::string> parts;
boost::split(parts, "a,b,c", boost::is_any_of(","));
```

### 文件系统

```cpp
#include <boost/filesystem.hpp>

namespace fs = boost::filesystem;

// 检查文件是否存在
fs::path path("example.txt");
if (fs::exists(path)) {
    std::cout << "File exists" << std::endl;
}

// 获取文件大小
fs::file_size(path);

// 遍历目录
for (const auto& entry : fs::directory_iterator(".")) {
    std::cout << entry.path() << std::endl;
}
```

---

## 2. 高级组件

### 线程库

```cpp
#include <boost/thread.hpp>

void worker() {
    std::cout << "Working..." << std::endl;
}

// 创建线程
boost::thread t(worker);

// 等待线程完成
t.join();

// 互斥锁
boost::mutex mtx;
void safe_increment(int& counter) {
    boost::lock_guard<boost::mutex> lock(mtx);
    counter++;
}
```

### 日期时间

```cpp
#include <boost/date_time/gregorian/gregorian.hpp>
#include <boost/date_time/posix_time/posix_time.hpp>

// 日期
boost::gregorian::date today(boost::gregorian::day_clock::local_day());
std::cout << today << std::endl;

// 时间
boost::posix_time::ptime now = boost::posix_time::second_clock::local_time();
std::cout << now.time_of_day() << std::endl;
```

### 程序选项

```cpp
#include <boost/program_options.hpp>

namespace po = boost::program_options;

po::options_description desc("Allowed options");
desc.add_options()
    ("help", "produce help message")
    ("compression", po::value<int>(), "set compression level");

po::variables_map vm;
po::store(po::parse_command_line(argc, argv, desc), vm);
po::notify(vm);

if (vm.count("help")) {
    std::cout << desc << std::endl;
}
```

### 序列化

```cpp
#include <boost/archive/text_oarchive.hpp>
#include <boost/archive/text_iarchive.hpp>
#include <fstream>

class MyClass {
private:
    friend class boost::serialization::access;
    
    template <typename Archive>
    void serialize(Archive& ar, const unsigned int version) {
        ar & value;
    }
    
    int value;
};

// 序列化
std::ofstream ofs("data.txt");
boost::archive::text_oarchive oa(ofs);
MyClass obj;
oa << obj;

// 反序列化
std::ifstream ifs("data.txt");
boost::archive::text_iarchive ia(ifs);
MyClass restored_obj;
ia >> restored_obj;
```

---

## 3. 数学与数值

### 任意精度算术

```cpp
#include <boost/multiprecision/cpp_int.hpp>

using namespace boost::multiprecision;

cpp_int large_num("123456789012345678901234567890");
cpp_int result = large_num * large_num;
std::cout << result << std::endl;
```

### 区间算术

```cpp
#include <boost/numeric/interval.hpp>

namespace interval = boost::numeric::interval;

interval::interval<double> a(1.0, 2.0);
interval::interval<double> b(3.0, 4.0);
interval::interval<double> c = a + b;  // [4.0, 6.0]
```

---

## 4. 正则表达式

```cpp
#include <boost/regex.hpp>

std::string text = "Hello 123 World 456";
boost::regex pattern(R"(\d+)");

// 查找匹配
boost::smatch matches;
if (boost::regex_search(text, matches, pattern)) {
    std::cout << "Found: " << matches[0] << std::endl;
}

// 替换
std::string result = boost::regex_replace(text, pattern, "***");
std::cout << result << std::endl;
```

---

## 常见面试题

### 1. Boost 库的作用是什么？

**答案：**
Boost 提供了许多高质量的库组件，填补了 C++ 标准库的空白，包括智能指针、线程、文件系统、正则表达式等，很多组件后来被纳入 C++ 标准。

### 2. Boost 和 STL 的关系？

**答案：**
Boost 是对 STL 的补充和扩展，许多 Boost 库（如智能指针、正则表达式）已经成为 C++ 标准的一部分。

### 3. Boost 智能指针和标准库智能指针的区别？

**答案：**
Boost 的智能指针是标准库智能指针的前身，功能类似但标准库版本更现代，推荐在新项目中使用标准库版本。

### 4. Boost 的线程库和标准库线程库的区别？

**答案：**
Boost.Thread 是标准库线程的基础，标准库线程库在 C++11 中引入，API 类似但更简洁。

### 5. 如何选择是否使用 Boost？

**答案：**
- 如果项目需要兼容旧编译器，使用 Boost
- 如果只需要标准库已有的功能，使用标准库
- 如果需要标准库没有的功能（如文件系统操作、程序选项），考虑使用 Boost

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
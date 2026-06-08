# CMake使用方法

## CMake是什么？
&emsp;&emsp;CMake是一个跨平台的构建系统生成工具，可以根据平台自动生成Makefile、Visual Studio项目文件等。

## CMake基本用法

### 简单示例

**CMakeLists.txt**：
```cmake
cmake_minimum_required(VERSION 3.10)

# 项目名称
project(MyProject)

# 设置C++标准
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# 添加可执行文件
add_executable(myapp main.cpp utils.cpp)

# 添加库
add_library(utils STATIC utils.cpp)
target_link_libraries(myapp utils)
```

### 构建流程
```bash
# 创建构建目录
mkdir build && cd build

# 生成构建文件
cmake ..

# 编译项目
make

# 安装（可选）
make install
```

## CMake常用命令

### 项目设置
```cmake
cmake_minimum_required(VERSION 3.10)  # 最低版本要求
project(MyProject VERSION 1.0)       # 项目名称和版本
```

### 添加可执行文件
```cmake
add_executable(target_name source1.cpp source2.cpp)
```

### 添加库
```cmake
# 静态库
add_library(mylib STATIC src/lib.cpp)

# 动态库
add_library(mylib SHARED src/lib.cpp)

# 目标库链接
target_link_libraries(myapp mylib)
```

### 包含目录
```cmake
include_directories(include)
target_include_directories(myapp PUBLIC include)
```

### 设置编译选项
```cmake
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall")
target_compile_options(myapp PRIVATE -Wextra)
```

## CMake变量

### 常用变量
```cmake
${PROJECT_NAME}      # 项目名称
${PROJECT_VERSION}   # 项目版本
${CMAKE_SOURCE_DIR}  # 源码根目录
${CMAKE_BINARY_DIR}  # 构建目录
${CMAKE_CURRENT_SOURCE_DIR}  # 当前目录
```

### 自定义变量
```cmake
set(MY_VAR "value")
message("My variable: ${MY_VAR}")
```

## 条件判断

```cmake
if(UNIX)
    message("Running on Unix-like system")
elseif(WIN32)
    message("Running on Windows")
endif()

if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    add_definitions(-DDEBUG)
endif()
```

## 查找依赖

### 查找包
```cmake
find_package(Boost REQUIRED COMPONENTS filesystem)
if(Boost_FOUND)
    include_directories(${Boost_INCLUDE_DIRS})
    target_link_libraries(myapp ${Boost_LIBRARIES})
endif()
```

### 查找库
```cmake
find_library(MYLIB mylib PATHS lib)
if(MYLIB)
    target_link_libraries(myapp ${MYLIB})
endif()
```

## 子目录

```cmake
# 添加子目录
add_subdirectory(src)
add_subdirectory(lib)
```

## 安装目标

```cmake
# 安装可执行文件
install(TARGETS myapp DESTINATION bin)

# 安装头文件
install(FILES include/myapp.h DESTINATION include)

# 安装库
install(TARGETS mylib DESTINATION lib)
```

## CMake示例

### 完整示例
```cmake
cmake_minimum_required(VERSION 3.10)
project(Calculator VERSION 1.0.0)

# 设置C++标准
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 添加编译选项
if(CMAKE_BUILD_TYPE MATCHES Debug)
    add_compile_definitions(DEBUG)
endif()

# 包含目录
include_directories(include)

# 添加库
add_library(calc STATIC
    src/add.cpp
    src/subtract.cpp
    src/multiply.cpp
    src/divide.cpp
)

# 添加可执行文件
add_executable(calc_app src/main.cpp)

# 链接库
target_link_libraries(calc_app calc)

# 安装
install(TARGETS calc_app DESTINATION bin)
install(TARGETS calc DESTINATION lib)
install(FILES include/calc.h DESTINATION include)
```

## 常用命令行选项

```bash
cmake .. -DCMAKE_BUILD_TYPE=Debug    # Debug模式
cmake .. -DCMAKE_BUILD_TYPE=Release  # Release模式
cmake .. -DCMAKE_INSTALL_PREFIX=/usr/local  # 指定安装路径
cmake .. -G "Visual Studio 16 2019"  # 指定生成器
cmake --build .                      # 编译项目
cmake --install .                    # 安装项目
```

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

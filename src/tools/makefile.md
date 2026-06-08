# Makefile使用方法

## Makefile是什么？
&emsp;&emsp;Makefile是一种构建自动化工具，用于管理项目的编译和链接过程。通过Makefile，可以自动化执行编译命令，避免手动输入冗长的编译指令。

## Makefile基本结构

### 规则格式
```makefile
目标: 依赖
    命令
```

### 简单示例
```makefile
# 定义编译器
CC = gcc
CFLAGS = -Wall -g

# 目标：生成可执行文件
hello: hello.c
    $(CC) $(CFLAGS) -o hello hello.c

# 清理目标
clean:
    rm -f hello *.o
```

## Makefile变量

### 预定义变量
```makefile
CC      # C编译器，默认cc
CXX     # C++编译器，默认g++
CFLAGS  # C编译选项
CXXFLAGS # C++编译选项
LDFLAGS # 链接选项
```

### 自定义变量
```makefile
SRCS = main.c utils.c
OBJS = $(SRCS:.c=.o)
TARGET = app
```

## 模式规则

### 批量编译
```makefile
# 将所有.c文件编译为.o文件
%.o: %.c
    $(CC) $(CFLAGS) -c $< -o $@
```

### 自动变量
```makefile
$@  # 目标文件名
$<  # 第一个依赖文件名
$^  # 所有依赖文件列表
$*  # 目标文件的基础名（不含扩展名）
```

## 伪目标

### .PHONY声明
```makefile
.PHONY: all clean install

all: $(TARGET)

clean:
    rm -f $(OBJS) $(TARGET)

install:
    cp $(TARGET) /usr/local/bin/
```

## 条件判断

```makefile
DEBUG ?= 0

ifeq ($(DEBUG), 1)
    CFLAGS += -DDEBUG -g
else
    CFLAGS += -O2
endif
```

## 包含其他Makefile

```makefile
# 包含子目录的Makefile
include config.mk
include $(wildcard src/*.mk)
```

## 递归Make

```makefile
SUBDIRS = src lib

all:
    for dir in $(SUBDIRS); do \
        $(MAKE) -C $$dir; \
    done
```

## 常用命令

```makefile
make          # 执行第一个目标
make all      # 执行all目标
make clean    # 清理目标文件
make install  # 安装目标
make -j4      # 并行编译（4个线程）
make -n       # 显示命令但不执行
make -k       # 遇到错误继续执行
```

## Makefile示例

### 完整示例
```makefile
# 编译器设置
CC = gcc
CXX = g++
CFLAGS = -Wall -Wextra -g
CXXFLAGS = $(CFLAGS)
LDFLAGS = -lm

# 文件列表
SRCS = main.c utils.c
OBJS = $(SRCS:.c=.o)
TARGET = myapp

# 默认目标
.PHONY: all clean install debug release

all: debug

debug: CFLAGS += -DDEBUG
debug: $(TARGET)

release: CFLAGS += -O2
release: LDFLAGS += -s
release: $(TARGET)

$(TARGET): $(OBJS)
    $(CC) $(LDFLAGS) -o $@ $^

%.o: %.c
    $(CC) $(CFLAGS) -c $< -o $@

clean:
    rm -f $(OBJS) $(TARGET)

install: $(TARGET)
    mkdir -p /usr/local/bin
    cp $(TARGET) /usr/local/bin/
```

## 常见问题

### 命令前必须使用Tab
&emsp;&emsp;Makefile中命令行必须以Tab开头，不能用空格。

### 如何处理空格问题
&emsp;&emsp;文件名包含空格时，需要用引号或转义。

### 循环依赖问题
&emsp;&emsp;确保依赖关系正确，避免循环依赖。

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

# top - 进程监控工具

## 概述

`top`是一个实时的系统监控工具，用于查看系统的CPU、内存使用情况以及进程状态。

## 基本语法

```bash
top [选项]
```

## 常用选项

| 选项 | 说明 |
|------|------|
| `-d n` | 设置刷新间隔为n秒 |
| `-p pid` | 只监控指定PID的进程 |
| `-u user` | 只显示指定用户的进程 |
| `-H` | 显示线程信息 |
| `-b` | 批处理模式（非交互式） |
| `-n n` | 刷新n次后退出 |

## 输出解读

### 顶部统计信息

```
top - 10:00:00 up 1 day,  2:30,  2 users,  load average: 0.10, 0.05, 0.01
Tasks: 150 total,   1 running, 149 sleeping,   0 stopped,   0 zombie
%Cpu(s):  5.0 us,  2.0 sy,  0.0 ni, 92.0 id,  1.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   7821.3 total,   5500.0 free,   1200.0 used,   1121.3 buff/cache
MiB Swap:   1024.0 total,   1024.0 free,      0.0 used.   6200.0 avail Mem
```

| 字段 | 说明 |
|------|------|
| `load average` | 系统负载（1分钟、5分钟、15分钟） |
| `Tasks` | 进程总数、运行中、睡眠中、停止、僵尸进程 |
| `%Cpu(s)` | CPU使用率（us=用户态，sy=系统态，id=空闲，wa=等待I/O） |
| `Mem` | 内存使用情况 |
| `Swap` | 交换分区使用情况 |

### 进程列表

| 字段 | 说明 |
|------|------|
| `PID` | 进程ID |
| `USER` | 进程所有者 |
| `PR` | 优先级 |
| `NI` | 谦让值（nice值） |
| `VIRT` | 虚拟内存大小 |
| `RES` | 常驻内存大小 |
| `SHR` | 共享内存大小 |
| `S` | 状态（R=运行，S=睡眠，D=不可中断睡眠，Z=僵尸） |
| `%CPU` | CPU使用率 |
| `%MEM` | 内存使用率 |
| `TIME+` | 累计CPU时间 |
| `COMMAND` | 命令名称 |

## 交互命令

在top界面中可以使用以下快捷键：

| 键 | 功能 |
|----|------|
| `h` | 显示帮助 |
| `q` | 退出 |
| `k` | 终止进程 |
| `r` | 改变进程优先级 |
| `P` | 按CPU使用率排序 |
| `M` | 按内存使用率排序 |
| `T` | 按累计时间排序 |
| `1` | 显示所有CPU核心 |
| `z` | 切换彩色显示 |
| `b` | 高亮显示 |

## 使用场景

### 场景1：实时监控系统状态

```bash
# 启动top
top

# 设置刷新间隔为2秒
top -d 2
```

### 场景2：监控指定进程

```bash
# 监控指定PID
top -p 1234

# 监控多个进程
top -p 1234,5678
```

### 场景3：监控指定用户

```bash
top -u www-data
```

### 场景4：批处理模式输出

```bash
# 输出一次后退出
top -b -n 1

# 输出到文件
top -b -n 5 > top_output.txt
```

## 问题分析原理

### 常见问题

**问题1：CPU使用率过高**

```bash
# 按CPU排序
top -P

# 查找占用CPU最高的进程
ps aux --sort=-%cpu | head -5
```

**问题2：内存占用过高**

```bash
# 按内存排序
top -M

# 查找内存占用高的进程
ps aux --sort=-%mem | head -5
```

**问题3：僵尸进程**

```bash
# 查找僵尸进程
ps aux | grep Z

# 杀死僵尸进程（需要杀死父进程）
kill -9 <parent_pid>
```

**问题4：系统负载高**

```bash
# 查看负载
uptime

# 如果负载高但CPU空闲，可能是I/O等待
top  # 查看wa字段
```

## 面试题

**Q1：如何查看系统中占用CPU最高的进程？**
```bash
top -P
```

**Q2：top中的load average是什么？**
- 系统负载表示等待CPU的进程数
- 理想值不应超过CPU核心数

**Q3：如何终止一个进程？**
```bash
# 在top中按k，输入PID
# 或使用kill命令
kill <pid>
```

**Q4：如何查看所有CPU核心的使用情况？**
```bash
top -1  # 按1键
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

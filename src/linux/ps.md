# ps - 进程状态查看

## 概述

`ps`命令用于查看系统中进程的状态信息，包括进程ID、所有者、CPU使用率、内存使用率等。

## 基本语法

```bash
ps [选项]
```

## 常用选项

| 选项 | 说明 |
|------|------|
| `-a` | 显示所有终端的进程 |
| `-u` | 显示用户相关的进程信息 |
| `-x` | 显示无终端的进程 |
| `-e` | 显示所有进程 |
| `-f` | 显示完整格式 |
| `-l` | 显示长格式 |
| `-p pid` | 显示指定PID的进程 |
| `-u user` | 显示指定用户的进程 |
| `--sort` | 按指定字段排序 |

## 输出字段说明

### 标准输出

```
  PID TTY          TIME CMD
 1234 pts/0    00:00:01 bash
 5678 pts/0    00:00:00 ps
```

### 完整格式（-f）

```
UID        PID  PPID  C STIME TTY          TIME CMD
root       1234  1000  0 10:00 pts/0    00:00:01 bash
```

| 字段 | 说明 |
|------|------|
| `UID` | 用户ID |
| `PID` | 进程ID |
| `PPID` | 父进程ID |
| `C` | CPU使用率 |
| `STIME` | 启动时间 |
| `TTY` | 终端 |
| `TIME` | 累计CPU时间 |
| `CMD` | 命令名称 |

### 长格式（-l）

```
F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
0 S     0  1234  1000  0  80   0 -  1234 wait   pts/0    00:00:01 bash
```

| 字段 | 说明 |
|------|------|
| `F` | 进程标志 |
| `S` | 状态（R=运行，S=睡眠，D=不可中断，Z=僵尸） |
| `PRI` | 优先级 |
| `NI` | 谦让值 |
| `SZ` | 内存大小（页数） |
| `WCHAN` | 等待的内核函数 |

## 使用场景

### 场景1：查看所有进程

```bash
# 显示所有进程
ps aux

# 显示所有进程（完整格式）
ps -ef
```

### 场景2：查看指定用户的进程

```bash
ps aux | grep www-data

# 或
ps -u www-data
```

### 场景3：查看指定进程

```bash
ps -p 1234

# 查看进程树
pstree
```

### 场景4：按CPU或内存排序

```bash
# 按CPU使用率排序
ps aux --sort=-%cpu | head -10

# 按内存使用率排序
ps aux --sort=-%mem | head -10
```

### 场景5：查找特定进程

```bash
# 查找nginx进程
ps aux | grep nginx

# 排除grep自身
ps aux | grep nginx | grep -v grep

# 使用pgrep
pgrep nginx
```

## 问题分析原理

### 常见问题

**问题1：查找占用资源高的进程**

```bash
# 按CPU排序
ps aux --sort=-%cpu | head -5

# 按内存排序
ps aux --sort=-%mem | head -5
```

**问题2：查找僵尸进程**

```bash
ps aux | grep Z

# 统计僵尸进程数量
ps aux | grep -c Z
```

**问题3：查看进程树**

```bash
pstree

# 显示PID
pstree -p

# 显示指定进程的树
pstree <pid>
```

**问题4：查看进程的详细信息**

```bash
# 查看进程的完整命令行
ps aux | grep <process>

# 查看进程的环境变量
cat /proc/<pid>/environ

# 查看进程打开的文件
lsof -p <pid>
```

## 面试题

**Q1：如何查看系统中所有进程？**
```bash
ps aux
```

**Q2：如何按CPU使用率排序？**
```bash
ps aux --sort=-%cpu
```

**Q3：如何查找僵尸进程？**
```bash
ps aux | grep Z
```

**Q4：ps aux和ps -ef有什么区别？**
- `ps aux`：BSD风格，显示更多信息
- `ps -ef`：System V风格，显示完整格式

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

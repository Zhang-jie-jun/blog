# free - 内存使用情况

## 概述

`free`命令用于查看系统内存和交换分区的使用情况，是监控系统内存状态的重要工具。

## 基本语法

```bash
free [选项]
```

## 常用选项

| 选项 | 说明 |
|------|------|
| `-b` | 以字节为单位显示 |
| `-k` | 以KB为单位显示（默认） |
| `-m` | 以MB为单位显示 |
| `-g` | 以GB为单位显示 |
| `-h` | 以人类可读格式显示 |
| `-t` | 显示总计（内存+交换分区） |
| `-s n` | 每隔n秒刷新一次 |
| `-c n` | 刷新n次后退出 |

## 输出字段说明

```
              total        used        free      shared  buff/cache   available
Mem:           7.7Gi       1.2Gi       5.4Gi       147Mi       1.1Gi       6.2Gi
Swap:          1.0Gi          0B       1.0Gi
```

| 字段 | 说明 |
|------|------|
| `total` | 总内存 |
| `used` | 已使用内存 |
| `free` | 空闲内存 |
| `shared` | 共享内存 |
| `buff/cache` | 缓冲区/缓存 |
| `available` | 可用内存（free + buff/cache中可回收部分） |

## 使用场景

### 场景1：查看内存使用概览

```bash
# 人类可读格式
free -h

# 显示总计
free -ht
```

### 场景2：持续监控内存

```bash
# 每2秒刷新一次
free -h -s 2

# 每2秒刷新一次，共5次
free -h -s 2 -c 5
```

### 场景3：查看内存详细信息

```bash
# 查看内存统计
cat /proc/meminfo

# 查看内存映射
cat /proc/maps
```

## 问题分析原理

### 内存状态解读

```bash
# 计算内存使用率
free | awk '/Mem:/ {print "使用率:", $3/$2*100 "%"}'

# 检查交换分区使用
free | awk '/Swap:/ {print "Swap使用:", $3/$2*100 "%"}'
```

### 常见问题

**问题1：内存不足**

```bash
# 查看内存使用
free -h

# 查找内存占用高的进程
top

# 或
ps aux --sort=-%mem | head -10
```

**问题2：交换分区频繁使用**

```bash
# 检查swap使用
free -h | grep Swap

# 如果swap使用过高，可能需要：
# 1. 增加物理内存
# 2. 调整swappiness
cat /proc/sys/vm/swappiness
echo 10 > /proc/sys/vm/swappiness  # 临时调整
```

**问题3：缓存占用过多**

```bash
# 清理页缓存
echo 1 > /proc/sys/vm/drop_caches

# 清理目录项和inodes
echo 2 > /proc/sys/vm/drop_caches

# 清理所有缓存
echo 3 > /proc/sys/vm/drop_caches
```

## 面试题

**Q1：如何查看系统内存使用情况？**
```bash
free -h
```

**Q2：available和free有什么区别？**
- `free`：真正未使用的物理内存
- `available`：可用于新进程的内存（包括free + 可回收缓存）

**Q3：如何持续监控内存使用？**
```bash
free -h -s 2
```

**Q4：如何清理系统缓存？**
```bash
echo 3 > /proc/sys/vm/drop_caches
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

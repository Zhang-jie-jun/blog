# iostat - 磁盘I/O统计工具

## 概述

`iostat`是一个用于报告系统磁盘I/O活动统计信息的工具，可以帮助管理员了解磁盘使用情况、发现I/O瓶颈。

## 基本语法

```bash
iostat [选项] [设备] [间隔时间] [次数]
```

## 常用选项

| 选项 | 说明 |
|------|------|
| `-c` | 只显示CPU统计信息 |
| `-d` | 只显示磁盘统计信息 |
| `-k` | 以KB为单位显示 |
| `-m` | 以MB为单位显示 |
| `-x` | 显示扩展统计信息 |
| `-t` | 显示时间戳 |

## 输出字段说明

### 基本输出

| 字段 | 说明 |
|------|------|
| `tps` | 每秒向磁盘发起的I/O请求数 |
| `rMB/s` | 每秒读取的MB数 |
| `wMB/s` | 每秒写入的MB数 |
| `rMB` | 累计读取的MB数 |
| `wMB` | 累计写入的MB数 |

### 扩展输出（-x选项）

| 字段 | 说明 |
|------|------|
| `rrqm/s` | 每秒合并的读请求数 |
| `wrqm/s` | 每秒合并的写请求数 |
| `r/s` | 每秒完成的读I/O次数 |
| `w/s` | 每秒完成的写I/O次数 |
| `rMB/s` | 每秒读取的MB数 |
| `wMB/s` | 每秒写入的MB数 |
| `avgrq-sz` | 平均I/O请求大小（扇区） |
| `avgqu-sz` | 平均I/O队列长度 |
| `await` | 平均I/O等待时间（毫秒） |
| `r_await` | 平均读I/O等待时间（毫秒） |
| `w_await` | 平均写I/O等待时间（毫秒） |
| `svctm` | 平均I/O服务时间（毫秒） |
| `%util` | 设备繁忙时间百分比 |

## 使用场景

### 场景1：查看系统整体I/O状态

```bash
# 查看所有磁盘的I/O统计
iostat

# 以MB为单位显示
iostat -m

# 显示扩展信息
iostat -x
```

### 场景2：持续监控磁盘I/O

```bash
# 每2秒刷新一次，共刷新10次
iostat -x 2 10

# 只监控特定磁盘
iostat -x sda 2
```

### 场景3：分析磁盘性能问题

```bash
# 查看磁盘繁忙程度
iostat -x | grep sda

# 比较多个磁盘的性能
iostat -x sda sdb sdc
```

## 问题分析原理

### 指标解读

1. **%util**：设备繁忙时间百分比
    - 如果持续接近100%，说明磁盘I/O饱和
    - 可能导致系统响应变慢

2. **avgqu-sz**：平均队列长度
    - 值过高表示I/O请求积压严重
    - 通常不应超过设备队列深度的50%

3. **await**：平均等待时间
    - 包括队列等待时间 + 服务时间
    - 过高说明I/O压力大

4. **svctm**：服务时间
    - 实际处理I/O请求的时间
    - 过高可能表示磁盘硬件问题

### 常见问题诊断

**问题1：磁盘I/O过高**

```bash
# 找出繁忙的磁盘
iostat -x 1 | grep -E "(sda|sdb|%util)"

# 找出占用I/O的进程
iotop
```

**问题2：写入性能差**

```bash
# 查看写等待时间
iostat -x | awk '/sda/ {print "写等待时间:", $10, "ms"}'
```

**问题3：I/O队列过长**

```bash
# 查看队列长度
iostat -x | awk '/sda/ {print "队列长度:", $9}'
```

## 面试题

**Q1：如何查看磁盘I/O使用情况？**
```bash
iostat -x
```

**Q2：如何判断磁盘是否I/O饱和？**
```bash
# 查看%util字段，如果持续接近100%表示饱和
iostat -x | grep sda
```

**Q3：如何持续监控磁盘I/O？**
```bash
iostat -x 2 10  # 每2秒显示一次，共显示10次
```

**Q4：iostat中的tps和r/s、w/s有什么区别？**
- `tps`：每秒向磁盘发起的I/O请求数
- `r/s`：每秒完成的读I/O次数
- `w/s`：每秒完成的写I/O次数

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

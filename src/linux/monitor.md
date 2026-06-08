# 系统监控及诊断命令详解

## 概述

系统监控是Linux运维的重要部分，包括CPU、内存、磁盘、网络等资源的监控，以及系统故障的诊断和排查。

---

## 一、CPU监控

### 1. top

```bash
# 实时监控CPU
top

# 按CPU排序
P

# 查看所有CPU核心
1

# 退出
q
```

### 2. mpstat

```bash
# 查看CPU统计
mpstat

# 查看所有核心
mpstat -P ALL

# 每秒刷新一次
mpstat 1
```

### 3. vmstat

```bash
# 查看虚拟内存统计
vmstat

# 指定间隔和次数
vmstat 2 10

# 详细模式
vmstat -a
```

---

## 二、内存监控

### 1. free

```bash
# 查看内存使用
free -h

# 查看内存和交换分区
free -ht

# 查看详细信息
cat /proc/meminfo
```

### 2. vmstat

```bash
# 查看内存统计
vmstat -s

# 查看内存分页
vmstat -p
```

---

## 三、磁盘监控

### 1. iostat

```bash
# 查看磁盘I/O
iostat -x

# 指定设备
iostat -x /dev/sda

# 持续监控
iostat -x 2
```

### 2. df

```bash
# 查看磁盘空间
df -h

# 查看inode
df -i

# 查看指定文件系统
df -h /
```

### 3. du

```bash
# 查看目录大小
du -sh /var

# 查看子目录大小
du -h --max-depth=1 /var

# 按大小排序
du -ah /var | sort -rh | head -10
```

---

## 四、网络监控

### 1. netstat

```bash
# 查看网络连接
netstat -tunp

# 查看监听端口
netstat -tlnp

# 查看路由表
netstat -r
```

### 2. ss

```bash
# 查看网络连接（替代netstat）
ss -tunp

# 查看监听端口
ss -tlnp

# 查看UDP连接
ss -uapn
```

### 3. tcpdump

```bash
# 捕获所有网络流量
tcpdump

# 捕获指定接口
tcpdump -i eth0

# 捕获指定端口
tcpdump port 80

# 捕获指定IP
tcpdump host 192.168.1.1

# 保存到文件
tcpdump -w capture.pcap

# 读取捕获文件
tcpdump -r capture.pcap
```

---

## 五、进程监控

### 1. ps

```bash
# 查看所有进程
ps aux

# 按CPU排序
ps aux --sort=-%cpu | head -10

# 按内存排序
ps aux --sort=-%mem | head -10

# 查看进程树
pstree

# 查看指定进程
ps aux | grep nginx
```

### 2. top

```bash
# 实时监控进程
top

# 按内存排序
M

# 按CPU排序
P

# 终止进程
k
```

### 3. htop

```bash
# 更友好的进程监控
htop

# 按CPU排序
F6 -> PERCENT_CPU

# 搜索进程
/nginx

# 终止进程
F9
```

---

## 六、系统诊断

### 1. dmesg

```bash
# 查看内核日志
dmesg

# 查看最近的日志
dmesg | tail -20

# 查看错误信息
dmesg | grep -i error

# 查看硬件信息
dmesg | grep -i memory
```

### 2. journalctl

```bash
# 查看系统日志
journalctl

# 查看指定服务日志
journalctl -u nginx

# 实时查看日志
journalctl -u nginx -f

# 查看最近的日志
journalctl -n 50

# 查看指定时间的日志
journalctl --since "2024-01-01" --until "2024-01-02"
```

### 3. strace

```bash
# 追踪系统调用
strace -p <pid>

# 追踪指定进程
strace -o output.txt -p <pid>

# 追踪命令执行
strace ls -la

# 统计系统调用
strace -c ls -la
```

### 4. ltrace

```bash
# 追踪库调用
ltrace -p <pid>

# 追踪命令
ltrace ls -la
```

---

## 七、性能分析工具

### 1. perf

```bash
# 性能分析
perf top

# 记录性能数据
perf record -g -p <pid>

# 分析记录数据
perf report
```

### 2. vmstat

```bash
# 查看系统状态
vmstat 1

# 查看磁盘I/O
vmstat -d
```

### 3. iostat

```bash
# 查看磁盘统计
iostat -xz 1
```

---

## 八、故障诊断流程

### 步骤1：检查系统负载

```bash
uptime
top
```

### 步骤2：检查CPU使用

```bash
mpstat -P ALL
ps aux --sort=-%cpu | head -10
```

### 步骤3：检查内存使用

```bash
free -h
cat /proc/meminfo
```

### 步骤4：检查磁盘使用

```bash
df -h
iostat -x
```

### 步骤5：检查网络连接

```bash
netstat -tunp
ss -tlnp
```

### 步骤6：查看日志

```bash
dmesg | tail -20
journalctl -p err
```

---

## 九、实战案例

### 案例1：CPU使用率过高

```bash
# 查找占用CPU高的进程
top -P

# 查看进程详细信息
ps aux | grep <pid>

# 分析进程
strace -p <pid>
```

### 案例2：内存不足

```bash
# 查看内存使用
free -h

# 查找内存占用高的进程
ps aux --sort=-%mem | head -10

# 检查交换分区使用
vmstat 1
```

### 案例3：磁盘I/O过高

```bash
# 查看磁盘使用
iostat -xz 1

# 查找占用I/O的进程
iotop
```

---

## 十、面试题

**Q1：如何查看系统负载？**
```bash
uptime
```

**Q2：如何查看CPU使用情况？**
```bash
top
mpstat -P ALL
```

**Q3：如何查看内存使用？**
```bash
free -h
cat /proc/meminfo
```

**Q4：如何查看进程？**
```bash
ps aux
top
```

**Q5：如何追踪系统调用？**
```bash
strace -p <pid>
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

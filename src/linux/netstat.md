# netstat - 网络连接状态

## 概述

`netstat`命令用于查看系统的网络连接状态、路由表、接口统计等信息。

## 基本语法

```bash
netstat [选项]
```

## 常用选项

| 选项 | 说明 |
|------|------|
| `-a` | 显示所有连接（包括监听和非监听） |
| `-t` | 只显示TCP连接 |
| `-u` | 只显示UDP连接 |
| `-n` | 以数字形式显示地址和端口 |
| `-p` | 显示进程ID和名称 |
| `-l` | 只显示监听状态的连接 |
| `-r` | 显示路由表 |
| `-i` | 显示网络接口统计 |
| `-s` | 显示网络统计信息 |

## 输出字段说明

### 连接信息

```
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 192.168.1.100:22       10.0.0.5:54321          ESTABLISHED 1234/sshd
```

| 字段 | 说明 |
|------|------|
| `Proto` | 协议（tcp/udp） |
| `Recv-Q` | 接收队列（未处理的字节数） |
| `Send-Q` | 发送队列（未确认的字节数） |
| `Local Address` | 本地地址和端口 |
| `Foreign Address` | 远程地址和端口 |
| `State` | 连接状态 |
| `PID/Program name` | 进程ID和名称 |

### TCP连接状态

| 状态 | 说明 |
|------|------|
| `LISTEN` | 监听中 |
| `ESTABLISHED` | 已建立连接 |
| `SYN_SENT` | 发送SYN请求 |
| `SYN_RECV` | 收到SYN请求 |
| `FIN_WAIT1` | 主动关闭，等待ACK |
| `FIN_WAIT2` | 等待对方关闭 |
| `TIME_WAIT` | 等待连接关闭 |
| `CLOSE_WAIT` | 被动关闭，等待应用层关闭 |
| `LAST_ACK` | 等待最后ACK |
| `CLOSED` | 连接已关闭 |

## 使用场景

### 场景1：查看所有网络连接

```bash
# 显示所有连接
netstat -a

# 以数字形式显示
netstat -an

# 显示进程信息
netstat -tunp
```

### 场景2：查看监听端口

```bash
# 查看所有监听端口
netstat -tlnp

# 只查看TCP监听
netstat -tlp
```

### 场景3：查看路由表

```bash
netstat -r
```

### 场景4：查看网络接口

```bash
netstat -i
```

### 场景5：查找占用指定端口的进程

```bash
# 查找80端口
netstat -tlnp | grep :80

# 查找指定端口
netstat -tunp | grep 443
```

## 问题分析原理

### 常见问题

**问题1：端口被占用**

```bash
# 查找占用端口的进程
netstat -tlnp | grep :8080

# 使用ss命令（更高效）
ss -tlnp | grep :8080

# 杀死进程
kill -9 <pid>
```

**问题2：大量TIME_WAIT连接**

```bash
# 查看TIME_WAIT连接数
netstat -tnp | grep TIME_WAIT | wc -l

# 调整系统参数
echo 1 > /proc/sys/net/ipv4/tcp_tw_reuse
echo 1 > /proc/sys/net/ipv4/tcp_tw_recycle
```

**问题3：网络连接异常**

```bash
# 检查SYN_RECV状态
netstat -tnp | grep SYN_RECV

# 如果大量SYN_RECV，可能是SYN攻击
# 可以调整tcp_syncookies
echo 1 > /proc/sys/net/ipv4/tcp_syncookies
```

**问题4：路由问题**

```bash
# 查看路由表
netstat -r

# 添加路由
route add -net 192.168.0.0/24 gw 10.0.0.1

# 删除路由
route del -net 192.168.0.0/24
```

## 面试题

**Q1：如何查看系统中监听的端口？**
```bash
netstat -tlnp
```

**Q2：如何查找占用80端口的进程？**
```bash
netstat -tlnp | grep :80
```

**Q3：TIME_WAIT状态是什么？**
- 主动关闭连接的一方在发送FIN后等待2MSL时间
- 确保对方收到ACK，防止旧的TCP分段干扰新连接

**Q4：netstat和ss命令有什么区别？**
- `ss`是`netstat`的替代品，性能更好
- `ss`显示更多TCP状态信息

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

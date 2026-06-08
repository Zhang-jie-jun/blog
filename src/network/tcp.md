# TCP协议详细原理

## 概述

TCP（Transmission Control Protocol）是一种面向连接的、可靠的传输层协议。它提供了数据的有序、可靠传输，是互联网中最常用的协议之一。

---

## 一、TCP协议特点

### 1. 面向连接

TCP是面向连接的协议，在传输数据前必须先建立连接：

```
┌──────────────┐          三次握手          ┌──────────────┐
│   客户端     │ ─────────────────────────> │   服务器     │
│              │ <───────────────────────── │              │
│              │ ─────────────────────────> │              │
│  ESTABLISHED │                           │  ESTABLISHED │
└──────────────┘                           └──────────────┘
```

### 2. 可靠传输

TCP通过多种机制保证数据的可靠传输：

| 机制 | 作用 |
|------|------|
| **确认应答** | 接收方收到数据后发送ACK确认 |
| **重传机制** | 未收到确认时重新发送数据 |
| **序号机制** | 保证数据按序到达 |
| **流量控制** | 防止发送过快导致接收方溢出 |
| **拥塞控制** | 防止网络拥塞 |

### 3. 有序传输

TCP使用序号（Sequence Number）保证数据的有序性：

```
发送方:  [1-100] [101-200] [201-300]
接收方:  [1-100] [201-300] [101-200]  (乱序到达)
         ↓
接收方:  [1-100] [101-200] [201-300]  (重新排序后)
```

---

## 二、TCP头部结构

```
0                   1                   2                   3
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Sequence Number                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Acknowledgment Number                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Data |           |U|A|P|R|S|F|                               |
| Offset| Reserved  |R|C|S|S|Y|I|            Window             |
|       |           |G|K|H|T|N|N|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           Checksum            |         Urgent Pointer        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options                    |    Padding    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

### 关键字段说明

| 字段 | 长度 | 说明 |
|------|------|------|
| **Source Port** | 16位 | 源端口号 |
| **Destination Port** | 16位 | 目的端口号 |
| **Sequence Number** | 32位 | 序列号，标识当前发送的数据位置 |
| **Acknowledgment Number** | 32位 | 确认号，期望接收的下一个字节序号 |
| **Data Offset** | 4位 | TCP头部长度（以32位为单位） |
| **Flags** | 6位 | 控制标志位 |
| **Window** | 16位 | 接收窗口大小（流量控制） |
| **Checksum** | 16位 | 校验和 |
| **Urgent Pointer** | 16位 | 紧急数据指针 |

### 控制标志位

| 标志 | 含义 |
|------|------|
| **URG** | 紧急指针有效 |
| **ACK** | 确认号有效 |
| **PSH** | 立即推送数据到应用层 |
| **RST** | 重置连接 |
| **SYN** | 同步序列号（建立连接） |
| **FIN** | 发送方数据发送完毕 |

---

## 三、TCP连接管理

### 1. 三次握手（建立连接）

```
客户端                    服务器
  |                         |
  |  SYN=1, seq=x          |  第一次握手
  |------------------------>|
  |                         |
  |  SYN=1, ACK=1,         |  第二次握手
  |  seq=y, ack=x+1        |
  |<------------------------|
  |                         |
  |  ACK=1, ack=y+1        |  第三次握手
  |------------------------>|
  |                         |
  |     连接已建立           |
```

**详细过程**：

1. **第一次握手**：客户端发送SYN包，进入`SYN_SENT`状态
   - SYN=1，表示请求建立连接
   - seq=x，初始序列号

2. **第二次握手**：服务器收到SYN包，回复SYN+ACK包，进入`SYN_RCVD`状态
   - SYN=1，同意建立连接
   - ACK=1，确认收到客户端的SYN
   - seq=y，服务器的初始序列号
   - ack=x+1，期望接收的下一个字节

3. **第三次握手**：客户端收到SYN+ACK包，发送ACK包，进入`ESTABLISHED`状态
   - ACK=1，确认收到服务器的SYN
   - ack=y+1，期望接收的下一个字节

**为什么需要三次握手？**
- 确认双方的发送和接收能力正常
- 防止已失效的连接请求报文突然到达服务器

### 2. 四次挥手（关闭连接）

```
客户端                    服务器
  |                         |
  |  FIN=1, seq=x          |  第一次挥手
  |------------------------>|  (客户端不再发送数据)
  |                         |
  |  ACK=1, ack=x+1        |  第二次挥手
  |<------------------------|  (服务器确认收到FIN)
  |                         |
  |  FIN=1, ACK=1,         |  第三次挥手
  |  seq=y, ack=x+1        |  (服务器不再发送数据)
  |<------------------------|
  |                         |
  |  ACK=1, ack=y+1        |  第四次挥手
  |------------------------>|
  |                         |
  |     连接已关闭           |
```

**详细过程**：

1. **第一次挥手**：客户端发送FIN包，进入`FIN_WAIT_1`状态
   - FIN=1，表示数据发送完毕，请求关闭连接

2. **第二次挥手**：服务器收到FIN包，发送ACK包，进入`CLOSE_WAIT`状态
   - ACK=1，确认收到FIN
   - ack=x+1，期望接收的下一个字节

3. **第三次挥手**：服务器发送FIN包，进入`LAST_ACK`状态
   - FIN=1，表示服务器数据发送完毕
   - ACK=1，确认收到客户端的ACK
   - seq=y，服务器的序列号
   - ack=x+1

4. **第四次挥手**：客户端收到FIN包，发送ACK包，进入`TIME_WAIT`状态
   - ACK=1，确认收到FIN
   - ack=y+1，期望接收的下一个字节

**TIME_WAIT状态的作用**：
- 确保最后一个ACK能到达服务器
- 等待网络中残留的分组过期（2MSL时间）

---

## 四、TCP可靠传输机制

### 1. 序号与确认

**序列号（Sequence Number）**：标识数据在流中的位置

**确认号（Acknowledgment Number）**：期望接收的下一个字节的序号

```
发送方发送:  [seq=100, data=50字节]
接收方收到后回复: [ACK=1, ack=150]  (表示已收到100-149字节)
```

### 2. 重传机制

#### 超时重传

```
发送方                    接收方
  |                         |
  |  [seq=100]             |
  |------------------------>|
  |                         |  (数据丢失)
  |  [超时]                 |
  |                         |
  |  [seq=100] (重传)      |
  |------------------------>|
  |                         |
  |         ACK=150        |
  |<------------------------|
```

**超时时间计算**：
- 使用动态超时算法（RTO）
- RTO = RTT_smoothed + 4 * RTT_variance

#### 快速重传

当收到3个重复的ACK时，立即重传：

```
发送方                    接收方
  |                         |
  |  [100-149] [150-199]    |
  |------------------------>|
  |                         |  ([100-149]丢失)
  |         ACK=100        |  (期望收到100)
  |<------------------------|
  |                         |
  |         ACK=100        |  (重复ACK)
  |<------------------------|
  |                         |
  |         ACK=100        |  (第三个重复ACK)
  |<------------------------|
  |                         |
  |  [100-149] (快速重传)   |
  |------------------------>|
```

### 3. 流量控制

使用滑动窗口实现流量控制：

```
发送方窗口:
  ┌─────────────────────────────────────────────────┐
  │ 已发送已确认 | 已发送未确认 |    可发送     | 不可发送 |
  │   [1-100]   |  [101-200]  |   [201-300]  |  [301+]  │
  └─────────────────────────────────────────────────┘
                         ↑
                   窗口向右滑动

接收方通知: Window=100 (表示接收方缓冲区还有100字节空间)
```

**滑动窗口公式**：
```
可用窗口 = 接收窗口 - (已发送未确认的数据)
```

### 4. 拥塞控制

#### 慢启动（Slow Start）

- 初始拥塞窗口（cwnd）= 1个MSS
- 每收到一个ACK，cwnd翻倍
- 指数增长：1 → 2 → 4 → 8 → ...

#### 拥塞避免（Congestion Avoidance）

- 当cwnd达到慢启动阈值（ssthresh）后
- 每收到一个ACK，cwnd增加1/cwnd
- 线性增长

#### 快速恢复（Fast Recovery）

- 收到3个重复ACK时进入快速恢复
- 将ssthresh设为当前cwnd的一半
- cwnd从ssthresh开始线性增长

**状态转换**：
```
慢启动 → (cwnd >= ssthresh) → 拥塞避免 → (丢包) → 快速恢复 → 拥塞避免
```

---

## 五、TCP状态机

```
                    ┌───────────────────────────────┐
                    │                               │
                    ▼                               │
            ┌───────────────┐                       │
            │   CLOSED      │                       │
            └───────┬───────┘                       │
                    │ (SYN)                         │
                    ▼                               │
            ┌───────────────┐                       │
            │  SYN_SENT     │◄──────────────────────┤
            └───────┬───────┘                       │
                    │ (SYN+ACK)                     │
                    ▼                               │
            ┌───────────────┐                       │
            │ SYN_RCVD      │                       │
            └───────┬───────┘                       │
                    │ (ACK)                         │
                    ▼                               │
    ┌───────────────────────────────┐               │
    │                               │               │
    │         ESTABLISHED           │               │
    │                               │               │
    └───────┬─────────────────┬─────┘               │
            │ (FIN)           │ (FIN)               │
            ▼                 ▼                     │
    ┌───────────────┐   ┌───────────────┐           │
    │ FIN_WAIT_1    │   │ CLOSE_WAIT    │           │
    └───────┬───────┘   └───────┬───────┘           │
            │ (ACK)             │ (FIN)              │
            ▼                   ▼                   │
    ┌───────────────┐   ┌───────────────┐           │
    │ FIN_WAIT_2    │   │  LAST_ACK     │           │
    └───────┬───────┘   └───────┬───────┘           │
            │ (FIN)             │ (ACK)              │
            ▼                   ▼                   │
    ┌───────────────┐           │                   │
    │ TIME_WAIT     │           └───────────┬───────┘
    └───────┬───────┘                       │
            │ (2MSL超时)                     │
            ▼                               │
    ┌───────────────┐                       │
    │   CLOSED      │◄──────────────────────┘
    └───────────────┘
```

### 常见状态说明

| 状态 | 说明 |
|------|------|
| **CLOSED** | 初始状态，连接已关闭 |
| **SYN_SENT** | 已发送SYN，等待SYN+ACK |
| **SYN_RCVD** | 已收到SYN，发送了SYN+ACK |
| **ESTABLISHED** | 连接已建立，可以传输数据 |
| **FIN_WAIT_1** | 已发送FIN，等待ACK |
| **FIN_WAIT_2** | 已收到ACK，等待FIN |
| **CLOSE_WAIT** | 已收到FIN，等待应用层关闭 |
| **LAST_ACK** | 已发送FIN，等待ACK |
| **TIME_WAIT** | 已发送ACK，等待2MSL超时 |

---

## 六、代码示例

### 1. TCP服务端（Go）

```go
package main

import (
    "bufio"
    "fmt"
    "net"
    "os"
)

func main() {
    // 监听端口
    listener, err := net.Listen("tcp", ":8080")
    if err != nil {
        fmt.Printf("Failed to listen: %v\n", err)
        os.Exit(1)
    }
    defer listener.Close()

    fmt.Println("Server listening on :8080")

    for {
        // 接受连接
        conn, err := listener.Accept()
        if err != nil {
            fmt.Printf("Failed to accept: %v\n", err)
            continue
        }

        // 处理连接（并发）
        go handleConnection(conn)
    }
}

func handleConnection(conn net.Conn) {
    defer conn.Close()

    reader := bufio.NewReader(conn)
    for {
        // 读取数据
        data, err := reader.ReadString('\n')
        if err != nil {
            fmt.Printf("Connection closed: %v\n", err)
            return
        }

        fmt.Printf("Received: %s", data)

        // 回复数据
        _, err = conn.Write([]byte("Hello from server!\n"))
        if err != nil {
            fmt.Printf("Failed to write: %v\n", err)
            return
        }
    }
}
```

### 2. TCP客户端（Go）

```go
package main

import (
    "bufio"
    "fmt"
    "net"
    "os"
)

func main() {
    // 连接服务器
    conn, err := net.Dial("tcp", "localhost:8080")
    if err != nil {
        fmt.Printf("Failed to connect: %v\n", err)
        os.Exit(1)
    }
    defer conn.Close()

    // 发送数据
    _, err = conn.Write([]byte("Hello from client!\n"))
    if err != nil {
        fmt.Printf("Failed to write: %v\n", err)
        return
    }

    // 接收响应
    reader := bufio.NewReader(conn)
    response, err := reader.ReadString('\n')
    if err != nil {
        fmt.Printf("Failed to read: %v\n", err)
        return
    }

    fmt.Printf("Server response: %s", response)
}
```

---

## 七、常见问题分析

### 1. 连接建立失败

**排查步骤**：

```bash
# 1. 检查端口是否监听
ss -tlnp | grep :8080

# 2. 检查防火墙规则
iptables -L INPUT | grep 8080
ufw status

# 3. 测试端口连通性
nc -zv localhost 8080
telnet localhost 8080

# 4. 抓包分析
tcpdump -i lo port 8080 -w capture.pcap
```

### 2. 数据丢失

**排查步骤**：

```bash
# 1. 查看TCP状态
ss -tunp | grep ESTABLISHED

# 2. 查看网络统计
netstat -s | grep -i retrans

# 3. 抓包分析
tcpdump -i eth0 port 80 -w capture.pcap

# 4. 检查拥塞控制算法
sysctl net.ipv4.tcp_congestion_control
```

### 3. TIME_WAIT过多

**问题原因**：短连接频繁建立和关闭

**解决方案**：

```bash
# 1. 减少TIME_WAIT时间
sysctl -w net.ipv4.tcp_fin_timeout=30

# 2. 启用TIME_WAIT复用
sysctl -w net.ipv4.tcp_tw_reuse=1

# 3. 增加TIME_WAIT数量
sysctl -w net.ipv4.tcp_max_tw_buckets=5000

# 4. 使用长连接
```

### 4. SYN攻击

**问题现象**：大量SYN_RCVD状态连接

**解决方案**：

```bash
# 1. 启用SYN cookies
sysctl -w net.ipv4.tcp_syncookies=1

# 2. 增加半开连接队列
sysctl -w net.ipv4.tcp_max_syn_backlog=65535

# 3. 限制SYN速率
iptables -A INPUT -p tcp --syn -m limit --limit 100/s --limit-burst 200 -j ACCEPT
iptables -A INPUT -p tcp --syn -j DROP
```

---

## 八、UDP协议详解

### 1. UDP协议特点

UDP（User Datagram Protocol）是一种无连接的、不可靠的传输层协议：

| 特性 | 说明 |
|------|------|
| **无连接** | 不需要建立连接，直接发送数据 |
| **不可靠** | 不保证数据到达，不保证顺序 |
| **低延迟** | 开销小，适合实时应用 |
| **面向报文** | 数据以报文为单位传输，保留边界 |
| **无拥塞控制** | 发送速率不受网络状态影响 |

### 2. UDP头部结构

```
0                   1                   2                   3
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|            Length             |           Checksum            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

**关键字段说明**：

| 字段 | 长度 | 说明 |
|------|------|------|
| **Source Port** | 16位 | 源端口号（可选，0表示无） |
| **Destination Port** | 16位 | 目的端口号 |
| **Length** | 16位 | UDP数据报长度（头部+数据） |
| **Checksum** | 16位 | 校验和（可选，全0表示不校验） |

### 3. UDP校验和计算

UDP校验和覆盖伪头部、UDP头部和数据：

```
伪头部（12字节）：
+--------+--------+--------+--------+
| 源IP地址 | 目的IP地址 | 0 | 协议 | UDP长度 |
+--------+--------+--------+--------+
```

**计算步骤**：
1. 构造伪头部
2. 将UDP头部和数据拼接
3. 如果长度为奇数，末尾补0
4. 按16位分组求和
5. 取反得到校验和

### 4. UDP适用场景

| 场景 | 说明 |
|------|------|
| **实时音视频** | 直播、视频会议、语音通话 |
| **实时游戏** | 网络游戏、多人在线游戏 |
| **DNS查询** | 域名解析，短小请求 |
| **SNMP** | 网络管理协议 |
| **广播/组播** | 一对多通信 |

### 5. UDP可靠传输实现

虽然UDP本身不可靠，但可以在应用层实现可靠性：

```go
// UDP可靠传输示例（带确认和重传）
package main

import (
    "fmt"
    "net"
    "time"
)

func main() {
    conn, err := net.DialUDP("udp", nil, &net.UDPAddr{
        IP:   net.ParseIP("127.0.0.1"),
        Port: 8080,
    })
    if err != nil {
        fmt.Printf("Failed to connect: %v\n", err)
        return
    }
    defer conn.Close()

    // 设置读写超时
    conn.SetDeadline(time.Now().Add(5 * time.Second))

    // 发送数据
    message := []byte("Hello, UDP!")
    _, err = conn.Write(message)
    if err != nil {
        fmt.Printf("Failed to write: %v\n", err)
        return
    }

    // 等待确认
    buffer := make([]byte, 1024)
    n, _, err := conn.ReadFromUDP(buffer)
    if err != nil {
        fmt.Printf("Failed to read: %v\n", err)
        return
    }

    fmt.Printf("Received: %s\n", buffer[:n])
}
```

### 6. UDP服务端示例

```go
package main

import (
    "fmt"
    "net"
)

func main() {
    addr := &net.UDPAddr{
        IP:   net.ParseIP("0.0.0.0"),
        Port: 8080,
    }

    conn, err := net.ListenUDP("udp", addr)
    if err != nil {
        fmt.Printf("Failed to listen: %v\n", err)
        return
    }
    defer conn.Close()

    fmt.Println("UDP server listening on :8080")

    buffer := make([]byte, 1024)
    for {
        n, clientAddr, err := conn.ReadFromUDP(buffer)
        if err != nil {
            fmt.Printf("Failed to read: %v\n", err)
            continue
        }

        fmt.Printf("Received from %s: %s\n", clientAddr, buffer[:n])

        // 发送确认
        _, err = conn.WriteToUDP([]byte("ACK"), clientAddr)
        if err != nil {
            fmt.Printf("Failed to write: %v\n", err)
        }
    }
}
```

### 7. TCP与UDP对比

| 特性 | TCP | UDP |
|------|-----|-----|
| **连接性** | 面向连接 | 无连接 |
| **可靠性** | 可靠传输 | 不可靠 |
| **顺序性** | 保证顺序 | 不保证顺序 |
| **流量控制** | 滑动窗口 | 无 |
| **拥塞控制** | 有 | 无 |
| **头部开销** | 20-60字节 | 8字节 |
| **速度** | 较慢 | 较快 |
| **适用场景** | 文件传输、网页浏览 | 实时音视频、游戏 |

---

## 九、常见面试题

### 1. TCP三次握手的目的是什么？

**答案**：
- 确认双方的发送能力正常
- 确认双方的接收能力正常
- 同步双方的序列号
- 防止已失效的连接请求报文到达服务器

### 2. 为什么需要TIME_WAIT状态？

**答案**：
- 确保最后一个ACK能到达服务器
- 等待网络中残留的分组过期（2MSL时间）
- 防止新连接收到旧连接的分组

### 3. TCP和UDP的区别？

**答案**：

| 特性 | TCP | UDP |
|------|-----|-----|
| 连接性 | 面向连接 | 无连接 |
| 可靠性 | 可靠 | 不可靠 |
| 顺序性 | 保证顺序 | 不保证顺序 |
| 速度 | 较慢 | 较快 |
| 拥塞控制 | 有 | 无 |

### 4. TCP滑动窗口的作用？

**答案**：
- 实现流量控制
- 防止发送方发送过快导致接收方缓冲区溢出
- 提高网络利用率

### 5. TCP拥塞控制的过程？

**答案**：
1. **慢启动**：拥塞窗口指数增长
2. **拥塞避免**：拥塞窗口线性增长
3. **快速重传**：收到3个重复ACK后立即重传
4. **快速恢复**：从重传后恢复，线性增长

### 6. 什么是粘包？如何解决？

**答案**：
**粘包原因**：TCP是流式协议，数据无边界

**解决方法**：
- 固定长度协议
- 长度前缀协议
- 分隔符协议
- 应用层协议定义消息边界

### 7. 如何排查TCP连接问题？

**答案**：
1. 检查端口监听状态
2. 检查防火墙规则
3. 使用nc/telnet测试连通性
4. 抓包分析（tcpdump/wireshark）
5. 查看TCP状态和统计

### 8. 什么是SYN攻击？如何防范？

**答案**：
**SYN攻击**：攻击者发送大量SYN包但不完成三次握手，导致服务器半开连接队列满

**防范措施**：
- 启用SYN cookies
- 增加半开连接队列大小
- 使用防火墙限制SYN速率

### 9. TCP头部中Window字段的作用？

**答案**：
Window字段表示接收方的接收窗口大小，用于流量控制。发送方根据此字段确定最多能发送多少字节的数据而无需等待确认。

### 10. 为什么TCP需要确认号？

**答案**：
确认号用于告知发送方已成功接收的数据位置，发送方根据确认号确定哪些数据已被接收，哪些需要重传。

### 11. TCP重传机制有哪些？

**答案**：
1. **超时重传**：等待超时后重传
2. **快速重传**：收到3个重复ACK后立即重传

### 12. TCP四次挥手为什么需要四次？

**答案**：
因为TCP是全双工协议，双方需要分别关闭各自的发送通道。每一方都需要发送FIN表示自己不再发送数据，然后等待对方的ACK确认。

### 13. 什么是MSS？

**答案**：
MSS（Maximum Segment Size）是TCP允许的最大分段大小，不包括TCP头部。MSS通常等于MTU减去TCP和IP头部大小。

### 14. TCP选项有哪些？

**答案**：
- **MSS**：最大分段大小
- **Window Scale**：窗口缩放因子
- **SACK**：选择性确认
- **Timestamp**：时间戳

### 15. 如何优化TCP性能？

**答案**：
- 使用长连接减少连接建立开销
- 调整TCP参数（窗口大小、MSS等）
- 选择合适的拥塞控制算法
- 启用TCP选项（SACK、Window Scale等）

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

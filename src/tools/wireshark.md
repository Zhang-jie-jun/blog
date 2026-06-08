# Wireshark使用方法

## Wireshark是什么？
&emsp;&emsp;Wireshark是一款开源的网络协议分析工具，可以捕获和分析网络数据包，帮助诊断网络问题。

## 基本操作

### 捕获数据包

1. 选择网络接口：
    - 点击工具栏的接口列表
    - 选择要监控的网卡

2. 开始捕获：
    - 点击「开始捕获」按钮（鲨鱼鳍图标）
    - 或者按 `Ctrl+E`

3. 停止捕获：
    - 点击「停止捕获」按钮
    - 或者按 `Ctrl+E`

## 过滤器使用

### 显示过滤器

```bash
# 过滤特定IP地址
ip.addr == 192.168.1.1

# 过滤源IP
ip.src == 192.168.1.100

# 过滤目标IP
ip.dst == 192.168.1.100

# 过滤特定协议
tcp
udp
http
dns
icmp

# 过滤端口
tcp.port == 80
udp.port == 53

# 组合过滤
tcp.port == 80 && ip.src == 192.168.1.0/24

# 包含特定字符串
http contains "GET"
```

### 捕获过滤器

```bash
# 捕获特定协议
tcp
udp
host 192.168.1.1

# 捕获特定端口
port 80
portrange 1-1024

# 捕获特定IP
src host 192.168.1.100
dst host 192.168.1.100

# 组合捕获
tcp and port 80
```

## 常用协议分析

### HTTP分析

1. 过滤HTTP流量：
    ```bash
    http
    ```

2. 查看请求详情：
    - 选择HTTP数据包
    - 展开「Hypertext Transfer Protocol」

3. 常用HTTP过滤：
    ```bash
    http.request.method == "GET"
    http.request.method == "POST"
    http.response.code == 200
    http.host contains "example.com"
    ```

### TCP分析

1. 查看TCP三次握手：
    - SYN: `tcp.flags.syn == 1 && tcp.flags.ack == 0`
    - SYN+ACK: `tcp.flags.syn == 1 && tcp.flags.ack == 1`
    - ACK: `tcp.flags.ack == 1`

2. 分析TCP序列号和确认号

### DNS分析

```bash
# 过滤DNS
dns

# 过滤DNS查询
dns.qry.type == A
dns.qry.type == AAAA
dns.qry.type == CNAME
```

## 统计功能

### 协议分层统计
- 菜单：统计 → 协议分层

### 会话统计
- 菜单：统计 → 会话

### 端点统计
- 菜单：统计 → 端点

### IO图形
- 菜单：统计 → IO图形

## 保存和导出

### 保存捕获文件
- 文件 → 保存
- 支持格式：pcap, pcapng

### 导出特定数据包
- 选择数据包
- 文件 → 导出指定分组

### 导出为其他格式
- 文件 → 导出分组解析结果
- 支持：CSV、XML、JSON等

## 实用技巧

### 颜色规则
- 编辑 → 首选项 → 颜色规则
- 自定义不同协议的显示颜色

### 时间格式
- 视图 → 时间显示格式
- 选择合适的时间精度

### 追踪TCP流
- 右键数据包 → 追踪 → TCP流
- 查看完整的TCP会话内容

### 专家信息
- 分析 → 专家信息
- 查看警告和错误信息

## 常见问题

### 无法捕获数据包
- 确保以管理员身份运行
- 确认网卡选择正确
- 检查防火墙设置

### 显示乱码
- 在数据包详情中选择正确的字符编码

### 捕获文件过大
- 使用捕获过滤器限制捕获范围
- 定期保存并清空缓冲区

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

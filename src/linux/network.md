# 网络管理及故障诊断

## 概述

网络管理是Linux系统管理的重要部分，包括网络配置、连接测试、故障排查等。本文介绍常用的网络命令和故障诊断方法。

---

## 一、网络配置命令

### 1. ip命令

```bash
# 查看网络接口
ip addr

# 查看路由表
ip route

# 查看邻居（ARP缓存）
ip neigh

# 启用/禁用接口
sudo ip link set eth0 up
sudo ip link set eth0 down

# 设置IP地址
sudo ip addr add 192.168.1.100/24 dev eth0

# 添加路由
sudo ip route add 10.0.0.0/8 via 192.168.1.1
```

### 2. ifconfig（旧版）

```bash
# 查看接口信息
ifconfig

# 设置IP地址
sudo ifconfig eth0 192.168.1.100 netmask 255.255.255.0

# 启用接口
sudo ifconfig eth0 up
```

### 3. hostname

```bash
# 查看主机名
hostname

# 设置临时主机名
sudo hostname myserver

# 设置永久主机名
sudo hostnamectl set-hostname myserver
```

---

## 二、网络测试命令

### 1. ping

```bash
# 测试连通性
ping google.com

# 指定次数
ping -c 4 google.com

# 指定间隔
ping -i 0.5 google.com

# 使用IPv6
ping6 google.com
```

### 2. traceroute

```bash
# 追踪路由
traceroute google.com

# 使用UDP（默认）
traceroute -U google.com

# 使用ICMP
traceroute -I google.com

# 简化版
mtr google.com
```

### 3. curl

```bash
# 获取网页内容
curl https://example.com

# 保存到文件
curl -o output.html https://example.com

# 下载文件
curl -O https://example.com/file.zip

# 发送POST请求
curl -X POST -d "name=test" https://example.com/api

# 查看请求头
curl -I https://example.com

# 跟随重定向
curl -L https://example.com

# Basic认证
curl -u username:password https://example.com/api

# Basic认证（从文件读取密码）
curl -u username:$(cat password.txt) https://example.com/api

# Bearer Token认证
curl -H "Authorization: Bearer token_string" https://example.com/api

# API Key认证
curl -H "X-API-Key: your_api_key" https://example.com/api

# Digest认证
curl --digest -u username:password https://example.com/api

# 使用SSL/TLS加密（强制HTTPS）
curl --ssl https://example.com

# 指定SSL版本
curl --ssl-version TLSv1.2 https://example.com

# 忽略SSL证书验证（不推荐）
curl -k https://example.com

# 客户端证书认证
curl --cert client.crt --key client.key https://example.com

# 发送JSON数据
curl -X POST -H "Content-Type: application/json" -d '{"name":"test"}' https://example.com/api

# 传递cookie
curl -b "session=abc123" https://example.com

# 保存cookie到文件
curl -c cookies.txt https://example.com

# 使用cookie文件
curl -b cookies.txt https://example.com

# 代理服务器
curl -x http://proxy:port https://example.com

# 代理认证
curl -x http://proxy:port -U proxyuser:proxypass https://example.com

# 限速
curl --limit-rate 100k https://example.com/file.zip

# 设置超时时间
curl --max-time 30 https://example.com

# 显示详细信息
curl -v https://example.com
```

### 4. wget

```bash
# 下载文件
wget https://example.com/file.zip

# 后台下载
wget -b https://example.com/file.zip

# 断点续传
wget -c https://example.com/file.zip

# 批量下载
wget -i urls.txt

# 限制速度
wget --limit-rate=100k https://example.com/file.zip
```

---

## 三、远程连接命令

### 1. ssh

```bash
# 基本连接
ssh user@hostname

# 指定端口
ssh -p 2222 user@hostname

# 使用密钥登录
ssh -i ~/.ssh/id_rsa user@hostname

# 执行远程命令
ssh user@hostname 'ls -la'

# 端口转发
ssh -L 8080:localhost:80 user@hostname

# 免密登录配置
ssh-keygen -t rsa
ssh-copy-id user@hostname
```

### 2. scp

```bash
# 复制文件到远程
scp file.txt user@hostname:/path/

# 复制目录到远程
scp -r directory user@hostname:/path/

# 从远程复制到本地
scp user@hostname:/path/file.txt .

# 指定端口
scp -P 2222 file.txt user@hostname:/path/
```

### 3. rsync

```bash
# 同步目录
rsync -av /local/path user@hostname:/remote/path

# 增量同步
rsync -av --delete /local/path user@hostname:/remote/path

# 本地同步
rsync -av /source /destination

# 通过SSH同步
rsync -av -e ssh /local/path user@hostname:/remote/path
```

---

## 四、防火墙配置

### iptables

```bash
# 查看规则
sudo iptables -L

# 允许SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# 允许HTTP
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# 允许HTTPS
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# 允许回环接口
sudo iptables -A INPUT -i lo -j ACCEPT

# 允许已建立的连接
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# 默认拒绝所有入站
sudo iptables -P INPUT DROP

# 保存规则
sudo iptables-save > /etc/iptables/rules.v4
```

---

## 五、DNS工具

### 1. dig

```bash
# 查询域名
dig example.com

# 查询指定记录类型
dig example.com A
dig example.com MX
dig example.com TXT

# 指定DNS服务器
dig @8.8.8.8 example.com

# 反向解析
dig -x 8.8.8.8
```

### 2. nslookup

```bash
# 查询域名
nslookup example.com

# 指定DNS服务器
nslookup example.com 8.8.8.8
```

---

## 六、故障诊断流程

### 步骤1：检查网络接口

```bash
ip addr
ip link
```

### 步骤2：检查路由

```bash
ip route
route -n
```

### 步骤3：测试连通性

```bash
ping 127.0.0.1      # 本地回环
ping 192.168.1.1   # 网关
ping 8.8.8.8       # 外网
ping google.com     # DNS
```

### 步骤4：检查DNS

```bash
cat /etc/resolv.conf
dig google.com
```

### 步骤5：检查端口

```bash
# 检查本地端口
ss -tlnp

# 检查远程端口
nc -zv localhost 80
```

### 步骤6：检查防火墙

```bash
sudo iptables -L
sudo ufw status
```

---

## 七、实战案例

### 案例1：无法访问外网

```bash
# 检查接口状态
ip addr show eth0

# 检查网关
ip route | grep default

# 测试网关连通性
ping 192.168.1.1

# 测试DNS
dig google.com
```

### 案例2：端口无法访问

```bash
# 检查本地监听
ss -tlnp | grep :80

# 检查防火墙
sudo iptables -L INPUT | grep 80

# 测试端口连通性
nc -zv localhost 80
```

---

## 八、面试题

**Q1：如何测试网络连通性？**
```bash
ping google.com
```

**Q2：如何查看路由表？**
```bash
ip route
route -n
```

**Q3：如何配置SSH免密登录？**
```bash
ssh-keygen -t rsa
ssh-copy-id user@hostname
```

**Q4：如何检查端口是否开放？**
```bash
ss -tlnp | grep :80
nc -zv localhost 80
```

**Q5：如何使用iptables开放80端口？**
```bash
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
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

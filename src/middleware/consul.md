# Consul详解

## Consul是什么？
&emsp;&emsp;Consul是一个服务网格解决方案，提供服务发现、健康检查、键值存储和自动分段功能。它是分布式系统中实现服务治理的重要工具。

## Consul的特点

| 特性 | 说明 |
|------|------|
| **服务发现** | 自动发现服务实例 |
| **健康检查** | 自动检测服务健康状态 |
| **键值存储** | 分布式配置管理 |
| **服务网格** | 支持Connect服务网格 |
| **多数据中心** | 支持跨数据中心部署 |

## 快速开始

### 安装Consul

```bash
# macOS
brew install consul

# Linux
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
echo "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install consul
```

### 启动Consul

```bash
# 开发模式启动
consul agent -dev

# 查看成员
consul members

# 查看服务
consul catalog services
```

## 服务注册

### 通过配置文件注册

```json
{
  "service": {
    "name": "web",
    "id": "web-1",
    "address": "127.0.0.1",
    "port": 8080,
    "checks": [
      {
        "http": "http://localhost:8080/health",
        "interval": "10s"
      }
    ]
  }
}
```

```bash
consul services register web.json
```

### 通过API注册

```bash
curl -X PUT -d '{
  "ID": "web-1",
  "Name": "web",
  "Address": "127.0.0.1",
  "Port": 8080,
  "Check": {
    "HTTP": "http://localhost:8080/health",
    "Interval": "10s"
  }
}' http://localhost:8500/v1/agent/service/register
```

## 服务发现

### 查询服务

```bash
# 查询所有服务
curl http://localhost:8500/v1/catalog/services

# 查询特定服务
curl http://localhost:8500/v1/catalog/service/web

# 查询健康服务
curl http://localhost:8500/v1/health/service/web
```

### DNS查询

```bash
dig @127.0.0.1 -p 8600 web.service.consul

# A记录
dig @127.0.0.1 -p 8600 web.service.consul A

# SRV记录
dig @127.0.0.1 -p 8600 web.service.consul SRV
```

## 键值存储

### 基本操作

```bash
# 设置键值
consul kv put config/database/url "mysql://localhost:3306/mydb"

# 获取键值
consul kv get config/database/url

# 删除键值
consul kv delete config/database/url

# 列出所有键
consul kv get -recurse
```

### 目录操作

```bash
# 创建目录
consul kv put config/app/ ""

# 递归获取目录
consul kv get -recurse config/app/

# 递归删除目录
consul kv delete -recurse config/app/
```

## Consul代码示例（Go）

### 安装依赖

```bash
go get github.com/hashicorp/consul/api
```

### 服务注册

```go
package main

import (
    "fmt"

    "github.com/hashicorp/consul/api"
)

func main() {
    // 创建客户端
    config := api.DefaultConfig()
    client, err := api.NewClient(config)
    if err != nil {
        panic(err)
    }

    // 注册服务
    registration := &api.AgentServiceRegistration{
        ID:      "web-1",
        Name:    "web",
        Address: "127.0.0.1",
        Port:    8080,
        Check: &api.AgentServiceCheck{
            HTTP:     "http://localhost:8080/health",
            Interval: "10s",
        },
    }

    err = client.Agent().ServiceRegister(registration)
    if err != nil {
        panic(err)
    }
    fmt.Println("Service registered successfully")
}
```

### 服务发现

```go
func DiscoverService(client *api.Client, serviceName string) ([]*api.ServiceEntry, error) {
    services, _, err := client.Health().Service(serviceName, "", true, nil)
    if err != nil {
        return nil, err
    }
    return services, nil
}

func main() {
    config := api.DefaultConfig()
    client, _ := api.NewClient(config)

    services, err := DiscoverService(client, "web")
    if err != nil {
        panic(err)
    }

    for _, service := range services {
        fmt.Printf("Service: %s, Address: %s:%d\n", 
            service.Service.ID,
            service.Service.Address,
            service.Service.Port)
    }
}
```

### 键值操作

```go
func main() {
    config := api.DefaultConfig()
    client, _ := api.NewClient(config)

    // 设置键值
    _, err := client.KV().Put(&api.KVPair{Key: "config/app/debug", Value: []byte("true")}, nil)
    if err != nil {
        panic(err)
    }

    // 获取键值
    pair, _, err := client.KV().Get("config/app/debug", nil)
    if err != nil {
        panic(err)
    }
    fmt.Printf("Value: %s\n", string(pair.Value))

    // 列出所有键
    pairs, _, err := client.KV().List("config/", nil)
    if err != nil {
        panic(err)
    }
    for _, p := range pairs {
        fmt.Printf("Key: %s, Value: %s\n", p.Key, string(p.Value))
    }
}
```

## Consul集群

### 启动服务器节点

```bash
# 第一个服务器节点
consul agent -server -bootstrap-expect=3 -data-dir=/tmp/consul -node=server1 -bind=192.168.1.101

# 第二个服务器节点
consul agent -server -join=192.168.1.101 -data-dir=/tmp/consul -node=server2 -bind=192.168.1.102

# 第三个服务器节点
consul agent -server -join=192.168.1.101 -data-dir=/tmp/consul -node=server3 -bind=192.168.1.103

# 客户端节点
consul agent -client -join=192.168.1.101 -data-dir=/tmp/consul -node=client1 -bind=192.168.1.110
```

## 一、Consul核心原理

### 1.1 Consul架构

```
                    ┌─────────────────────────────────────────────┐
                    │              客户端层                        │
                    │  (服务注册、健康检查、DNS查询、KV操作)        │
                    └───────────────────┬─────────────────────────┘
                                        │
        ┌───────────────────────────────┼───────────────────────────────┐
        │                               │                               │
        ▼                               ▼                               ▼
┌───────────────┐              ┌───────────────┐              ┌───────────────┐
│   Client1     │              │   Client2     │              │   Client3     │
│  (服务发现)    │              │  (配置获取)    │              │  (健康检查)    │
└───────────────┘              └───────────────┘              └───────────────┘
        │                               │                               │
        └───────────────────────────────┼───────────────────────────────┘
                                        │
                    ┌───────────────────▼─────────────────────────┐
                    │              服务器层                        │
                    │   (Raft共识、Leader选举、数据复制)           │
                    └───────────────────┬─────────────────────────┘
                                        │
        ┌───────────────────────────────┼───────────────────────────────┐
        │                               │                               │
        ▼                               ▼                               ▼
┌───────────────┐              ┌───────────────┐              ┌───────────────┐
│   Server1     │              │   Server2     │              │   Server3     │
│   (Leader)    │              │  (Follower)   │              │  (Follower)   │
│  Raft节点     │              │  Raft节点     │              │  Raft节点     │
└───────────────┘              └───────────────┘              └───────────────┘
        │                               │                               │
        └───────────────────────────────┴───────────────────────────────┘
                                        │
                    ┌───────────────────▼─────────────────────────┐
                    │              存储层                          │
                    │   (Raft Log、状态机、持久化存储)             │
                    └─────────────────────────────────────────────┘
```

### 1.2 Raft共识算法

**Raft角色：**
- **Leader**：负责接收客户端请求，复制日志到Follower
- **Follower**：被动接收Leader的日志复制
- **Candidate**：选举期间的临时角色

**选举流程：**
```
1. 节点超时 → 转为Candidate → 发起选举
2. 向其他节点发送Vote请求
3. 获得多数票 → 成为Leader
4. 定期发送心跳维持Leader地位
5. 若Leader宕机 → 重新选举
```

**日志复制：**
```
客户端 → Leader → 写入日志 → 复制到Follower → 多数确认 → 应用到状态机 → 返回成功
```

### 1.3 服务发现原理

**服务注册流程：**
```
1. 服务启动时向Consul Agent注册
2. Agent将注册信息发送给Server
3. Server通过Raft复制到所有节点
4. 其他服务通过DNS或HTTP查询发现服务
```

**健康检查机制：**
```
1. Agent定期执行健康检查（HTTP/TCP/脚本）
2. 检查失败标记服务为不健康
3. 自动从服务发现列表中移除
4. 检查恢复后重新加入
```

### 1.4 键值存储实现

**KV存储架构：**
```
┌─────────────────────────────────────────────────────────────┐
│                      KV Store                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │   Raft Log  │→ │  State Machine│→│  Memory DB  │         │
│  │  (持久化)   │  │  (复制状态)  │  │  (读写)     │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
└─────────────────────────────────────────────────────────────┘
```

**事务支持：**
```bash
consul kv txn -cas -key "config/version" -val "2" -prevval "1"
```

---

## 二、Consul常见问题分析与排查

### 问题1：服务注册失败

**现象：**
```
Error registering service: Put http://localhost:8500/v1/agent/service/register: dial tcp 127.0.0.1:8500: connect: connection refused
```

**排查步骤：**
```bash
# 1. 检查Consul Agent是否运行
consul members

# 2. 检查Agent状态
consul agent -status

# 3. 检查网络连接
curl http://localhost:8500/v1/agent/services
```

**解决方案：**
1. 确保Consul Agent已启动
2. 检查防火墙规则，确保8500端口可访问
3. 检查Agent配置的`client_addr`是否正确

---

### 问题2：服务健康检查失败

**现象：**
```
curl http://localhost:8500/v1/health/service/web
# 返回的服务状态为critical
```

**排查步骤：**
```bash
# 1. 查看健康检查状态
consul health checks

# 2. 手动执行健康检查
curl http://localhost:8080/health

# 3. 查看Agent日志
consul monitor
```

**解决方案：**
1. 确保健康检查端点正常响应
2. 调整检查间隔（interval）
3. 增加超时时间（timeout）

---

### 问题3：服务发现返回空列表

**现象：**
```bash
curl http://localhost:8500/v1/health/service/web
# 返回空数组
```

**排查步骤：**
```bash
# 1. 检查服务是否注册
consul catalog services

# 2. 检查服务实例
consul catalog service web

# 3. 检查健康状态
consul health service web
```

**解决方案：**
1. 确认服务已正确注册
2. 检查健康检查配置
3. 验证服务实例是否正常运行

---

### 问题4：Raft选举失败

**现象：**
```
[WARN]  raft: no known peers, aborting election
```

**排查步骤：**
```bash
# 1. 检查集群成员
consul members

# 2. 查看Leader状态
consul operator raft list-peers

# 3. 检查网络连通性
ping server2.example.com
```

**解决方案：**
1. 确保所有节点网络互通
2. 检查`bind_addr`配置
3. 使用`consul join`命令加入集群

---

### 问题5：Consul集群脑裂

**现象：**
```
[ERROR] raft: duplicate request from ...
```

**排查步骤：**
```bash
# 1. 查看集群状态
consul operator raft list-peers

# 2. 检查网络分区
ip addr show

# 3. 查看日志中的分裂信息
grep -i "split" /var/log/consul.log
```

**解决方案：**
1. 修复网络分区
2. 使用奇数个Server节点
3. 配置适当的`raft_protocol`版本

---

### 问题6：KV存储数据不一致

**现象：**
```bash
# 不同节点返回不同的值
consul kv get config/version  # 节点1返回"1"
consul kv get config/version  # 节点2返回"2"
```

**排查步骤：**
```bash
# 1. 检查Leader状态
consul operator raft list-peers

# 2. 检查日志复制状态
consul debug

# 3. 验证数据一致性
consul kv get -detailed config/version
```

**解决方案：**
1. 确保所有Follower能同步Leader日志
2. 检查网络延迟
3. 增加`raft_timeout`配置

---

### 问题7：DNS查询失败

**现象：**
```bash
dig @127.0.0.1 -p 8600 web.service.consul
# 返回空结果
```

**排查步骤：**
```bash
# 1. 检查DNS服务是否启用
consul config get dns

# 2. 测试DNS端口
telnet localhost 8600

# 3. 检查服务是否存在
consul catalog service web
```

**解决方案：**
1. 确保`client_addr`包含DNS监听地址
2. 检查防火墙规则
3. 验证服务已注册且健康

---

### 问题8：性能问题

**现象：**
- 服务发现延迟高
- KV操作响应慢
- CPU/内存使用率高

**排查步骤：**
```bash
# 1. 查看性能指标
consul monitor -log-level=trace

# 2. 检查内存使用
free -h

# 3. 检查CPU使用
top -p $(pgrep consul)
```

**解决方案：**
1. 增加Server节点资源
2. 优化健康检查频率
3. 使用本地缓存减少查询次数

---

### 问题9：配置同步失败

**现象：**
```bash
consul kv put config/app/debug true
# 但应用未获取到新配置
```

**排查步骤：**
```bash
# 1. 检查KV存储
consul kv get config/app/debug

# 2. 检查应用监听机制
curl http://localhost:8500/v1/kv/config/app/?recurse

# 3. 验证watch机制
consul watch -type=keyprefix -prefix=config/app/
```

**解决方案：**
1. 确保应用使用Consul Watch机制
2. 检查ACL权限配置
3. 增加配置变更通知

---

### 问题10：多数据中心通信失败

**现象：**
```
[ERROR] consul: failed to sync remote datacenter
```

**排查步骤：**
```bash
# 1. 检查WAN连接
consul members -wan

# 2. 检查数据中心配置
consul config get datacenter

# 3. 测试跨数据中心网络
ping remote-dc-server.example.com
```

**解决方案：**
1. 确保WAN网络连通
2. 配置正确的`datacenter`名称
3. 使用`consul join -wan`连接远程数据中心

---

## 三、Consul面试题

### 基础问题

**Q1：Consul是什么？有哪些核心功能？**

**A1：**
Consul是一个服务网格解决方案，核心功能包括：
- **服务发现**：自动发现服务实例
- **健康检查**：检测服务健康状态
- **键值存储**：分布式配置管理
- **服务网格**：Connect服务网格
- **多数据中心**：跨数据中心部署

---

**Q2：Consul的架构是怎样的？**

**A2：**
Consul采用客户端-服务器架构：
- **Server节点**：维护Raft共识，存储数据，处理查询
- **Client节点**：轻量级代理，转发请求到Server
- **数据中心**：多个数据中心可通过WAN连接

---

**Q3：Consul的服务发现是如何实现的？**

**A3：**
1. 服务通过Agent注册到Consul
2. Agent将注册信息同步到Server集群
3. Server通过Raft复制数据到所有节点
4. 客户端通过DNS或HTTP API查询服务

---

**Q4：Consul的健康检查有哪些类型？**

**A4：**
- **HTTP检查**：发送HTTP请求检查响应
- **TCP检查**：尝试建立TCP连接
- **脚本检查**：执行自定义脚本
- **TTL检查**：由应用定期上报健康状态

---

**Q5：Consul使用什么共识算法？**

**A5：**
Consul使用**Raft共识算法**，保证数据一致性和高可用性：
- Leader选举
- 日志复制
- 状态机复制

---

**Q6：Consul的KV存储有什么特点？**

**A6：**
- **分布式**：数据复制到所有Server节点
- **事务支持**：支持CAS（Compare-And-Swap）操作
- **层级结构**：支持目录和键值对
- **Watch机制**：监听键值变化

---

**Q7：Consul和其他服务发现工具的区别？**

**A7：**

| 工具 | 特点 |
|------|------|
| **Consul** | 功能全面（服务发现、健康检查、KV、服务网格） |
| **Eureka** | 简单易用，Spring Cloud集成好 |
| **etcd** | 专注KV存储，API简单 |
| **ZooKeeper** | 功能强大但复杂 |

---

**Q8：Consul的多数据中心是如何实现的？**

**A8：**
- 通过WAN连接实现跨数据中心通信
- 每个数据中心有独立的Raft集群
- 使用Gossip协议发现远程数据中心
- 支持跨数据中心服务发现

---

**Q9：Consul的ACL是什么？**

**A9：**
ACL（Access Control List）用于控制对Consul资源的访问：
- 定义规则控制服务注册、查询、KV操作等
- 支持令牌（Token）认证
- 可以限制特定服务或KV路径的访问权限

---

**Q10：Consul的Gossip协议有什么作用？**

**A10：**
Gossip协议用于：
- **成员发现**：自动发现集群成员
- **健康检测**：检测节点健康状态
- **事件传播**：传播集群事件

---

### 复杂场景问题

**Q11：如何设计高可用的Consul集群？**

**A11：**

**架构设计：**
```
              ┌──────────────────────────────────────────┐
              │              负载均衡层                  │
              │         (Nginx/Consul DNS)              │
              └──────────────────┬─────────────────────┘
                                 │
        ┌────────────────────────┼────────────────────────┐
        ▼                        ▼                        ▼
┌─────────────┐          ┌─────────────┐          ┌─────────────┐
│   Client1   │          │   Client2   │          │   Client3   │
│   (本地)    │          │   (本地)    │          │   (本地)    │
└─────────────┘          └─────────────┘          └─────────────┘
        │                        │                        │
        └────────────────────────┼────────────────────────┘
                                 │
        ┌────────────────────────┼────────────────────────┐
        ▼                        ▼                        ▼
┌─────────────┐          ┌─────────────┐          ┌─────────────┐
│   Server1   │          │   Server2   │          │   Server3   │
│  (Leader)   │          │  (Follower) │          │  (Follower) │
└─────────────┘          └─────────────┘          └─────────────┘
```

**关键要点：**
1. **奇数个Server节点**（推荐3或5个）
2. **多可用区部署**：分散到不同机房
3. **Client节点本地化**：每个应用节点部署Client
4. **定期备份**：使用`consul snapshot`

---

**Q12：如何处理Consul的服务发现一致性问题？**

**A12：**

**问题分析：**
- 服务注册/注销可能存在延迟
- 网络分区可能导致数据不一致
- 健康检查状态可能不同步

**解决方案：**
```go
// 使用强一致性查询
func GetService(client *api.Client, name string) ([]*api.ServiceEntry, error) {
    // 使用consistent参数确保从Leader获取数据
    queryOpts := &api.QueryOptions{
        Consistency: api.Consistent,
    }
    services, _, err := client.Health().Service(name, "", true, queryOpts)
    return services, err
}
```

**策略：**
1. 使用`Consistent`查询模式
2. 实现服务发现重试机制
3. 结合健康检查过滤不健康实例

---

**Q13：如何实现Consul的配置热更新？**

**A13：**

**方案：使用Consul Watch**
```go
func WatchConfig(client *api.Client, prefix string, callback func(map[string]string)) {
    params := &api.QueryOptions{
        WaitIndex: 0,
    }
    
    for {
        pairs, meta, err := client.KV().List(prefix, params)
        if err != nil {
            time.Sleep(1 * time.Second)
            continue
        }
        
        params.WaitIndex = meta.LastIndex
        
        config := make(map[string]string)
        for _, pair := range pairs {
            config[pair.Key] = string(pair.Value)
        }
        
        callback(config)
    }
}
```

**使用示例：**
```go
WatchConfig(client, "config/app/", func(config map[string]string) {
    // 重新加载配置
    reloadConfig(config)
})
```

---

**Q14：如何处理Consul的性能瓶颈？**

**A14：**

**性能优化策略：**

1. **增加Client节点**：分散查询压力
2. **本地缓存**：减少Consul查询次数
```go
type Cache struct {
    services map[string][]*api.ServiceEntry
    mu       sync.RWMutex
}

func (c *Cache) GetServices(name string) ([]*api.ServiceEntry, bool) {
    c.mu.RLock()
    defer c.mu.RUnlock()
    services, ok := c.services[name]
    return services, ok
}
```

3. **优化健康检查**：减少检查频率
4. **水平扩展Server节点**：增加处理能力

---

**Q15：如何实现Consul的分布式锁？**

**A15：**

**方案：使用KV的CAS操作**
```go
func AcquireLock(client *api.Client, lockKey string, sessionTTL string) (string, error) {
    // 创建会话
    session, _, err := client.Session().Create(&api.SessionEntry{
        TTL: sessionTTL,
    }, nil)
    if err != nil {
        return "", err
    }
    
    // CAS获取锁
    _, err = client.KV().Put(&api.KVPair{
        Key:     lockKey,
        Value:   []byte(session),
        Session: session,
    }, nil)
    
    if err != nil {
        client.Session().Destroy(session, nil)
        return "", err
    }
    
    return session, nil
}

func ReleaseLock(client *api.Client, lockKey string, session string) error {
    // 只有持有锁的会话才能释放
    pair, _, err := client.KV().Get(lockKey, nil)
    if err != nil {
        return err
    }
    
    if string(pair.Value) != session {
        return errors.New("not owner of lock")
    }
    
    _, err = client.KV().Delete(lockKey, nil)
    if err != nil {
        return err
    }
    
    return client.Session().Destroy(session, nil)
}
```

---

**Q16：如何设计Consul的多数据中心架构？**

**A16：**

**架构设计：**
```
              DC1                        DC2
    ┌───────────────┐          ┌───────────────┐
    │   Server1     │          │   Server4     │
    │   Server2     │          │   Server5     │
    │   Server3     │          │   Server6     │
    └───────┬───────┘          └───────┬───────┘
            │                          │
            └───────────┬──────────────┘
                        │ WAN连接
                        ▼
              ┌───────────────┐
              │   WAN Join    │
              └───────────────┘
```

**配置步骤：**
```bash
# DC1配置
consul agent -server -datacenter=dc1 -bind=192.168.1.10

# DC2配置
consul agent -server -datacenter=dc2 -bind=192.168.2.10

# WAN连接
consul join -wan 192.168.1.10
```

**关键要点：**
1. 每个数据中心独立Raft集群
2. 使用WAN连接实现跨数据中心通信
3. 配置适当的`retry_join_wan`

---

**Q17：如何处理Consul的脑裂问题？**

**A17：**

**问题分析：**
- 网络分区导致多个Leader
- 数据写入冲突
- 服务发现不一致

**解决方案：**

1. **使用奇数个Server节点**：确保多数派选举
2. **配置合适的超时时间**：
```hcl
raft_timeout = "10s"
```

3. **监控脑裂状态**：
```bash
# 检测多个Leader
consul operator raft list-peers
```

4. **手动干预**：
```bash
# 强制移除故障节点
consul operator raft remove-peer -peer-id=node-id
```

---

**Q18：如何实现Consul的服务网格？**

**A18：**

**方案：使用Consul Connect**

**步骤1：启用Connect**
```hcl
connect {
  enabled = true
}
```

**步骤2：配置服务网格**
```json
{
  "service": {
    "name": "web",
    "connect": {
      "sidecar_service": {
        "proxy": {
          "upstreams": [
            {
              "destination_name": "api",
              "local_bind_port": 9090
            }
          ]
        }
      }
    }
  }
}
```

**步骤3：启动Sidecar**
```bash
consul connect envoy -sidecar-for web -admin-bind 127.0.0.1:19000
```

**优势：**
- 自动服务间加密
- 透明的服务发现
- 流量控制和熔断

---

**Q19：如何进行Consul的监控和告警？**

**A19：**

**监控指标：**
| 指标 | 说明 | 告警阈值 |
|------|------|----------|
| `raft.leader` | Leader状态 | 无Leader时告警 |
| `consul.catalog.services.count` | 服务数量 | 异常增减时告警 |
| `consul.health.service.unhealthy` | 不健康服务数 | > 0时告警 |
| `consul.member.count` | 集群成员数 | 少于预期时告警 |

**监控方案：**
```yaml
# Prometheus配置
scrape_configs:
  - job_name: 'consul'
    consul_sd_configs:
      - server: 'localhost:8500'
        services: ['consul']
    metrics_path: '/v1/agent/metrics'
    params:
      format: ['prometheus']
```

**告警规则：**
```yaml
groups:
- name: consul.rules
  rules:
  - alert: ConsulNoLeader
    expr: consul_raft_leader == 0
    for: 5m
    labels:
      severity: critical
```

---

**Q20：如何进行Consul的备份和恢复？**

**A20：**

**备份命令：**
```bash
# 创建快照
consul snapshot save backup.snap

# 查看快照信息
consul snapshot inspect backup.snap
```

**恢复命令：**
```bash
# 停止所有Consul节点
systemctl stop consul

# 在Leader节点恢复
consul snapshot restore backup.snap

# 重启所有节点
systemctl start consul
```

**备份策略：**
```bash
# 定期备份（每天凌晨2点）
0 2 * * * consul snapshot save /backup/consul_$(date +\%Y\%m\%d).snap

# 保留最近7天备份
find /backup -name "consul_*.snap" -mtime +7 -delete
```

**注意事项：**
- 备份必须在Leader节点执行
- 恢复时所有节点必须停止
- 定期测试恢复流程

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

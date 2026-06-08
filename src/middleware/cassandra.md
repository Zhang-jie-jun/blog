# Cassandra详解

## Cassandra是什么？
&emsp;&emsp;Cassandra是一个分布式NoSQL数据库，设计用于处理大规模数据。它提供高可用性和线性可扩展性，是Facebook开发的开源项目。

## Cassandra的特点

| 特性 | 说明 |
|------|------|
| **分布式** | 无中心节点设计 |
| **高可用** | 多副本数据存储 |
| **线性扩展** | 轻松扩展节点 |
| **容错性** | 自动故障恢复 |
| **高性能** | 读写性能优秀 |

---

## 一、Cassandra核心原理

### 1.1 分布式架构

**架构特点：**

```
┌────────────────────────────────────────────────────────────────┐
│                      Cassandra Cluster                         │
├───────┬───────┬───────┬───────┬───────┬───────┬───────┬───────┤
│ Node1 │ Node2 │ Node3 │ Node4 │ Node5 │ Node6 │ Node7 │ Node8 │
│  DC1  │  DC1  │  DC1  │  DC1  │  DC2  │  DC2  │  DC2  │  DC2  │
└───┬───┴───┬───┴───┬───┴───┬───┴───┬───┴───┬───┴───┬───┴───┬───┘
    │       │       │       │       │       │       │       │
    └───────┴───────┴───────┴───────┴───────┴───────┴───────┘
            Gossip协议通信
```

**Gossip协议：**

```go
type GossipState struct {
    NodeID       string
    Heartbeat    int64
    SchemaVersion string
    TokenRange   []Token
}

func (n *Node) Gossip() {
    // 随机选择节点交换状态
    peer := selectRandomPeer()
    exchangeState(peer)
    
    // 更新本地状态
    updateState()
}
```

### 1.2 数据分布

**一致性哈希与Token Ring：**

```go
type TokenRing struct {
    Tokens []Token
    Nodes  map[Token]Node
}

func (r *TokenRing) GetNodeForKey(key string) Node {
    // 计算token
    token := murmur3.Hash32(key)
    
    // 找到负责该token的节点
    for i, t := range r.Tokens {
        if token <= t {
            return r.Nodes[t]
        }
    }
    
    return r.Nodes[r.Tokens[0]]
}
```

**数据复制策略：**

```cql
-- SimpleStrategy（单数据中心）
CREATE KEYSPACE mykeyspace 
WITH replication = {'class': 'SimpleStrategy', 'replication_factor': '3'};

-- NetworkTopologyStrategy（多数据中心）
CREATE KEYSPACE mykeyspace 
WITH replication = {
    'class': 'NetworkTopologyStrategy', 
    'datacenter1': '3',
    'datacenter2': '2'
};
```

### 1.3 一致性级别

**一致性级别配置：**

```go
type ConsistencyLevel int

const (
    ANY ConsistencyLevel = iota
    ONE
    TWO
    THREE
    QUORUM
    ALL
    LOCAL_QUORUM
    EACH_QUORUM
    SERIAL
    LOCAL_SERIAL
)

func (c *Client) QueryWithConsistency(query string, cl ConsistencyLevel) {
    c.session.Query(query).Consistency(cl).Exec()
}
```

**一致性与可用性权衡：**

```
一致性级别越高 → 可用性越低 → 延迟越高
一致性级别越低 → 可用性越高 → 延迟越低

QUORUM: (replication_factor / 2) + 1
```

---

## 二、Cassandra高级特性

### 2.1 数据模型

**主键设计：**

```cql
-- 简单主键
CREATE TABLE users (
    id UUID PRIMARY KEY
);

-- 复合主键（分区键+聚类键）
CREATE TABLE user_posts (
    user_id UUID,
    post_id UUID,
    title TEXT,
    content TEXT,
    created_at TIMESTAMP,
    PRIMARY KEY (user_id, post_id)
) WITH CLUSTERING ORDER BY (post_id DESC);
```

**数据类型：**

| 类型 | 说明 |
|------|------|
| TEXT | 字符串 |
| INT/BIGINT | 整数 |
| UUID | 唯一标识符 |
| TIMESTAMP | 时间戳 |
| BOOLEAN | 布尔值 |
| LIST | 列表 |
| MAP | 映射 |
| SET | 集合 |

### 2.2 批处理

**批量操作：**

```cql
BEGIN BATCH
    INSERT INTO users (id, name, email) VALUES (uuid(), 'John', 'john@example.com');
    INSERT INTO users (id, name, email) VALUES (uuid(), 'Jane', 'jane@example.com');
APPLY BATCH;
```

**条件批处理：**

```cql
BEGIN BATCH
    UPDATE users SET name = 'John Updated' WHERE id = <user_id> IF EXISTS;
    DELETE FROM posts WHERE user_id = <user_id> IF EXISTS;
APPLY BATCH;
```

### 2.3 物化视图

**物化视图创建：**

```cql
CREATE MATERIALIZED VIEW user_by_email AS
    SELECT id, name, email
    FROM users
    WHERE email IS NOT NULL
    PRIMARY KEY (email, id);
```

---

## 三、Cassandra常见问题分析与排查

### 问题1：读写性能慢

**现象：**
查询或写入操作响应时间过长。

**排查步骤：**

```bash
# 查看节点状态
nodetool status

# 查看查询性能
nodetool tpstats

# 查看磁盘IO
iostat -x 1
```

**解决方案：**

```bash
# 调整读写一致性级别
cqlsh> CONSISTENCY QUORUM;

# 添加索引
CREATE INDEX idx_users_email ON users(email);

# 优化数据模型
# 使用复合主键减少查询范围
```

---

### 问题2：数据不一致

**现象：**
不同节点返回的数据不一致。

**排查步骤：**

```bash
# 检查副本状态
nodetool status

# 检查数据同步
nodetool repair

# 查看日志
cat /var/log/cassandra/system.log | grep -i "repair"
```

**解决方案：**

```bash
# 手动修复
nodetool repair mykeyspace

# 配置自动修复
# cassandra.yaml
auto_snapshot: true
repair_interval: 7200
```

---

### 问题3：节点故障

**现象：**
部分节点从集群中失联。

**排查步骤：**

```bash
# 查看节点状态
nodetool status

# 检查Gossip状态
nodetool gossipinfo

# 检查网络连通性
ping node2
```

**解决方案：**

```bash
# 重启故障节点
systemctl restart cassandra

# 移除故障节点
nodetool decommission

# 添加新节点
# 在新节点上配置cassandra.yaml后启动
```

---

### 问题4：内存不足

**现象：**
JVM内存不足，导致OOM错误。

**排查步骤：**

```bash
# 查看JVM内存
nodetool info

# 查看堆内存配置
cat /etc/cassandra/jvm.options
```

**解决方案：**

```bash
# 修改JVM配置
# jvm.options
-Xms8g
-Xmx8g

# 调整缓存大小
# cassandra.yaml
key_cache_size_in_mb: 512
row_cache_size_in_mb: 256
```

---

### 问题5：磁盘空间不足

**现象：**
磁盘使用超过阈值，写入被拒绝。

**排查步骤：**

```bash
# 查看磁盘使用
df -h

# 查看数据目录大小
du -sh /var/lib/cassandra/data/
```

**解决方案：**

```bash
# 清理旧数据
nodetool truncate mykeyspace.users

# 添加新磁盘
# 修改cassandra.yaml中的data_file_directories
```

---

### 问题6：查询超时

**现象：**
查询操作超时失败。

**排查步骤：**

```bash
# 查看超时配置
cat /etc/cassandra/cassandra.yaml | grep timeout

# 查看查询日志
cat /var/log/cassandra/debug.log | grep -i "timeout"
```

**解决方案：**

```bash
# 调整超时设置
# cassandra.yaml
read_request_timeout_in_ms: 10000
write_request_timeout_in_ms: 10000

# 使用分页查询
SELECT * FROM users LIMIT 1000;
```

---

### 问题7：索引失效

**现象：**
查询使用索引时性能没有提升。

**排查步骤：**

```bash
# 查看索引状态
nodetool indexstatus

# 检查查询计划
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';
```

**解决方案：**

```bash
# 重建索引
DROP INDEX idx_users_email;
CREATE INDEX idx_users_email ON users(email);

# 使用合适的查询条件
# 查询必须包含分区键
```

---

### 问题8：数据写入失败

**现象：**
写入数据时出现错误。

**排查步骤：**

```bash
# 检查节点状态
nodetool status

# 检查副本数
cqlsh> DESCRIBE KEYSPACE mykeyspace;

# 查看日志
cat /var/log/cassandra/system.log | grep -i "error"
```

**解决方案：**

```bash
# 降低一致性级别
cqlsh> CONSISTENCY ONE;

# 等待节点恢复
# 检查网络连接
```

---

### 问题9：Token Ring不平衡

**现象：**
数据分布不均匀，部分节点负载过高。

**排查步骤：**

```bash
# 查看Token分布
nodetool ring

# 查看节点负载
nodetool status
```

**解决方案：**

```bash
# 重新平衡Token
nodetool move <token>

# 添加新节点分担负载
```

---

### 问题10：GC暂停

**现象：**
垃圾回收导致服务暂停。

**排查步骤：**

```bash
# 查看GC日志
cat /var/log/cassandra/gc.log | grep -i "pause"

# 查看JVM状态
jstat -gc <pid> 1000
```

**解决方案：**

```bash
# 调整GC配置
# jvm.options
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200

# 增加内存
-Xms16g
-Xmx16g
```

---

## 四、Cassandra面试题

### 基础问题

**Q1：Cassandra是什么？有什么特点？**

**A1：**
Cassandra是分布式NoSQL数据库，特点：
- 无中心节点设计
- 高可用性和容错性
- 线性扩展性
- 支持多数据中心部署
- 灵活的一致性级别

---

**Q2：Cassandra的数据分布是如何实现的？**

**A2：**
使用一致性哈希和Token Ring实现数据分布。每个节点负责一定范围的Token，数据根据Token分配到对应的节点。

---

**Q3：Cassandra的一致性级别有哪些？**

**A3：**
- ANY：只要有一个节点接收即可
- ONE/TWO/THREE：需要1/2/3个节点确认
- QUORUM：需要多数派确认
- ALL：需要所有节点确认
- LOCAL_QUORUM：本地数据中心多数派
- EACH_QUORUM：每个数据中心多数派

---

**Q4：Cassandra的复制策略有哪些？**

**A4：**
- SimpleStrategy：单数据中心复制
- NetworkTopologyStrategy：多数据中心复制

---

**Q5：Cassandra的主键设计有什么特点？**

**A5：**
- 主键由分区键和聚类键组成
- 分区键决定数据分布到哪个节点
- 聚类键决定同一分区内的数据排序

---

**Q6：Cassandra如何实现高可用性？**

**A6：**
- 多副本复制
- 自动故障检测和恢复
- Gossip协议维护集群状态
- 无单点故障设计

---

**Q7：Cassandra的Gossip协议是什么？**

**A7：**
Gossip协议是Cassandra用于节点间状态同步的协议。节点定期交换状态信息，维护集群视图。

---

**Q8：Cassandra如何处理数据备份？**

**A8：**

```bash
# 创建快照
nodetool snapshot mykeyspace

# 备份快照
cp -r /var/lib/cassandra/data/mykeyspace /backup/

# 恢复快照
nodetool restore mykeyspace /backup/mykeyspace
```

---

**Q9：Cassandra支持哪些数据类型？**

**A9：**
TEXT、INT、BIGINT、UUID、TIMESTAMP、BOOLEAN、LIST、MAP、SET等。

---

**Q10：Cassandra的批处理如何使用？**

**A10：**

```cql
BEGIN BATCH
    INSERT INTO users (id, name) VALUES (uuid(), 'John');
    UPDATE posts SET views = views + 1 WHERE post_id = <id>;
APPLY BATCH;
```

---

### 复杂场景问题

**Q11：如何设计Cassandra的数据模型？**

**A11：**

**设计原则：**

```cql
-- 根据查询模式设计表
-- 用户时间线查询
CREATE TABLE user_timeline (
    user_id UUID,
    post_id UUID,
    content TEXT,
    created_at TIMESTAMP,
    PRIMARY KEY (user_id, created_at)
) WITH CLUSTERING ORDER BY (created_at DESC);

-- 热门帖子查询
CREATE TABLE popular_posts (
    category TEXT,
    post_id UUID,
    views INT,
    PRIMARY KEY (category, views)
) WITH CLUSTERING ORDER BY (views DESC);
```

**设计要点：**
- 根据查询模式设计表结构
- 使用复合主键优化查询
- 避免大范围查询

---

**Q12：如何处理Cassandra的数据一致性问题？**

**A12：**

**一致性策略：**

```go
// 根据业务需求选择一致性级别
func GetUserWithConsistency(client *gocql.Session, id uuid.UUID, cl gocql.Consistency) (*User, error) {
    var user User
    err := client.Query(`SELECT * FROM users WHERE id = ?`, id).
        Consistency(cl).
        Scan(&user.ID, &user.Name, &user.Email)
    return &user, err
}

// 高一致性场景
user, err := GetUserWithConsistency(session, id, gocql.Quorum)

// 高可用场景
user, err := GetUserWithConsistency(session, id, gocql.One)
```

---

**Q13：如何设计Cassandra的多数据中心部署？**

**A13：**

**多数据中心架构：**

```
┌─────────────────┐         ┌─────────────────┐
│   DataCenter 1  │         │   DataCenter 2  │
├───────┬─────────┤         ├───────┬─────────┤
│ node1 │ node2   │   WAN   │ node3 │ node4   │
│       │ node3   │─────────│ node5 │ node6   │
└───────┴─────────┘         └───────┴─────────┘
```

**配置要点：**

```cql
CREATE KEYSPACE mykeyspace 
WITH replication = {
    'class': 'NetworkTopologyStrategy',
    'datacenter1': '3',
    'datacenter2': '2'
};
```

**注意事项：**
- 使用NetworkTopologyStrategy
- 根据数据中心重要性配置副本数
- 监控跨数据中心延迟

---

**Q14：如何优化Cassandra的查询性能？**

**A14：**

**优化策略：**

```bash
# 添加索引
CREATE INDEX idx_users_email ON users(email);

# 使用复合主键
CREATE TABLE user_posts (
    user_id UUID,
    post_id UUID,
    PRIMARY KEY (user_id, post_id)
);

# 限制查询范围
SELECT * FROM users WHERE user_id = ? LIMIT 100;

# 使用物化视图
CREATE MATERIALIZED VIEW user_by_email AS
    SELECT id, name, email
    FROM users
    WHERE email IS NOT NULL
    PRIMARY KEY (email, id);
```

---

**Q15：如何处理Cassandra的节点故障？**

**A15：**

**故障处理流程：**

```bash
# 1. 检测故障节点
nodetool status

# 2. 如果节点可恢复
systemctl restart cassandra

# 3. 如果节点不可恢复
nodetool decommission

# 4. 添加新节点
# 配置cassandra.yaml
seed_provider:
    - class_name: org.apache.cassandra.locator.SimpleSeedProvider
      parameters:
          - seeds: "node1,node2"

# 5. 等待数据同步
nodetool status
```

---

**Q16：如何设计Cassandra的监控告警系统？**

**A16：**

**监控指标：**

```bash
# 节点状态
nodetool status

# 性能指标
nodetool tpstats

# 存储使用
nodetool info

# 副本状态
nodetool ring
```

**告警规则：**
- 节点离线
- 副本数不足
- 读写超时频繁
- GC暂停过长

---

**Q17：如何实现Cassandra的数据迁移？**

**A17：**

**数据迁移方案：**

```bash
# 使用sstableloader迁移数据
sstableloader -d node1,node2 /path/to/sstables

# 使用cqlsh导出导入
cqlsh -e "COPY users TO 'users.csv'"
cqlsh -e "COPY users FROM 'users.csv'"

# 滚动迁移
# 1. 添加新节点到集群
# 2. 等待数据同步
# 3. 移除旧节点
```

---

**Q18：如何处理Cassandra的并发写入冲突？**

**A18：**

**并发处理方案：**

```cql
-- 使用轻量级事务
INSERT INTO users (id, name) VALUES (<id>, 'John') IF NOT EXISTS;

UPDATE users SET name = 'John' WHERE id = <id> IF name = 'OldName';
```

**时间戳解决方案：**

```go
// 使用客户端时间戳
client.Query(`INSERT INTO users (id, name) VALUES (?, ?)`, id, name).
    WithTimestamp(time.Now().UnixNano()).
    Exec()
```

---

**Q19：如何设计Cassandra的灾备方案？**

**A19：**

**灾备策略：**

```bash
# 跨数据中心复制
CREATE KEYSPACE mykeyspace 
WITH replication = {
    'class': 'NetworkTopologyStrategy',
    'primary_dc': '3',
    'backup_dc': '2'
};

# 定期快照
nodetool snapshot mykeyspace
cp -r /var/lib/cassandra/data/mykeyspace /backup/

# 增量备份
nodetool incrementalbackup
```

---

**Q20：如何处理Cassandra的大数据量查询？**

**A20：**

**大数据量查询方案：**

```bash
# 使用分页查询
SELECT * FROM users LIMIT 1000;

# 使用Token范围查询
SELECT * FROM users WHERE token(id) > token('start') AND token(id) < token('end');

# 使用Spark进行大规模数据分析
spark.read.format("org.apache.spark.sql.cassandra").
    options(table="users", keyspace="mykeyspace").
    load().show()
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

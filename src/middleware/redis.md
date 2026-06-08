# Redis详解

## Redis是什么？
&emsp;&emsp;Redis（Remote Dictionary Server）是一个开源的内存数据结构存储系统，可用作数据库、缓存和消息中间件。它支持多种数据结构，如字符串、哈希、列表、集合、有序集合等。

## Redis的特点

| 特性 | 说明 |
|------|------|
| **内存存储** | 数据存储在内存中，读写速度极快 |
| **持久化** | 支持RDB和AOF两种持久化方式 |
| **数据结构丰富** | 支持字符串、哈希、列表、集合、有序集合等 |
| **高可用** | 支持主从复制、哨兵模式、集群模式 |
| **原子操作** | 所有操作都是原子性的 |
| **发布订阅** | 支持消息发布订阅功能 |

---

## 一、Redis数据结构原理

### 1.1 字符串（String）

**底层实现**：简单动态字符串（SDS）

```
+--------+--------+-----------+---------+
| len    | alloc  | flags     | buf[]   |
| (4/8)  | (4/8)  | (1/5)     | 字节数组 |
+--------+--------+-----------+---------+
```

**特点**：
- 预分配空间，减少内存分配次数
- 惰性释放，避免频繁内存分配
- 二进制安全

**常用命令**：
```bash
SET key value
GET key
INCR counter
DECR counter
APPEND key value
```

### 1.2 哈希（Hash）

**底层实现**：压缩列表（ziplist）或哈希表（hashtable）

```
# 压缩列表结构
+-------+-------+-------+-------+-------+-------+-------+
| zlbytes| zltail| zllen | entry1| entry2| ...   | zlend |
+-------+-------+-------+-------+-------+-------+-------+

# 哈希表结构
+-----------+-----------+
|  bucket0  |  bucket1  |
| 指针→节点 | 指针→节点 |
+-----------+-----------+
```

**常用命令**：
```bash
HSET user:1 name "John" age 30
HGET user:1 name
HGETALL user:1
HKEYS user:1
HVALS user:1
```

### 1.3 列表（List）

**底层实现**：压缩列表（ziplist）或双向链表（linkedlist）

```
head → node1 ←→ node2 ←→ node3 ←→ tail
```

**常用命令**：
```bash
LPUSH tasks "task1" "task2"
RPUSH tasks "task3"
LPOP tasks
RPOP tasks
LRANGE tasks 0 -1
```

### 1.4 集合（Set）

**底层实现**：整数集合（intset）或哈希表（hashtable）

**常用命令**：
```bash
SADD users "John" "Jane"
SMEMBERS users
SISMEMBER users "John"
SINTER set1 set2
SUNION set1 set2
```

### 1.5 有序集合（Sorted Set）

**底层实现**：压缩列表（ziplist）或跳跃表（skiplist）+ 哈希表

```
# 跳跃表结构
level 3:  header ----------------> 100
level 2:  header ----> 30 ------> 100
level 1:  header -> 10 -> 30 -> 50 -> 100
level 0:  header -> 10 -> 20 -> 30 -> 40 -> 50 -> ... -> 100
```

**常用命令**：
```bash
ZADD scores 95 "John" 88 "Jane"
ZRANK scores "John"
ZSCORE scores "John"
ZRANGEBYSCORE scores 80 100
```

---

## 二、Redis持久化机制

### 2.1 RDB（快照持久化）

**原理**：定期将内存中的数据快照写入磁盘

**配置**：
```conf
# 900秒内至少1次写入
save 900 1
# 300秒内至少10次写入
save 300 10
# 60秒内至少10000次写入
save 60 10000
```

**手动触发**：
```bash
SAVE    # 同步保存（阻塞）
BGSAVE  # 异步保存（fork子进程）
```

**RDB工作流程**：
```
1. Redis执行BGSAVE命令
2. fork子进程（写时复制）
3. 子进程遍历内存数据
4. 将数据写入临时RDB文件
5. 完成后替换旧RDB文件
```

**优缺点**：
| 优点 | 缺点 |
|------|------|
| 文件小，恢复快 | 可能丢失最后一次快照后的修改 |
| 适合备份 | 大内存时fork开销大 |

### 2.2 AOF（日志持久化）

**原理**：记录每个写操作到日志文件

**配置**：
```conf
# 开启AOF
appendonly yes

# 同步策略
appendfsync always    # 每次写入都同步（最安全）
appendfsync everysec  # 每秒同步（推荐）
appendfsync no        # 由操作系统决定
```

**AOF重写**：
```bash
BGREWRITEAOF  # 异步重写AOF文件
```

**AOF工作流程**：
```
1. 写命令写入AOF缓冲区
2. 根据策略刷入磁盘
3. 定期重写AOF文件（压缩）
```

**优缺点**：
| 优点 | 缺点 |
|------|------|
| 数据安全性高 | 文件体积大 |
| 可做增量备份 | 恢复速度慢 |

### 2.3 混合持久化

```conf
aof-use-rdb-preamble yes
```

**原理**：AOF文件头部为RDB格式，尾部为增量AOF日志

---

## 三、Redis集群架构

### 3.1 主从复制

```
        ┌──────────────┐
        │   Master     │
        │  (写操作)    │
        └──────┬───────┘
               │ 复制数据流
               ▼
        ┌──────────────┐
        │   Slave1     │
        │  (读操作)    │
        └──────┬───────┘
               │ 级联复制
               ▼
        ┌──────────────┐
        │   Slave2     │
        │  (读操作)    │
        └──────────────┘
```

**复制流程**：
```
1. Slave发送SYNC命令
2. Master执行BGSAVE生成RDB
3. Master发送RDB给Slave
4. Slave加载RDB
5. Master持续发送增量命令
```

**配置**：
```conf
# Slave配置
replicaof master_ip master_port
```

### 3.2 哨兵模式（Sentinel）

```
        ┌──────────────────────────────────┐
        │            Clients              │
        └──────────────────┬─────────────┘
                           │
        ┌──────────────────┼─────────────┐
        ▼                  ▼             ▼
┌─────────────┐    ┌─────────────┐  ┌─────────────┐
│  Sentinel1  │    │  Sentinel2  │  │  Sentinel3  │
│  (监控)     │    │  (监控)     │  │  (监控)     │
└──────┬──────┘    └──────┬──────┘  └──────┬──────┘
       │                  │                 │
       └────────┬────────┼────────┬────────┘
                │        │        │
                ▼        ▼        ▼
        ┌──────────┐ ┌──────────┐ ┌──────────┐
        │  Master  │ │  Slave1  │ │  Slave2  │
        │ (Active) │ │ (Standby)│ │ (Standby)│
        └──────────┘ └──────────┘ └──────────┘
```

**哨兵职责**：
1. **监控**：检查Master和Slave状态
2. **通知**：向客户端发送故障通知
3. **故障转移**：Master故障时选举新Master

**配置**（sentinel.conf）：
```conf
port 26379
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 30000
sentinel failover-timeout mymaster 180000
sentinel parallel-syncs mymaster 1
```

### 3.3 Cluster模式

```
          ┌───────────────────────────────────────┐
          │               Proxy                   │
          └──────────────────┬──────────────────┘
                             │
    ┌────────────────────────┼────────────────────────┐
    ▼                        ▼                        ▼
┌──────────┐          ┌──────────┐          ┌──────────┐
│  Node1   │          │  Node2   │          │  Node3   │
│ (Master) │          │ (Master) │          │ (Master) │
│ Slots 0-5│          │ Slots 6-10│         │ Slots 11-16│
└────┬─────┘          └────┬─────┘          └────┬─────┘
     │                     │                     │
     ▼                     ▼                     ▼
┌──────────┐          ┌──────────┐          ┌──────────┐
│ Slave1-1 │          │ Slave2-1 │          │ Slave3-1 │
│ (Replica)│          │ (Replica)│          │ (Replica)│
└──────────┘          └──────────┘          └──────────┘
```

**创建集群**：
```bash
redis-cli --cluster create \
    127.0.0.1:7000 127.0.0.1:7001 \
    127.0.0.1:7002 127.0.0.1:7003 \
    127.0.0.1:7004 127.0.0.1:7005 \
    --cluster-replicas 1
```

**槽位分配**：
- 共16384个槽位
- 每个Master负责一部分槽位
- 数据根据key的CRC16哈希分配到对应槽位

---

## 四、Redis常见问题分析与排查

### 问题1：缓存击穿

**现象**：热点key过期瞬间，大量请求直接访问数据库。

**排查步骤**：
```bash
# 查看key的过期时间
TTL hot_key

# 查看访问日志
redis-cli MONITOR | grep hot_key
```

**解决方案**：
1. **设置热点key永不过期**：
   ```bash
   SET hot_key value    # 不设置过期时间
   ```
2. **使用互斥锁**：
   ```go
   func GetHotKey(ctx context.Context, key string) (string, error) {
       val, err := client.Get(ctx, key).Result()
       if err == redis.Nil {
           // 获取锁
           lockKey := "lock:" + key
           lock, err := client.SetNX(ctx, lockKey, "1", 10*time.Second).Result()
           if !lock {
               // 等待锁释放
               time.Sleep(100 * time.Millisecond)
               return GetHotKey(ctx, key)
           }
           defer client.Del(ctx, lockKey)
           // 从DB加载
           val = loadFromDB(key)
           client.Set(ctx, key, val, 5*time.Minute)
       }
       return val, nil
   }
   ```
3. **提前预热缓存**

---

### 问题2：缓存雪崩

**现象**：大量缓存key同时过期，导致数据库压力激增。

**排查步骤**：
```bash
# 查看即将过期的key
redis-cli --scan --pattern "*" | xargs redis-cli TTL
```

**解决方案**：
1. **设置随机过期时间**：
   ```go
   expiration := 30*time.Minute + time.Duration(rand.Intn(300))*time.Second
   client.Set(ctx, key, value, expiration)
   ```
2. **使用多级缓存**：
   - 本地缓存（进程内）
   - Redis缓存
   - 数据库
3. **开启持久化**：防止Redis重启后缓存丢失

---

### 问题3：缓存穿透

**现象**：请求不存在的key，每次都访问数据库。

**排查步骤**：
```bash
# 查看不存在的key访问
redis-cli MONITOR | grep "GET.*non_exist"
```

**解决方案**：
1. **缓存空值**：
   ```go
   val, err := client.Get(ctx, key).Result()
   if err == redis.Nil {
       val = loadFromDB(key)
       if val == "" {
           // 缓存空值，设置较短过期时间
           client.Set(ctx, key, "", 60*time.Second)
       } else {
           client.Set(ctx, key, val, 5*time.Minute)
       }
   }
   ```
2. **使用布隆过滤器**：
   ```go
   // 初始化布隆过滤器
   filter := bloom.New(1000000, 5)
   
   // 添加存在的key
   filter.Add([]byte("existing_key"))
   
   // 查询前检查
   if !filter.Test([]byte(key)) {
       return nil, errors.New("key not exists")
   }
   ```
3. **参数校验**：在应用层过滤无效参数

---

### 问题4：内存溢出

**现象**：OOM killer杀死Redis进程，或Redis拒绝写入。

**排查步骤**：
```bash
# 查看内存使用
redis-cli INFO memory

# 查看大key
redis-cli --bigkeys

# 查看内存策略
redis-cli CONFIG GET maxmemory-policy
```

**解决方案**：
1. **设置最大内存限制**：
   ```conf
   maxmemory 4gb
   maxmemory-policy allkeys-lru
   ```
2. **定期清理无效数据**：
   ```bash
   redis-cli SCAN 0 MATCH "temp:*" | xargs redis-cli DEL
   ```
3. **分片存储**：将数据分散到多个Redis实例

---

### 问题5：主从复制延迟

**现象**：从库数据落后于主库。

**排查步骤**：
```bash
# 查看复制状态
redis-cli INFO replication

# 查看延迟
redis-cli -p 6380 INFO replication | grep lag
```

**解决方案**：
1. **优化网络**：使用高速网络，减少跨地域复制
2. **减少大事务**：将大事务拆分为小事务
3. **使用管道**：批量发送命令减少往返时间
4. **增加从库资源**：提升从库硬件配置

---

### 问题6：哨兵故障转移失败

**现象**：Master故障后，哨兵无法选举出新Master。

**排查步骤**：
```bash
# 查看哨兵日志
tail -f /var/log/redis/sentinel.log

# 查看哨兵状态
redis-cli -p 26379 SENTINEL master mymaster
```

**解决方案**：
1. **检查网络**：确保哨兵之间、哨兵与Redis节点之间网络通畅
2. **调整超时时间**：
   ```conf
   sentinel down-after-milliseconds mymaster 5000
   ```
3. **增加哨兵数量**：至少3个哨兵节点

---

### 问题7：集群槽位分配不均

**现象**：部分节点负载过高，其他节点空闲。

**排查步骤**：
```bash
# 查看槽位分布
redis-cli -p 7000 CLUSTER SLOTS

# 查看节点状态
redis-cli -p 7000 CLUSTER NODES
```

**解决方案**：
1. **重新分配槽位**：
   ```bash
   redis-cli --cluster reshard 127.0.0.1:7000
   ```
2. **均衡分配**：确保每个Master负责相近数量的槽位

---

### 问题8：Big Key问题

**现象**：单个key过大，导致内存分布不均、网络阻塞。

**排查步骤**：
```bash
# 查找大key
redis-cli --bigkeys

# 查看key大小
redis-cli MEMORY USAGE big_key
```

**解决方案**：
1. **拆分大key**：将单个大哈希拆分为多个小哈希
   ```go
   // 原设计：user:1 -> {name, age, email, address, ...}
   // 优化后：user:1:basic -> {name, age}
   //        user:1:contact -> {email, address}
   ```
2. **使用流式处理**：对于大列表，使用LRANGE分批获取

---

### 问题9：连接池耗尽

**现象**：应用无法获取Redis连接，报连接超时。

**排查步骤**：
```bash
# 查看客户端连接数
redis-cli INFO clients

# 查看连接池配置
redis-cli CONFIG GET maxclients
```

**解决方案**：
1. **增加最大连接数**：
   ```conf
   maxclients 10000
   ```
2. **优化应用连接池**：
   ```go
   pool := &redis.Pool{
       MaxIdle:     100,
       MaxActive:   1000,
       IdleTimeout: 30 * time.Second,
   }
   ```
3. **检查连接泄漏**：确保每次获取连接后都正确释放

---

### 问题10：AOF文件过大

**现象**：AOF文件持续增长，占用大量磁盘空间。

**排查步骤**：
```bash
# 查看AOF文件大小
ls -lh /var/lib/redis/*.aof

# 查看AOF重写状态
redis-cli INFO persistence
```

**解决方案**：
1. **定期重写AOF**：
   ```bash
   redis-cli BGREWRITEAOF
   ```
2. **调整重写阈值**：
   ```conf
   auto-aof-rewrite-percentage 100
   auto-aof-rewrite-min-size 64mb
   ```

---

## 五、Redis面试题

### 基础问题

**Q1：Redis支持哪些数据类型？**

**A1：**
- **String（字符串）**：最基础类型，支持字符串、数字、二进制数据
- **Hash（哈希）**：键值对集合，适合存储对象
- **List（列表）**：有序字符串列表，支持两端操作
- **Set（集合）**：无序唯一元素集合，支持交集、并集、差集
- **Sorted Set（有序集合）**：带分数的有序集合，适合排行榜

---

**Q2：Redis为什么这么快？**

**A2：**
1. **内存存储**：数据全部在内存中，读写速度极快
2. **单线程模型**：避免线程切换开销，无需锁竞争
3. **高效数据结构**：底层使用优化的数据结构（SDS、跳跃表等）
4. **IO多路复用**：使用epoll/kqueue实现高效网络IO
5. **避免磁盘IO**：大部分操作无需磁盘交互

---

**Q3：Redis的持久化方式有哪些？区别是什么？**

**A3：**

| 方式 | RDB | AOF |
|------|-----|-----|
| **原理** | 定期快照 | 记录写操作日志 |
| **数据安全性** | 较低（可能丢失数据） | 较高（可配置） |
| **文件大小** | 较小 | 较大 |
| **恢复速度** | 较快 | 较慢 |
| **性能影响** | 大内存时fork开销大 | 写操作有额外开销 |

---

**Q4：什么是Redis的事务？如何实现？**

**A4：**
Redis事务是一组命令的集合，保证：
- **原子性**：事务中的命令要么全部执行，要么全部不执行
- **隔离性**：事务执行期间不会被其他命令打断

```bash
MULTI      # 开始事务
SET key1 val1
SET key2 val2
EXEC       # 执行事务
DISCARD    # 取消事务
```

**注意**：Redis事务不支持回滚，执行过程中如果某条命令失败，后续命令仍会执行。

---

**Q5：Redis的主从复制原理是什么？**

**A5：**
1. **全量复制**：Slave首次连接Master时，Master生成RDB快照发送给Slave
2. **增量复制**：全量复制完成后，Master持续将写命令同步给Slave

**复制流程**：
```
1. Slave发送SYNC命令
2. Master执行BGSAVE生成RDB
3. Master缓存期间的写命令
4. Master发送RDB给Slave
5. Slave加载RDB
6. Master发送缓存的写命令
7. 持续增量同步
```

---

**Q6：什么是哨兵模式？作用是什么？**

**A6：**
哨兵是Redis的高可用解决方案，主要作用：
1. **监控**：定期检查Master和Slave的健康状态
2. **通知**：当节点故障时通知客户端
3. **故障转移**：Master故障时自动选举新Master
4. **配置更新**：更新Slave的复制配置

---

**Q7：Redis集群是如何工作的？**

**A7：**
Redis Cluster采用**槽位分片**方式：
- 共16384个槽位
- 每个Master节点负责一部分槽位
- key根据CRC16(key) % 16384分配到对应槽位
- 每个Master有多个Slave作为副本
- 支持自动故障转移

---

**Q8：什么是缓存穿透、击穿、雪崩？区别是什么？**

**A8：**

| 问题 | 描述 | 原因 |
|------|------|------|
| **缓存穿透** | 请求不存在的key，直接访问DB | 恶意攻击、无效请求 |
| **缓存击穿** | 热点key过期瞬间，大量请求访问DB | 热点key集中过期 |
| **缓存雪崩** | 大量key同时过期，DB压力激增 | 批量key设置相同过期时间 |

---

**Q9：Redis的内存淘汰策略有哪些？**

**A9：**
- **volatile-lru**：从设置了过期时间的key中淘汰最近最少使用的
- **allkeys-lru**：从所有key中淘汰最近最少使用的
- **volatile-ttl**：从设置了过期时间的key中淘汰即将过期的
- **volatile-random**：从设置了过期时间的key中随机淘汰
- **allkeys-random**：从所有key中随机淘汰
- **noeviction**：不淘汰，写入时返回错误

---

**Q10：Redis如何实现分布式锁？**

**A10：**
```bash
# 获取锁
SET lock:resource unique_value NX PX 30000

# 释放锁（Lua脚本保证原子性）
if redis.call("get", KEYS[1]) == ARGV[1] then
    return redis.call("del", KEYS[1])
else
    return 0
end
```

**关键点**：
- 使用SET NX保证互斥
- 设置过期时间避免死锁
- 使用唯一值防止误删其他客户端的锁

---

### 复杂场景问题

**Q11：如何设计高可用的Redis架构？**

**A11：**

**架构设计：**
```
        ┌──────────────────────────────────┐
        │            负载均衡层            │
        │      (Twemproxy/Cluster Client) │
        └───────────────┬────────────────┘
                        │
        ┌───────────────┼───────────────┐
        ▼               ▼               ▼
┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│  Sentinel1  │   │  Sentinel2  │   │  Sentinel3  │
└──────┬──────┘   └──────┬──────┘   └──────┬──────┘
       │                 │                 │
       └────────┬────────┼────────┬────────┘
                │        │        │
                ▼        ▼        ▼
        ┌──────────┐ ┌──────────┐ ┌──────────┐
        │  Master1 │ │  Master2 │ │  Master3 │
        │ (Active) │ │ (Active) │ │ (Active) │
        └────┬─────┘ └────┬─────┘ └────┬─────┘
             │            │            │
             ▼            ▼            ▼
        ┌──────────┐ ┌──────────┐ ┌──────────┐
        │  Slave1  │ │  Slave2  │ │  Slave3  │
        └──────────┘ └──────────┘ └──────────┘
```

**关键组件：**
1. **哨兵集群**：至少3个节点，实现自动故障转移
2. **主从复制**：每个Master配置多个Slave
3. **客户端路由**：支持自动发现和故障切换
4. **监控告警**：Prometheus + Grafana

**故障切换流程：**
1. 哨兵检测到Master故障
2. 哨兵集群选举Leader
3. Leader执行故障转移
4. 从Slave中选举新Master
5. 其他Slave指向新Master
6. 通知客户端

---

**Q12：如何解决高并发下的热点数据问题？**

**A12：**

**问题分析：**
- 热点数据导致：锁竞争严重、主从延迟增大、缓存击穿风险

**解决方案：**

1. **多级缓存架构**：
   ```go
   func GetHotData(ctx context.Context, key string) (string, error) {
       // 1. 本地缓存（进程内）
       if val := localCache.Get(key); val != nil {
           return val, nil
       }
       // 2. Redis缓存
       val, err := client.Get(ctx, key).Result()
       if err == nil {
           localCache.Set(key, val, 1*time.Minute)
           return val, nil
       }
       // 3. 数据库
       val = loadFromDB(key)
       client.Set(ctx, key, val, 5*time.Minute)
       localCache.Set(key, val, 1*time.Minute)
       return val, nil
   }
   ```

2. **数据分片**：
   ```
   # 将热点数据分散到多个key
   hot_data:0 ~ hot_data:99
   ```

3. **读写分离**：
   - 写入：Master
   - 读取：多个Slave轮询

4. **热点数据预热**：
   ```bash
   # 预热脚本
   redis-cli SET hot_key "$(cat hot_data.txt)"
   ```

---

**Q13：如何处理Redis与数据库的数据一致性问题？**

**A13：**

**场景分析：**
- 缓存与数据库数据不一致是常见问题
- 需要根据业务场景选择合适的策略

**策略一：先更新数据库，再删除缓存（推荐）**
```go
func UpdateData(ctx context.Context, key string, value string) error {
    // 1. 更新数据库
    err := db.Update(key, value)
    if err != nil {
        return err
    }
    // 2. 删除缓存（让下次读取重新加载）
    client.Del(ctx, key)
    return nil
}
```

**策略二：延迟双删（解决并发问题）**
```go
func UpdateData(ctx context.Context, key string, value string) error {
    // 1. 删除缓存
    client.Del(ctx, key)
    // 2. 更新数据库
    err := db.Update(key, value)
    if err != nil {
        return err
    }
    // 3. 延迟删除（处理并发写入）
    go func() {
        time.Sleep(500 * time.Millisecond)
        client.Del(ctx, key)
    }()
    return nil
}
```

**策略三：使用消息队列异步更新**
```
应用 → 更新DB → 发送消息 → 消费者删除缓存
```

---

**Q14：如何设计Redis的key过期策略？**

**A14：**

**过期策略分类：**

1. **定时过期**：设置精确过期时间
   ```bash
   SET session:user123 token EX 3600  # 1小时过期
   ```

2. **相对过期**：基于访问时间刷新过期
   ```go
   func GetSession(ctx context.Context, sessionID string) (string, error) {
       val, err := client.Get(ctx, sessionID).Result()
       if err == nil {
           // 刷新过期时间
           client.Expire(ctx, sessionID, 3600*time.Second)
       }
       return val, err
   }
   ```

3. **惰性删除**：访问时检查是否过期
   - Redis默认策略
   - 优点：节省CPU
   - 缺点：内存可能存在过期key

4. **定期删除**：后台定期扫描过期key
   ```conf
   # 每秒扫描10次，每次最多处理25个key
   hz 10
   maxmemory-samples 25
   ```

**最佳实践：**
- 热点数据：设置较长过期时间 + 惰性刷新
- 临时数据：设置较短过期时间
- 配置合理的内存淘汰策略

---

**Q15：如何实现Redis的分布式限流？**

**A15：**

**方案一：基于计数器的限流**
```go
func RateLimit(ctx context.Context, key string, limit int, duration time.Duration) bool {
    count, err := client.Incr(ctx, key).Result()
    if err != nil {
        return false
    }
    if count == 1 {
        client.Expire(ctx, key, duration)
    }
    return count <= int64(limit)
}
```

**方案二：基于滑动窗口的限流**
```go
func SlidingWindowLimit(ctx context.Context, key string, limit int, window time.Duration) bool {
    now := time.Now().Unix()
    windowStart := now - int64(window.Seconds())
    
    // 移除窗口外的记录
    client.ZRemRangeByScore(ctx, key, "-inf", strconv.Itoa(windowStart))
    
    // 获取当前窗口内的请求数
    count, _ := client.ZCard(ctx, key).Result()
    if count >= int64(limit) {
        return false
    }
    
    // 添加当前请求
    client.ZAdd(ctx, key, &redis.Z{
        Score:  float64(now),
        Member: strconv.Itoa(rand.Int()),
    })
    
    // 设置过期时间
    client.Expire(ctx, key, window)
    return true
}
```

**方案三：使用令牌桶算法**
```go
func TokenBucketLimit(ctx context.Context, key string, capacity int, rate int) bool {
    // 尝试获取令牌
    tokens, _ := client.Get(ctx, key).Int()
    if tokens <= 0 {
        // 补充令牌
        tokens = capacity
        client.Set(ctx, key, tokens, 1*time.Second)
    }
    
    if tokens > 0 {
        client.Decr(ctx, key)
        return true
    }
    return false
}
```

---

**Q16：如何处理Redis的大数据量存储？**

**A16：**

**问题分析：**
- 单节点内存有限
- 数据量过大导致：内存不足、查询变慢、备份困难

**解决方案：**

1. **数据分片**：
   ```
   # 按用户ID分片
   user:0001 ~ user:0099 → Redis节点1
   user:0100 ~ user:0199 → Redis节点2
   ...
   ```

2. **冷热数据分离**：
   ```go
   func GetData(ctx context.Context, key string) (string, error) {
       // 先查热数据（内存）
       val, err := hotClient.Get(ctx, key).Result()
       if err == nil {
           return val, nil
       }
       // 再查冷数据（磁盘）
       val, err = coldClient.Get(ctx, key).Result()
       if err == nil {
           // 提升为热数据
           hotClient.Set(ctx, key, val, 5*time.Minute)
       }
       return val, err
   }
   ```

3. **使用Redis Cluster**：
   - 自动分片
   - 水平扩展
   - 自动故障转移

4. **数据压缩**：
   - 对大文本数据进行压缩存储
   - 使用MessagePack等高效序列化格式

---

**Q17：如何实现Redis的消息队列？**

**A17：**

**方案一：使用List实现简单队列**
```bash
# 生产者
LPUSH queue:tasks "task1"
LPUSH queue:tasks "task2"

# 消费者
BRPOP queue:tasks 0  # 阻塞等待
```

**方案二：使用Pub/Sub实现发布订阅**
```bash
# 订阅者
SUBSCRIBE channel:news

# 发布者
PUBLISH channel:news "Hello World"
```

**方案三：使用Stream实现消息队列**
```bash
# 添加消息
XADD stream:orders * user_id 1 product "iPhone"

# 消费消息（创建消费者组）
XGROUP CREATE stream:orders group1 0
XREADGROUP GROUP group1 consumer1 COUNT 10 STREAMS stream:orders >

# 确认消息
XACK stream:orders group1 message_id
```

**方案对比：**

| 方案 | 特点 | 适用场景 |
|------|------|----------|
| List | 简单、阻塞读取 | 任务队列 |
| Pub/Sub | 广播模式、无持久化 | 实时通知 |
| Stream | 持久化、消费者组、ACK | 可靠消息队列 |

---

**Q18：如何进行Redis的性能优化？**

**A18：**

**优化方向：**

1. **命令优化**：
   ```bash
   # 不好：多次往返
   GET key1
   GET key2
   GET key3
   
   # 好：批量操作
   MGET key1 key2 key3
   ```

2. **管道优化**：
   ```go
   pipe := client.Pipeline()
   pipe.Set(ctx, "key1", "val1", 0)
   pipe.Set(ctx, "key2", "val2", 0)
   pipe.Get(ctx, "key1")
   _, err := pipe.Exec(ctx)
   ```

3. **连接池优化**：
   ```go
   pool := &redis.Pool{
       MaxIdle:     100,
       MaxActive:   1000,
       IdleTimeout: 30 * time.Second,
       Dial: func() (redis.Conn, error) {
           return redis.Dial("tcp", "localhost:6379")
       },
   }
   ```

4. **数据结构优化**：
   - 使用Hash代替多个String
   - 使用Set代替List进行去重
   - 使用Sorted Set实现排行榜

5. **配置优化**：
   ```conf
   # 关闭保护模式
   protected-mode no
   
   # 禁用THP（透明大页）
   vm.overcommit_memory = 1
   
   # 设置最大内存
   maxmemory 8gb
   maxmemory-policy allkeys-lru
   ```

---

**Q19：如何进行Redis的监控和告警？**

**A19：**

**监控指标：**

| 指标 | 说明 | 告警阈值 |
|------|------|----------|
| used_memory | 已使用内存 | > 80% maxmemory |
| connected_clients | 客户端连接数 | > maxclients * 80% |
| used_cpu_sys | 系统CPU使用率 | > 80% |
| replication_offset | 复制偏移量 | > 100000 |
| keyspace_hits/misses | 缓存命中率 | < 90% |

**监控方案：**

1. **使用INFO命令**：
   ```bash
   redis-cli INFO memory
   redis-cli INFO clients
   redis-cli INFO replication
   ```

2. **使用Prometheus + Grafana**：
   ```yaml
   # redis_exporter配置
   scrape_configs:
     - job_name: 'redis'
       static_configs:
         - targets: ['localhost:6379']
   ```

3. **自定义监控脚本**：
   ```bash
   # 检查内存使用率
   used=$(redis-cli INFO memory | grep used_memory_percent | cut -d':' -f2)
   if [ $(echo "$used > 80" | bc) -eq 1 ]; then
       echo "Memory usage exceeded 80%" | mail -s "Redis Alert" admin@example.com
   fi
   ```

---

**Q20：如何进行Redis的备份和恢复？**

**A20：**

**备份策略：**

1. **RDB备份**：
   ```bash
   # 手动触发备份
   redis-cli BGSAVE
   
   # 备份文件位置
   cp /var/lib/redis/dump.rdb /backup/redis_$(date +%Y%m%d).rdb
   ```

2. **AOF备份**：
   ```bash
   # 备份AOF文件
   cp /var/lib/redis/appendonly.aof /backup/aof_$(date +%Y%m%d).aof
   ```

3. **定期备份**：
   ```bash
   # crontab配置（每天凌晨2点备份）
   0 2 * * * redis-cli BGSAVE && cp /var/lib/redis/dump.rdb /backup/redis_$(date +\%Y\%m\%d).rdb
   ```

**恢复流程：**

```
1. 停止Redis服务
2. 备份现有数据目录
3. 将备份文件复制到数据目录
4. 启动Redis服务
5. 验证数据完整性
```

**注意事项：**
- 备份时使用BGSAVE避免阻塞
- 定期测试恢复流程
- 异地备份防止单点故障

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

# MySQL详解

## MySQL是什么？
&emsp;&emsp;MySQL是一个开源的关系型数据库管理系统（RDBMS），使用SQL（结构化查询语言）进行数据管理。它是Web开发中最常用的数据库之一，由Oracle公司维护。

## MySQL的特点

| 特性 | 说明 |
|------|------|
| **开源** | 免费使用，社区活跃 |
| **跨平台** | 支持多种操作系统 |
| **高性能** | 优化的存储引擎（InnoDB） |
| **高可用** | 支持主从复制、集群 |
| **事务支持** | ACID事务 |
| **存储引擎** | 支持多种存储引擎 |

---

## 一、InnoDB引擎深度解析

### 1.1 InnoDB架构

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        Buffer Pool                                      │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │
│  │  数据页缓存  │  │  索引页缓存  │  │   自适应哈希 │  │   插入缓冲  │    │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘    │
└───────────────────────────┬─────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                        InnoDB Core                                     │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │   Buffer Pool Manager    Log Buffer    Lock Manager            │    │
│  │   Change Buffer          Purge Thread  Master Thread          │    │
│  └─────────────────────────────────────────────────────────────────┘    │
└───────────────────────────┬─────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                        Disk Storage                                    │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │
│  │   .ibd文件  │  │ ib_logfile* │  │ ibdata1     │  │ undo日志    │    │
│  │  (表空间)  │  │ (重做日志)  │  │ (系统表空间)│  │  (回滚日志)  │    │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘    │
└─────────────────────────────────────────────────────────────────────────┘
```

### 1.2 InnoDB核心组件

#### 1.2.1 Buffer Pool（缓冲池）

Buffer Pool是InnoDB最重要的内存结构，用于缓存表数据和索引数据：

```sql
-- 查看Buffer Pool配置
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';
SHOW VARIABLES LIKE 'innodb_buffer_pool_instances';

-- 查看Buffer Pool状态
SHOW ENGINE INNODB STATUS\G
```

**Buffer Pool优化建议：**
- 设置为物理内存的50%-70%
- 多实例配置（innodb_buffer_pool_instances）
- 使用O_DIRECT避免double buffering

#### 1.2.2 重做日志（Redo Log）

Redo Log确保事务的持久性，采用WAL（Write-Ahead Logging）策略：

```sql
-- 查看Redo Log配置
SHOW VARIABLES LIKE 'innodb_log_file_size';
SHOW VARIABLES LIKE 'innodb_log_buffer_size';
SHOW VARIABLES LIKE 'innodb_flush_log_at_trx_commit';
```

**innodb_flush_log_at_trx_commit取值：**
| 值 | 含义 | 安全性 | 性能 |
|----|------|--------|------|
| 0 | 每秒刷盘 | 低 | 高 |
| 1 | 每次事务刷盘 | 高 | 低 |
| 2 | 每次事务写入OS缓存，每秒刷盘 | 中 | 中 |

#### 1.2.3 回滚日志（Undo Log）

Undo Log用于事务回滚和MVCC（多版本并发控制）：

```sql
-- 查看Undo Log配置
SHOW VARIABLES LIKE 'innodb_undo_tablespaces';
SHOW VARIABLES LIKE 'innodb_undo_log_truncate';
```

### 1.3 InnoDB事务实现

#### 1.3.1 ACID特性

| 特性 | 说明 | InnoDB实现 |
|------|------|-----------|
| **Atomicity** | 原子性 | Undo Log |
| **Consistency** | 一致性 | 约束检查、触发器 |
| **Isolation** | 隔离性 | MVCC、锁 |
| **Durability** | 持久性 | Redo Log |

#### 1.3.2 事务隔离级别

```sql
-- 查看当前隔离级别
SELECT @@tx_isolation;

-- 设置隔离级别
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

| 隔离级别 | 脏读 | 不可重复读 | 幻读 |
|----------|------|------------|------|
| READ UNCOMMITTED | 可能 | 可能 | 可能 |
| READ COMMITTED | 不可能 | 可能 | 可能 |
| REPEATABLE READ | 不可能 | 不可能 | 不可能（InnoDB） |
| SERIALIZABLE | 不可能 | 不可能 | 不可能 |

### 1.4 InnoDB锁机制

#### 1.4.1 锁类型

| 锁类型 | 说明 | 场景 |
|--------|------|------|
| **行锁** | 锁定单行记录 | UPDATE、DELETE、SELECT...FOR UPDATE |
| **表锁** | 锁定整个表 | ALTER TABLE、LOCK TABLES |
| **意向锁** | 表明事务意图 | 行锁前自动加 |
| **间隙锁** | 锁定索引间隙 | 防止幻读 |

#### 1.4.2 死锁处理

```sql
-- 查看死锁日志
SHOW ENGINE INNODB STATUS\G

-- 开启死锁检测
SET GLOBAL innodb_deadlock_detect = ON;

-- 设置锁等待超时
SET GLOBAL innodb_lock_wait_timeout = 50;
```

---

## 二、MySQL集群架构

### 2.1 主从复制架构

```
        ┌──────────────┐
        │   Master     │
        │  (写操作)    │
        └──────┬───────┘
               │ binlog
               ▼
        ┌──────────────┐
        │   Relay Log  │
        └──────┬───────┘
               │
     ┌─────────┼─────────┐
     ▼         ▼         ▼
┌──────────┐┌──────────┐┌──────────┐
│  Slave1  ││  Slave2  ││  Slave3  │
│ (读操作) ││ (读操作) ││ (读操作) │
└──────────┘└──────────┘└──────────┘
```

### 2.2 主主复制架构

```
    ┌─────────────────────────────┐
    │          VIP               │
    └────────────┬──────────────┘
                 │
     ┌───────────┴───────────┐
     ▼                       ▼
┌──────────┐           ┌──────────┐
│ Master1  │◄────────►│ Master2  │
│ (可读写) │ 互相同步 │ (可读写) │
└──────────┘           └──────────┘
```

### 2.3 读写分离架构

```
        ┌──────────────┐
        │   Proxy      │
        │ (SQL解析)    │
        └──────┬───────┘
               │
     ┌─────────┴─────────┐
     ▼                   ▼
┌──────────┐       ┌──────────┐
│  Master  │       │  Slaves  │
│ (写)     │       │ (读集群) │
└──────────┘       └──────────┘
```

### 2.4 MGR（MySQL Group Replication）

```
        ┌───────────────────────────────────────┐
        │              Group Coordinator        │
        └───────────────┬───────────────────────┘
                        │
     ┌──────────────────┼──────────────────┐
     ▼                  ▼                  ▼
┌──────────┐       ┌──────────┐       ┌──────────┐
│  Node1   │       │  Node2   │       │  Node3   │
│ (Primary)│       │ (Secondary)│     │ (Secondary)│
└──────────┘       └──────────┘       └──────────┘
```

---

## 三、MySQL主从复制原理

### 3.1 复制流程

```
Master                    Slave
  │                          │
  │ 1. 执行SQL               │
  │      │                   │
  │      ▼                   │
  │ 2. 写入Binlog           │
  │      │                   │
  │      ▼                   │
  │ 3. Dump Thread发送      │
  └─────────►│               │
             │ 4. IO Thread接收
             ▼               │
        Relay Log           │
             │               │
             ▼               │
        5. SQL Thread执行    │
             │               │
             ▼               │
        6. 更新数据          │
```

### 3.2 Binlog详解

#### 3.2.1 Binlog格式

```sql
-- 查看Binlog格式
SHOW VARIABLES LIKE 'binlog_format';

-- 设置Binlog格式
SET GLOBAL binlog_format = 'ROW';
```

| 格式 | 说明 | 优点 | 缺点 |
|------|------|------|------|
| **STATEMENT** | 记录SQL语句 | 体积小 | 可能不一致 |
| **ROW** | 记录行变更 | 数据一致 | 体积大 |
| **MIXED** | 混合模式 | 平衡优缺点 | 复杂 |

#### 3.2.2 Binlog操作

```sql
-- 查看Binlog列表
SHOW BINARY LOGS;

-- 查看Binlog内容
SHOW BINLOG EVENTS IN 'mysql-bin.000001';

-- 使用mysqlbinlog工具
mysqlbinlog /var/log/mysql/mysql-bin.000001
```

### 3.3 复制类型

#### 3.3.1 异步复制（默认）

```sql
-- 异步复制配置（默认）
sync_binlog = 0
```

#### 3.3.2 半同步复制

```sql
-- 启用半同步复制
INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so';
INSTALL PLUGIN rpl_semi_sync_slave SONAME 'semisync_slave.so';

SET GLOBAL rpl_semi_sync_master_enabled = 1;
SET GLOBAL rpl_semi_sync_slave_enabled = 1;
```

#### 3.3.3 GTID复制

```sql
-- 启用GTID
gtid_mode = ON
enforce_gtid_consistency = ON
```

### 3.4 复制延迟问题

```sql
-- 查看复制延迟
SHOW SLAVE STATUS\G

-- 延迟指标
Seconds_Behind_Master: 0
```

**延迟原因分析：**
1. 网络延迟
2. 大事务
3. 索引缺失
4. 从库硬件不足

---

## 四、MySQL性能优化

### 4.1 索引优化

#### 4.1.1 索引类型

| 索引类型 | 说明 | 适用场景 |
|----------|------|----------|
| **B+Tree** | 默认索引类型 | 范围查询、排序 |
| **Hash** | 哈希索引 | 等值查询 |
| **Full-text** | 全文索引 | 文本搜索 |
| **Spatial** | 空间索引 | 地理数据 |

#### 4.1.2 索引失效场景

```sql
-- 1. 函数操作
SELECT * FROM users WHERE YEAR(created_at) = 2024;

-- 2. 类型转换
SELECT * FROM users WHERE id = '123';

-- 3. 模糊查询前缀
SELECT * FROM users WHERE name LIKE '%John';

-- 4. OR条件不全有索引
SELECT * FROM users WHERE name = 'John' OR age = 30;

-- 5. 复合索引不满足最左前缀
SELECT * FROM users WHERE age = 30; -- idx_name_age
```

#### 4.1.3 执行计划分析

```sql
EXPLAIN SELECT * FROM users WHERE name = 'John';
```

**EXPLAIN输出字段：**
| 字段 | 说明 |
|------|------|
| **id** | 查询序列号 |
| **select_type** | 查询类型 |
| **table** | 表名 |
| **type** | 访问类型（ALL、index、range、ref、eq_ref、const） |
| **key** | 使用的索引 |
| **rows** | 预估行数 |
| **Extra** | 额外信息（Using index、Using where、Using filesort） |

### 4.2 查询优化

#### 4.2.1 慢查询日志

```ini
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 2
log_queries_not_using_indexes = 1
```

#### 4.2.2 查询优化策略

```sql
-- 优化前：SELECT *
SELECT * FROM users WHERE status = 1;

-- 优化后：只查询需要的字段
SELECT id, name FROM users WHERE status = 1;

-- 优化前：子查询
SELECT * FROM orders WHERE user_id IN (SELECT id FROM users WHERE status = 1);

-- 优化后：JOIN
SELECT o.* FROM orders o JOIN users u ON o.user_id = u.id WHERE u.status = 1;
```

### 4.3 配置优化

```ini
[mysqld]
# 连接配置
max_connections = 2000
wait_timeout = 60
interactive_timeout = 60

# InnoDB配置
innodb_buffer_pool_size = 8G
innodb_buffer_pool_instances = 8
innodb_log_file_size = 2G
innodb_log_buffer_size = 64M
innodb_flush_log_at_trx_commit = 1
innodb_flush_method = O_DIRECT

# 查询优化
query_cache_type = 0
query_cache_size = 0
sort_buffer_size = 2M
join_buffer_size = 2M

# 日志配置
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 2
```

---

## 五、MySQL常见问题分析与排查

### 问题1：数据库连接数耗尽

**现象：**
```
ERROR 1040 (08004): Too many connections
```

**排查步骤：**
```sql
-- 1. 查看当前连接数
SHOW GLOBAL STATUS LIKE 'Threads_connected';

-- 2. 查看最大连接数
SHOW VARIABLES LIKE 'max_connections';

-- 3. 查看连接详情
SHOW PROCESSLIST;
```

**解决方案：**
1. 增加`max_connections`
2. 检查应用是否正确释放连接
3. 配置连接池参数

---

### 问题2：慢查询导致性能问题

**现象：**
- CPU使用率高
- 查询响应慢
- 锁等待时间长

**排查步骤：**
```sql
-- 1. 查看慢查询日志
mysqldumpslow /var/log/mysql/slow.log

-- 2. 分析执行计划
EXPLAIN SELECT * FROM large_table WHERE ...;

-- 3. 查看当前运行的慢查询
SHOW FULL PROCESSLIST;
```

**解决方案：**
1. 添加合适的索引
2. 优化SQL语句
3. 增加硬件资源

---

### 问题3：主从复制延迟

**现象：**
```sql
SHOW SLAVE STATUS\G
-- Seconds_Behind_Master: 120
```

**排查步骤：**
```sql
-- 1. 查看从库状态
SHOW SLAVE STATUS\G

-- 2. 查看主库Binlog状态
SHOW MASTER STATUS;

-- 3. 检查是否有大事务
SELECT * FROM information_schema.innodb_trx;
```

**解决方案：**
1. 优化大事务
2. 增加从库硬件配置
3. 使用并行复制

---

### 问题4：死锁问题

**现象：**
```
ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction
```

**排查步骤：**
```sql
-- 1. 查看死锁日志
SHOW ENGINE INNODB STATUS\G

-- 2. 查看当前事务
SELECT * FROM information_schema.innodb_trx;

-- 3. 查看锁等待
SELECT * FROM information_schema.innodb_lock_waits;
```

**解决方案：**
1. 调整事务顺序
2. 缩短事务时间
3. 使用较低隔离级别

---

### 问题5：磁盘空间不足

**现象：**
```
ERROR 1030 (HY000): Got error 28 from storage engine
```

**排查步骤：**
```bash
# 1. 查看磁盘空间
df -h

# 2. 查看MySQL数据目录大小
du -sh /var/lib/mysql

# 3. 查看Binlog占用
ls -lh /var/log/mysql/*.00000*
```

**解决方案：**
1. 清理过期Binlog
2. 清理无效数据
3. 扩展磁盘空间

---

### 问题6：索引失效

**现象：**
- 查询突然变慢
- 执行计划显示全表扫描

**排查步骤：**
```sql
-- 1. 查看执行计划
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';

-- 2. 查看索引状态
SHOW INDEX FROM users;

-- 3. 检查索引碎片
ANALYZE TABLE users;
```

**解决方案：**
1. 修复无效索引
2. 删除并重建索引
3. 分析表统计信息

---

### 问题7：锁等待超时

**现象：**
```
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction
```

**排查步骤：**
```sql
-- 1. 查看锁等待超时时间
SHOW VARIABLES LIKE 'innodb_lock_wait_timeout';

-- 2. 查看当前锁等待
SELECT * FROM information_schema.innodb_lock_waits;
```

**解决方案：**
1. 优化事务逻辑
2. 增加锁等待超时时间
3. 减少事务持有锁的时间

---

### 问题8：Binlog损坏

**现象：**
```
ERROR 1236 (HY000): Slave can't handle replication event from master
```

**排查步骤：**
```bash
# 检查Binlog完整性
mysqlbinlog --verify /var/log/mysql/mysql-bin.000001
```

**解决方案：**
1. 使用备份恢复
2. 使用`mysqlbinlog`修复
3. 重新搭建从库

---

### 问题9：内存不足导致OOM

**现象：**
- MySQL进程被系统kill
- 系统日志显示OOM killer

**排查步骤：**
```bash
# 查看内存使用
free -h

# 查看MySQL内存配置
mysql -e "SHOW VARIABLES LIKE '%buffer%'"
```

**解决方案：**
1. 调整内存配置参数
2. 增加系统内存
3. 使用内存管理工具

---

### 问题10：表损坏

**现象：**
```
ERROR 1194 (HY000): Table 'users' is marked as crashed and should be repaired
```

**排查步骤：**
```sql
-- 检查表状态
CHECK TABLE users;

-- 修复表
REPAIR TABLE users;
```

**解决方案：**
1. 使用REPAIR TABLE修复
2. 从备份恢复
3. 检查磁盘健康

---

## 六、MySQL面试题

### 基础问题

**Q1：MySQL支持哪些存储引擎？区别是什么？**

**A1：**
- **InnoDB**：支持事务、行锁、外键、MVCC，适合OLTP场景
- **MyISAM**：不支持事务、表锁、全文索引，适合OLAP场景
- **Memory**：内存存储，速度快但数据不持久
- **Archive**：压缩存储，适合归档场景

**核心区别：**
| 特性 | InnoDB | MyISAM |
|------|--------|--------|
| 事务支持 | 支持 | 不支持 |
| 锁粒度 | 行锁 | 表锁 |
| 外键 | 支持 | 不支持 |
| 崩溃恢复 | 支持 | 不支持 |

---

**Q2：什么是ACID？MySQL如何保证ACID？**

**A2：**
- **Atomicity（原子性）**：通过Undo Log实现回滚
- **Consistency（一致性）**：通过约束检查、触发器保证
- **Isolation（隔离性）**：通过MVCC和锁机制实现
- **Durability（持久性）**：通过Redo Log和WAL策略保证

---

**Q3：MySQL有哪些隔离级别？默认是什么？**

**A3：**
- **READ UNCOMMITTED**：可读取未提交数据
- **READ COMMITTED**：只能读取已提交数据
- **REPEATABLE READ**：同一事务内读取结果一致（默认）
- **SERIALIZABLE**：串行化执行

MySQL默认隔离级别是**REPEATABLE READ**，InnoDB通过Next-Key Locking防止幻读。

---

**Q4：什么是MVCC？如何实现的？**

**A4：**
MVCC（Multi-Version Concurrency Control）是一种并发控制技术，通过以下方式实现：

1. **隐藏字段**：每行记录包含`DB_TRX_ID`（创建事务ID）和`DB_ROLL_PTR`（回滚指针）
2. **Undo Log**：保存旧版本数据
3. **Read View**：事务启动时创建的快照

**优势：** 读不阻塞写，写不阻塞读

---

**Q5：索引的作用是什么？有哪些类型？**

**A5：**
**作用：** 加速查询、约束数据唯一性

**类型：**
- **B+Tree索引**：默认，适合范围查询
- **Hash索引**：适合等值查询
- **全文索引**：适合文本搜索
- **空间索引**：适合地理数据

---

**Q6：什么是索引覆盖？**

**A6：**
索引覆盖（Covering Index）指查询所需的所有列都包含在索引中，不需要回表查询。

```sql
-- 非覆盖索引：需要回表
SELECT * FROM users WHERE name = 'John';

-- 覆盖索引：不需要回表
SELECT id, name FROM users WHERE name = 'John'; -- idx_name
```

---

**Q7：什么是最左前缀原则？**

**A7：**
复合索引`idx(a, b, c)`只会在以下情况使用：
- `WHERE a = ?`
- `WHERE a = ? AND b = ?`
- `WHERE a = ? AND b = ? AND c = ?`

不满足最左前缀的查询不会使用索引：
- `WHERE b = ?`
- `WHERE b = ? AND c = ?`

---

**Q8：什么是死锁？如何避免？**

**A8：**
死锁是两个或多个事务互相等待对方释放锁的状态。

**避免策略：**
1. 按固定顺序访问表
2. 缩短事务时间
3. 使用较低隔离级别
4. 设置合理的锁等待超时

---

**Q9：Binlog有哪些格式？区别是什么？**

**A9：**
- **STATEMENT**：记录SQL语句，体积小但可能不一致
- **ROW**：记录行变更，数据一致但体积大
- **MIXED**：混合模式，自动选择合适的格式

---

**Q10：主从复制的原理是什么？**

**A10：**
1. Master执行SQL后写入Binlog
2. Slave的IO Thread读取Binlog写入Relay Log
3. Slave的SQL Thread执行Relay Log中的SQL

---

### 复杂场景问题

**Q11：如何设计高可用的MySQL架构？**

**A11：**

**方案设计：**

```
           ┌──────────────────────────────────┐
           │          负载均衡层              │
           │      (ProxySQL/MaxScale)        │
           └───────────────┬────────────────┘
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
    ┌──────────┐      ┌──────────┐      ┌──────────┐
    │ Master1  │      │ Master2  │      │   MGR    │
    │ (Active) │      │ (Standby)│      │ Cluster  │
    └──────────┘      └──────────┘      └──────────┘
         │                 │                 │
         ▼                 ▼                 ▼
    ┌──────────┐      ┌──────────┐      ┌──────────┐
    │ Slaves   │      │ Slaves   │      │  Nodes   │
    │ (读集群) │      │ (读集群) │      │ (多节点) │
    └──────────┘      └──────────┘      └──────────┘
```

**关键组件：**
1. **主主复制**：双主互备
2. **MGR**：组复制，自动故障转移
3. **ProxySQL**：读写分离、故障检测
4. **监控告警**：Prometheus + Grafana

**故障切换流程：**
1. 监控检测Master故障
2. ProxySQL自动切换到备用Master
3. MGR自动选举新Primary
4. 通知运维人员处理

---

**Q12：如何解决高并发下的热点数据问题？**

**A12：**

**问题分析：**
热点数据会导致：
- 锁竞争严重
- 主从延迟增大
- 缓存击穿风险

**解决方案：**

1. **缓存层优化**：
```go
// 使用多级缓存
func GetUser(id int) *User {
    // 先查本地缓存
    if user := localCache.Get(id); user != nil {
        return user
    }
    // 再查Redis
    if user := redisCache.Get(id); user != nil {
        localCache.Set(id, user)
        return user
    }
    // 最后查DB
    user := db.GetUser(id)
    redisCache.Set(id, user, 5*time.Minute)
    localCache.Set(id, user, 1*time.Minute)
    return user
}
```

2. **读写分离**：
```
写入 → Master
读取 → Slave（轮询）
```

3. **数据分片**：
```
user_00: id % 100 = 00
user_01: id % 100 = 01
...
user_99: id % 100 = 99
```

4. **热点数据预热**：
```sql
-- 预热缓存
SELECT * FROM hot_items WHERE is_hot = 1;
```

---

**Q13：如何处理大表的查询性能问题？**

**A13：**

**问题分析：**
- 表数据量过大（亿级）
- 查询响应慢
- 索引失效

**解决方案：**

1. **分区表**：
```sql
-- 按时间分区
CREATE TABLE logs (
    id INT,
    log_time DATETIME,
    message TEXT
) PARTITION BY RANGE (TO_DAYS(log_time)) (
    PARTITION p202401 VALUES LESS THAN (TO_DAYS('2024-02-01')),
    PARTITION p202402 VALUES LESS THAN (TO_DAYS('2024-03-01')),
    PARTITION p202403 VALUES LESS THAN (TO_DAYS('2024-04-01'))
);
```

2. **分库分表**：
```
db0: user_00 ~ user_24
db1: user_25 ~ user_49
db2: user_50 ~ user_74
db3: user_75 ~ user_99
```

3. **延迟删除**：
```sql
-- 标记删除
UPDATE users SET deleted = 1 WHERE id = ?;

-- 异步清理
DELETE FROM users WHERE deleted = 1 AND delete_time < DATE_SUB(NOW(), INTERVAL 7 DAY);
```

4. **只读副本**：
```
Master: 处理写操作
Slave1: 处理分析查询
Slave2: 处理报表查询
```

---

**Q14：如何实现MySQL的数据备份与恢复？**

**A14：**

**备份策略：**

1. **全量备份（mysqldump）**：
```bash
mysqldump -u root -p --all-databases --single-transaction --master-data=2 > backup.sql
```

2. **增量备份（Binlog）**：
```bash
mysqlbinlog mysql-bin.000001 mysql-bin.000002 > incremental.sql
```

3. **物理备份（xtrabackup）**：
```bash
innobackupex --user=root --password=xxx /backup/
```

**恢复流程：**

```
1. 停止MySQL服务
2. 恢复全量备份
3. 应用增量Binlog
4. 启动MySQL服务
5. 验证数据一致性
```

**恢复命令：**
```bash
mysql -u root -p < backup.sql
mysql -u root -p < incremental.sql
```

---

**Q15：如何诊断和优化SQL性能问题？**

**A15：**

**诊断流程：**

```
1. 开启慢查询日志
   → slow_query_log = 1
   → long_query_time = 2

2. 分析慢查询日志
   → mysqldumpslow slow.log

3. 查看执行计划
   → EXPLAIN SELECT ...

4. 定位问题
   → 全表扫描？
   → 索引失效？
   → 大表关联？

5. 优化执行
```

**优化策略：**

```sql
-- 优化前
SELECT * FROM orders 
WHERE YEAR(create_time) = 2024 
  AND status = 1;

-- 优化后：避免函数操作索引
SELECT * FROM orders 
WHERE create_time BETWEEN '2024-01-01' AND '2025-01-01'
  AND status = 1;

-- 创建复合索引
CREATE INDEX idx_status_create_time ON orders(status, create_time);
```

**优化指标：**
- QPS（每秒查询次数）
- TPS（每秒事务次数）
- 响应时间
- 锁等待时间

---

**Q16：如何设计分布式事务？**

**A16：**

**方案一：XA事务（两阶段提交）**

```
协调者           参与者1           参与者2
   │                │                │
   │──Prepare───────►│                │
   │                │──Prepare───────►│
   │                │                │
   │◄─Prepared──────│                │
   │                │◄─Prepared──────│
   │                │                │
   │──Commit────────►│                │
   │                │──Commit────────►│
   │                │                │
   │◄─Committed─────│                │
   │                │◄─Committed─────│
```

**方案二：本地消息表**

```
应用A           消息表          应用B
   │                │               │
   │──保存业务数据──►│               │
   │──保存消息──────►│               │
   │                │──定时扫描─────►│
   │                │               │──执行业务
   │                │◄──确认成功─────│
   │                │──删除消息──────│
```

**方案三：Saga模式**

```
T1 → T2 → T3 → T4
  ←  ←  ←  ←
C1  C2  C3  C4

T: 事务
C: 补偿操作
```

---

**Q17：如何处理MySQL的并发写入冲突？**

**A17：**

**问题场景：**
- 库存扣减
- 余额更新
- 计数器递增

**解决方案：**

1. **乐观锁**：
```sql
UPDATE inventory 
SET stock = stock - 1 
WHERE id = 1 AND stock > 0;
```

2. **悲观锁**：
```sql
SELECT * FROM inventory WHERE id = 1 FOR UPDATE;
UPDATE inventory SET stock = stock - 1 WHERE id = 1;
```

3. **分布式锁**：
```go
func DeductStock(id int) error {
    lock := redis.NewLock("stock:" + id)
    if err := lock.Lock(); err != nil {
        return err
    }
    defer lock.Unlock()
    
    // 执行业务逻辑
    return db.DeductStock(id)
}
```

4. **队列串行化**：
```
请求 → RabbitMQ → 单消费者处理
```

---

**Q18：如何实现MySQL的读写分离？**

**A18：**

**架构设计：**

```
          ┌──────────────────────────────────────────┐
          │               应用层                      │
          │              (连接池)                     │
          └──────────────────┬───────────────────────┘
                             │
          ┌──────────────────┴───────────────────────┐
          ▼                                          ▼
    ┌──────────┐                              ┌──────────┐
    │  Proxy   │                              │  Proxy   │
    │ (写路由) │                              │ (读路由) │
    └────┬─────┘                              └────┬─────┘
         │                                        │
         ▼                                        ▼
    ┌──────────┐                        ┌─────────────────┐
    │  Master  │                        │     Slaves      │
    │ (写操作) │                        │ (读操作集群)    │
    └──────────┘                        └─────────────────┘
```

**实现方式：**

1. **ProxySQL**：
```sql
-- 配置读写分离
INSERT INTO mysql_servers (hostgroup_id, hostname, port) VALUES (1, 'master', 3306);
INSERT INTO mysql_servers (hostgroup_id, hostname, port) VALUES (2, 'slave1', 3306);
INSERT INTO mysql_servers (hostgroup_id, hostname, port) VALUES (2, 'slave2', 3306);

INSERT INTO query_rules (rule_id, match_pattern, destination_hostgroup) 
VALUES (1, '^SELECT.*', 2);
```

2. **应用层实现**：
```go
type DB struct {
    master *sql.DB
    slaves []*sql.DB
}

func (db *DB) Query(sql string) (*sql.Rows, error) {
    // 随机选择从库
    slave := db.slaves[rand.Intn(len(db.slaves))]
    return slave.Query(sql)
}

func (db *DB) Exec(sql string, args ...interface{}) (sql.Result, error) {
    return db.master.Exec(sql, args...)
}
```

---

**Q19：如何处理MySQL的主从延迟问题？**

**A19：**

**延迟原因分析：**

| 原因 | 表现 |
|------|------|
| 网络延迟 | Seconds_Behind_Master持续增加 |
| 大事务 | 单个事务执行时间长 |
| 索引缺失 | 从库SQL执行慢 |
| 硬件不足 | CPU/IO使用率高 |

**解决方案：**

1. **并行复制**：
```ini
slave_parallel_workers = 4
slave_parallel_type = LOGICAL_CLOCK
```

2. **半同步复制**：
```sql
SET GLOBAL rpl_semi_sync_master_enabled = 1;
SET GLOBAL rpl_semi_sync_slave_enabled = 1;
```

3. **优化大事务**：
```sql
-- 分批处理
WHILE EXISTS (SELECT 1 FROM large_table WHERE processed = 0 LIMIT 1) DO
    UPDATE large_table SET processed = 1 WHERE processed = 0 LIMIT 1000;
    COMMIT;
END WHILE;
```

4. **监控告警**：
```
告警阈值：Seconds_Behind_Master > 30s
```

---

**Q20：如何设计MySQL的表结构？**

**A20：**

**设计原则：**

1. **选择合适的数据类型**：
```sql
-- 不好
CREATE TABLE user (
    id BIGINT,           -- 使用INT即可
    name VARCHAR(255),   -- 实际不需要这么长
    phone VARCHAR(50)    -- 使用CHAR(11)更合适
);

-- 好
CREATE TABLE user (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50),
    phone CHAR(11)
);
```

2. **合理使用索引**：
```sql
-- 避免过度索引
CREATE INDEX idx_email ON users(email);
-- 删除不使用的索引
DROP INDEX idx_unused ON users;
```

3. **避免NULL字段**：
```sql
-- 使用默认值代替NULL
CREATE TABLE orders (
    status TINYINT DEFAULT 0,
    deleted_at DATETIME DEFAULT NULL
);
```

4. **分区表设计**：
```sql
-- 按时间分区
CREATE TABLE logs (...) PARTITION BY RANGE (TO_DAYS(create_time));
```

5. **外键约束**：
```sql
-- 需要数据完整性时使用
CREATE TABLE order_items (
    order_id INT,
    FOREIGN KEY (order_id) REFERENCES orders(id) ON DELETE CASCADE
);
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

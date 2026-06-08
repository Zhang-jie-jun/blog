# 中间件技术

## 目录结构

| 文档 | 内容 |
|------|------|
| [MySQL详解](mysql.md) | InnoDB引擎、集群架构、主从复制、Binlog、性能优化、常见问题、面试题 |
| [Redis详解](redis.md) | 数据结构原理、持久化机制、主从复制、哨兵模式、集群模式、缓存策略、常见问题、面试题 |
| [Gin详解](gin.md) | Radix树路由、中间件机制、Context上下文、参数绑定、错误处理、常见问题、面试题 |
| [gRPC详解](grpc.md) | Protocol Buffers、HTTP/2协议、流式调用、拦截器、负载均衡、常见问题、面试题 |
| [JWT详解](jwt.md) | Token结构、签名算法、验证流程、安全实践、常见问题、面试题 |
| [GORM详解](gorm.md) | SQL生成机制、连接池管理、事务管理、关联查询、常见问题、面试题 |
| [Consul详解](consul.md) | Raft共识、服务发现、KV存储、Gossip协议、健康检查、常见问题、面试题 |
| [etcd详解](etcd.md) | Raft共识、MVCC、Watch机制、TTL、集群管理、常见问题、面试题 |
| [Elasticsearch详解](elasticsearch.md) | 全文检索、索引管理、查询DSL、聚合分析、集群架构、常见问题、面试题 |
| [Cassandra详解](cassandra.md) | 分布式存储、数据模型、复制策略、一致性级别、常见问题、面试题 |
| [Thrift详解](thrift.md) | IDL定义、协议层、传输层、跨语言调用、常见问题、面试题 |
| [Kafka详解](kafka.md) | 分布式流处理、Topic/Partition、副本机制、Producer/Consumer、常见问题、面试题 |
| [NATS详解](nats.md) | 高性能消息系统、发布/订阅、队列组、JetStream、常见问题、面试题 |
| [Nginx详解](nginx.md) | HTTP服务器、反向代理、负载均衡、事件驱动、常见问题、面试题 |

## 中间件分类

### 数据库中间件
- **MySQL**：关系型数据库管理系统，支持事务、索引、主从复制
- **Redis**：内存数据结构存储，支持多种数据结构、持久化、高可用

### Web框架
- **Gin**：高性能Go Web框架，基于Radix树路由、中间件机制

### RPC框架
- **gRPC**：高性能RPC框架，基于HTTP/2、Protocol Buffers
- **Thrift**：跨语言RPC框架，支持多种协议和传输层

### 认证授权
- **JWT**：JSON Web Token，无状态认证机制

### ORM框架
- **GORM**：Go语言ORM库，提供优雅的数据库操作API

### 服务发现与配置
- **Consul**：服务发现、健康检查、KV存储、多数据中心
- **etcd**：分布式键值存储、Raft共识、Watch机制

### 搜索与分析
- **Elasticsearch**：全文搜索、日志分析、实时数据分析

### 分布式存储
- **Cassandra**：分布式NoSQL数据库、高可用、线性扩展

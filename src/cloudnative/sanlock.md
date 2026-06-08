# SANlock 详解

## SANlock 是什么？

SANlock 是一个用于管理共享存储锁的开源工具，主要用于虚拟化环境中确保多个节点对共享存储资源的安全访问。它提供了分布式锁机制，支持多种锁类型（如排它锁、共享锁），可以防止多个虚拟机同时访问同一存储资源导致数据损坏。SANlock 常与 libvirt、KVM 等虚拟化技术配合使用，是构建高可用性虚拟化集群的关键组件。

## SANlock 的特点

| 特性 | 说明 |
|------|------|
| **分布式锁** | 支持跨节点的分布式锁管理 |
| **多种锁类型** | 支持排它锁、共享锁、读锁等 |
| **高可用性** | 支持故障检测和自动故障转移 |
| **持久化存储** | 锁状态持久化到共享存储 |
| **心跳机制** | 通过心跳检测节点状态 |
| **多种存储支持** | 支持 SAN、iSCSI、NFS 等共享存储 |

---

## 一、SANlock 核心原理

### 1.1 SANlock 架构

SANlock 采用客户端-服务器架构：

```
┌─────────────────────────────────────────────────────────────┐
│                    SANlock 架构                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐              │
│  │ Node 1   │    │ Node 2   │    │ Node 3   │              │
│  │ KVM/libvirt│  │ KVM/libvirt│  │ KVM/libvirt│            │
│  └────┬─────┘    └────┬─────┘    └────┬─────┘              │
│       │               │               │                      │
│       ▼               ▼               ▼                      │
│  ┌─────────────────────────────────────────┐                │
│  │          SANlock Client                 │                │
│  │   (锁请求、心跳检测、状态同步)           │                │
│  └─────────────────────────────────────────┘                │
│                        │                                    │
│                        ▼                                    │
│  ┌─────────────────────────────────────────┐                │
│  │           Shared Storage               │                │
│  │  (SAN/iSCSI/NFS - 存储锁状态和心跳)     │                │
│  └─────────────────────────────────────────┘                │
│                        │                                    │
│                        ▼                                    │
│  ┌─────────────────────────────────────────┐                │
│  │          SANlock Server                 │                │
│  │   (锁管理、租约管理、故障检测)           │                │
│  └─────────────────────────────────────────┘                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 锁机制原理

SANlock 实现了基于租约的锁机制：

1. **锁请求**：客户端向 SANlock 服务器请求锁
2. **租约获取**：服务器授予锁并设置租约时间
3. **心跳维持**：客户端定期更新心跳以保持租约
4. **锁释放**：客户端主动释放锁或租约到期自动释放

```
锁获取流程:
[Client] → [请求锁] → [Server] → [检查锁状态] → [授予租约] → [Client持有锁]

心跳流程:
[Client] → [定期发送心跳] → [Server] → [更新租约时间] → [保持锁状态]

锁释放流程:
[Client] → [释放请求] → [Server] → [释放锁] → [锁可用]
```

### 1.3 故障检测机制

SANlock 通过心跳机制检测节点故障：

- **心跳间隔**：默认每 2 秒发送一次心跳
- **超时时间**：租约到期时间（默认 60 秒）
- **故障判定**：超过租约时间未收到心跳则判定节点故障
- **自动故障转移**：故障节点的锁自动释放，其他节点可以获取

### 1.4 锁类型

SANlock 支持多种锁类型：

| 锁类型 | 说明 | 适用场景 |
|-------|------|---------|
| **排它锁 (EXCL)** | 独占访问，其他节点无法获取 | 虚拟机启动、磁盘写入 |
| **共享锁 (SHRD)** | 多个节点可同时持有 | 只读访问、共享数据 |
| **读锁 (READ)** | 只读访问，不阻塞其他读锁 | 数据读取、备份 |
| **写锁 (WRITE)** | 独占写入，阻塞所有其他锁 | 数据写入、修改 |

---

## 二、SANlock 使用示例

### 2.1 安装 SANlock

```bash
# CentOS/RHEL
yum install sanlock sanlock-lib

# Debian/Ubuntu
apt-get install sanlock

# 启动服务
systemctl start sanlock
systemctl enable sanlock
```

### 2.2 配置 SANlock

```bash
# 创建共享存储目录
mkdir -p /mnt/shared-storage
mount -t nfs nfs-server:/exports /mnt/shared-storage

# 配置 SANlock 目录
mkdir -p /var/lib/sanlock
chown sanlock:sanlock /var/lib/sanlock

# 配置 sanlock.conf
cat > /etc/sanlock/sanlock.conf <<EOF
# SANlock 配置
host_name = node1
lockspace_dir = /mnt/shared-storage/sanlock
log_level = info
EOF
```

### 2.3 创建锁空间

```bash
# 创建锁空间
sanlock lockspace create mylockspace /mnt/shared-storage/sanlock/mylockspace

# 查看锁空间
sanlock lockspace list

# 删除锁空间
sanlock lockspace remove mylockspace
```

### 2.4 获取和释放锁

```bash
# 获取排它锁
sanlock client acquire -s mylockspace -n vm1 -t excl

# 获取共享锁
sanlock client acquire -s mylockspace -n vm1 -t shrd

# 查看锁状态
sanlock client status

# 释放锁
sanlock client release -s mylockspace -n vm1
```

### 2.5 租约管理

```bash
# 设置租约时间（秒）
sanlock client acquire -s mylockspace -n vm1 -t excl -l 120

# 更新租约
sanlock client renew -s mylockspace -n vm1

# 查看租约状态
sanlock client info
```

### 2.6 集成 libvirt

```xml
<!-- 在 libvirt XML 中配置 SANlock -->
<domain type='kvm'>
  <name>vm1</name>
  <lock>
    <lockspace>mylockspace</lockspace>
    <name>vm1</name>
    <type>sanlock</type>
  </lock>
  <devices>
    <disk type='file' device='disk'>
      <source file='/mnt/shared-storage/vm1/disk.qcow2'/>
      <target dev='vda' bus='virtio'/>
    </disk>
  </devices>
</domain>
```

---

## 三、常见问题与排查

### 问题 1：SANlock 服务无法启动

**现象**：执行 `systemctl start sanlock` 后服务无法启动

**排查步骤**：
```bash
# 查看服务状态
systemctl status sanlock

# 查看日志
journalctl -u sanlock

# 检查权限
ls -la /var/lib/sanlock
id sanlock
```

**解决方案**：
- 确保 sanlock 用户存在且有权限访问目录
- 检查配置文件格式是否正确
- 确保共享存储已挂载且可读写

### 问题 2：锁获取失败

**现象**：执行 `sanlock client acquire` 时报错

**排查步骤**：
```bash
# 检查锁空间是否存在
sanlock lockspace list

# 检查锁状态
sanlock client status

# 检查共享存储连接
mount | grep shared-storage
```

**解决方案**：
- 确保锁空间已创建
- 检查锁是否被其他节点持有
- 确保共享存储可访问

### 问题 3：心跳超时

**现象**：SANlock 报告心跳超时

**排查步骤**：
```bash
# 检查网络连接
ping other-node
nc -zv other-node 30031

# 检查防火墙
iptables -L | grep 30031
firewall-cmd --list-all

# 查看心跳日志
journalctl -u sanlock | grep heartbeat
```

**解决方案**：
- 确保节点间网络畅通
- 开放 SANlock 端口（默认 30031）
- 调整心跳间隔和超时时间

### 问题 4：锁无法释放

**现象**：节点故障后锁无法自动释放

**排查步骤**：
```bash
# 检查锁状态
sanlock client status

# 检查租约时间
sanlock client info | grep lease

# 强制释放锁
sanlock client release -s mylockspace -n vm1 -f
```

**解决方案**：
- 等待租约到期自动释放
- 使用 `-f` 参数强制释放
- 检查故障节点状态

### 问题 5：性能下降

**现象**：SANlock 操作响应缓慢

**排查步骤**：
```bash
# 检查存储性能
iostat -x 1

# 检查网络延迟
ping -f other-node

# 检查 SANlock 日志
journalctl -u sanlock | grep latency
```

**解决方案**：
- 优化共享存储性能
- 使用更快的网络（如 10Gbps）
- 调整锁的粒度

### 问题 6：节点脑裂

**现象**：多个节点同时认为自己持有锁

**排查步骤**：
```bash
# 检查各节点锁状态
sanlock client status

# 检查共享存储一致性
ls -la /mnt/shared-storage/sanlock/

# 检查网络分区
ip addr show
```

**解决方案**：
- 使用仲裁机制（如 QDevice）
- 配置 fencing 设备
- 确保网络稳定

### 问题 7：权限问题

**现象**：SANlock 无法访问共享存储

**排查步骤**：
```bash
# 检查目录权限
ls -la /mnt/shared-storage/

# 检查 SELinux
getenforce
ls -Z /mnt/shared-storage/

# 检查 ACL
getfacl /mnt/shared-storage/
```

**解决方案**：
- 设置正确的目录权限
- 配置 SELinux 策略
- 设置合适的 ACL

### 问题 8：锁空间损坏

**现象**：锁空间文件损坏导致无法获取锁

**排查步骤**：
```bash
# 检查锁空间完整性
sanlock lockspace check mylockspace

# 尝试修复
sanlock lockspace repair mylockspace
```

**解决方案**：
- 使用 `lockspace repair` 修复
- 从备份恢复锁空间
- 重建锁空间（需要所有节点配合）

### 问题 9：租约时间过短

**现象**：频繁出现租约过期

**排查步骤**：
```bash
# 查看当前租约时间
sanlock client info | grep lease

# 检查网络延迟
ping -c 10 other-node | grep rtt
```

**解决方案**：
- 增加租约时间（`-l` 参数）
- 优化网络性能
- 调整心跳间隔

### 问题 10：与 libvirt 集成问题

**现象**：libvirt 无法使用 SANlock

**排查步骤**：
```bash
# 检查 libvirt 配置
virsh dumpxml vm1 | grep lock

# 检查 libvirt 日志
journalctl -u libvirtd | grep sanlock

# 检查 sanlock 客户端状态
sanlock client status
```

**解决方案**：
- 确保 XML 配置正确
- 确保 libvirt 有权限访问 SANlock
- 检查 SELinux 策略

---

## 四、场景面试题

### 基础问题

1. **SANlock 是什么？它的主要作用是什么？**
   - SANlock 是共享存储锁管理工具
   - 用于虚拟化环境中的分布式锁管理
   - 确保多个节点安全访问共享存储

2. **SANlock 支持哪些锁类型？**
   - 排它锁 (EXCL)
   - 共享锁 (SHRD)
   - 读锁 (READ)
   - 写锁 (WRITE)

3. **SANlock 的心跳机制是如何工作的？**
   - 客户端定期发送心跳（默认每 2 秒）
   - 服务器更新租约时间
   - 超过租约时间未收到心跳则判定故障

4. **什么是租约？租约到期会发生什么？**
   - 租约是锁的有效期限
   - 到期后锁自动释放
   - 其他节点可以获取该锁

5. **SANlock 如何检测节点故障？**
   - 通过心跳机制检测
   - 租约到期未收到心跳判定故障
   - 自动释放故障节点的锁

6. **SANlock 支持哪些共享存储类型？**
   - SAN、iSCSI、NFS、Ceph 等

7. **如何创建 SANlock 锁空间？**
   - 使用 `sanlock lockspace create` 命令
   - 指定锁空间名称和目录

8. **SANlock 与 libvirt 如何集成？**
   - 在虚拟机 XML 中配置 `<lock>` 元素
   - 指定锁空间名称和类型

9. **如何强制释放一个锁？**
   - 使用 `-f` 参数：`sanlock client release -f`
   - 需要管理员权限

10. **SANlock 的默认端口是多少？**
    - 默认端口是 30031

### 复杂场景问题

11. **在高可用性集群中，如何设计 SANlock 的部署方案？**
    - **分析思路**：考虑冗余、性能、故障转移
    - **解决方案**：
      - 部署多个 SANlock 服务器实现冗余
      - 使用共享存储（如 Ceph）存储锁状态
      - 配置 fencing 设备防止脑裂

12. **当 SANlock 出现脑裂问题时，如何处理？**
    - **分析思路**：需要检测和解决节点间的分歧
    - **解决方案**：
      - 使用仲裁机制（QDevice）
      - 配置 STONITH（Shoot The Other Node In The Head）
      - 定期检查锁状态一致性

13. **如何优化 SANlock 的性能？**
    - **分析思路**：从存储、网络、配置三个维度优化
    - **解决方案**：
      - 使用高性能共享存储（SSD）
      - 使用低延迟网络（10Gbps+）
      - 调整租约时间和心跳间隔

14. **在大规模集群中，如何管理大量锁？**
    - **分析思路**：考虑锁的粒度、命名规范、监控
    - **解决方案**：
      - 使用分层锁管理
      - 建立锁命名规范
      - 集成监控系统（Prometheus + Grafana）

15. **SANlock 与其他分布式锁方案（如 ZooKeeper、etcd）有什么区别？**
    - **分析思路**：从一致性模型、性能、适用场景对比
    - **解决方案**：
      - SANlock：基于共享存储，适合虚拟化场景
      - ZooKeeper/etcd：基于一致性协议，适合分布式系统
      - 根据场景选择合适的锁方案

16. **如何实现 SANlock 的备份和恢复？**
    - **分析思路**：考虑锁状态的持久化和恢复
    - **解决方案**：
      - 定期备份锁空间目录
      - 使用快照技术备份共享存储
      - 制定恢复流程和测试计划

17. **在混合云环境中，如何使用 SANlock 管理跨云存储？**
    - **分析思路**：考虑网络连接、延迟、兼容性
    - **解决方案**：
      - 使用 VPN 连接不同云环境
      - 使用统一的共享存储（如 Ceph RBD）
      - 调整租约时间适应网络延迟

18. **如何监控 SANlock 的运行状态？**
    - **分析思路**：监控心跳、锁状态、性能指标
    - **解决方案**：
      - 使用 `sanlock client status` 查看状态
      - 解析日志获取指标
      - 集成到监控系统

19. **SANlock 的安全性如何保障？**
    - **分析思路**：从访问控制、数据加密、审计三个方面考虑
    - **解决方案**：
      - 限制 SANlock 目录权限
      - 使用加密存储传输
      - 记录操作日志进行审计

20. **当共享存储故障时，SANlock 如何处理？**
    - **分析思路**：考虑故障检测、数据保护、恢复策略
    - **解决方案**：
      - 检测存储故障并通知所有节点
      - 暂停锁操作防止数据损坏
      - 故障恢复后同步锁状态


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

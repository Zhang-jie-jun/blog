# OpenStack详解

## OpenStack概述

OpenStack是一个开源的云计算管理平台，用于构建和管理公有云和私有云基础设施。它提供了计算、存储、网络等核心服务，支持大规模虚拟机部署和管理。

### 核心组件架构

```
                              ┌──────────────────────┐
                              │    Horizon (UI)     │
                              └──────────┬───────────┘
                                         │
        ┌────────────────────────────────┼────────────────────────────────┐
        │                                │                                │
┌───────▼───────┐              ┌─────────▼─────────┐              ┌───────▼───────┐
│   Keystone    │              │      Nova         │              │   Neutron     │
│  (认证服务)   │              │   (计算服务)       │              │  (网络服务)   │
└───────┬───────┘              └─────────┬─────────┘              └───────┬───────┘
        │                                │                                │
┌───────▼───────┐              ┌─────────▼─────────┐              ┌───────▼───────┐
│   Glance      │              │    Cinder         │              │    Swift      │
│  (镜像服务)   │              │   (块存储服务)     │              │  (对象存储)   │
└───────────────┘              └───────────────────┘              └───────────────┘
```

---

## 虚拟机创建流程详解

### 创建流程架构图

```
用户请求
    │
    ▼
Horizon/CLI → Keystone认证
    │
    ▼
Nova API → 验证请求参数
    │
    ▼
Nova Scheduler → 选择合适的计算节点
    │
    ▼
Nova Conductor → 协调创建任务
    │
    ▼
Nova Compute (目标节点)
    │
    ├─► 创建VM目录和配置文件
    │
    ├─► 从Glance获取镜像
    │
    ├─► 创建虚拟磁盘(从Cinder获取卷)
    │
    ├─► 配置虚拟网络(Neutron)
    │
    └─► 启动VM实例
```

### 创建流程步骤详解

**步骤1：用户请求**
```bash
# 通过CLI创建虚拟机
openstack server create --image cirros --flavor m1.small --key-name mykey --network private-net my-instance

# 参数说明：
# --image: 指定镜像名称或ID
# --flavor: 指定虚拟机规格(CPU/内存/磁盘)
# --key-name: 指定SSH密钥对
# --network: 指定网络
```

**步骤2：认证与授权**
```python
# Keystone认证流程
1. 用户提供用户名/密码/项目名
2. Keystone验证凭证
3. 返回Token和服务目录
4. Nova使用Token访问其他服务
```

**步骤3：Nova API处理**
```python
# Nova API验证请求
1. 验证参数合法性
2. 检查配额限制
3. 创建Instance记录到数据库
4. 调用Scheduler选择节点
```

**步骤4：调度器选择节点**
```python
# Scheduler过滤和权重计算
1. 过滤器阶段：
   - AvailabilityZoneFilter: 检查可用域
   - RamFilter: 检查内存资源
   - DiskFilter: 检查磁盘资源
   - ComputeFilter: 检查计算节点状态

2. 权重计算阶段：
   - RamWeigher: 内存利用率权重
   - DiskWeigher: 磁盘利用率权重
   - LoadWeigher: 负载权重
```

**步骤5：创建虚拟机**
```bash
# Nova Compute执行的操作
1. 创建实例目录: /var/lib/nova/instances/<instance_id>/
2. 获取镜像: glance image-download <image_id>
3. 创建磁盘: qemu-img create -f qcow2 disk.qcow2 10G
4. 配置网络: neutron port-create --network private-net
5. 生成libvirt XML配置
6. 启动虚拟机: virsh start <instance_id>
```

---

## 虚拟机启动流程

### 启动流程架构图

```
启动请求
    │
    ▼
Nova API → 获取实例状态
    │
    ▼
Nova Compute → 检查实例状态
    │
    ├─► 状态=SHUTOFF/STOPPED
    │       │
    │       ▼
    │   检查资源可用性
    │       │
    │       ▼
    │   生成libvirt XML
    │       │
    │       ▼
    │   启动虚拟机
    │       │
    │       ▼
    │   更新状态为ACTIVE
    │
    └─► 状态=其他
            │
            ▼
        返回错误信息
```

### 启动流程详解

**阶段1：请求处理**
```python
# 启动请求处理流程
1. 接收启动请求
2. 验证实例状态(必须为SHUTOFF/STOPPED)
3. 检查计算节点状态
4. 锁定实例防止并发操作
```

**阶段2：资源准备**
```bash
# 资源检查和准备
1. 检查计算节点资源(CPU/内存/磁盘)
2. 检查网络端口状态
3. 检查卷连接状态(Cinder卷)
4. 恢复虚拟机配置
```

**阶段3：虚拟机启动**
```bash
# libvirt启动命令
virsh create /var/lib/nova/instances/<instance_id>/libvirt.xml

# libvirt XML配置示例关键部分
<domain type='kvm'>
  <name>instance-00000001</name>
  <memory unit='KiB'>2097152</memory>
  <vcpu placement='static'>2</vcpu>
  <devices>
    <disk type='file' device='disk'>
      <source file='/var/lib/nova/instances/<id>/disk'/>
      <target dev='vda' bus='virtio'/>
    </disk>
    <interface type='bridge'>
      <source bridge='br-int'/>
      <virtualport type='openvswitch'>
        <parameters interfaceid='<port-uuid>'/>
      </virtualport>
    </interface>
  </devices>
</domain>
```

**阶段4：状态更新**
```python
# 状态更新流程
1. 检查虚拟机是否成功启动
2. 更新数据库中实例状态为ACTIVE
3. 通知其他组件(Neutron/Cinder)
4. 释放实例锁
```

---

## 虚拟机迁移流程

### 迁移类型对比

| 迁移类型 | 描述 | 停机时间 | 适用场景 |
|---------|------|---------|---------|
| **冷迁移** | 关闭VM后迁移 | 较长(分钟级) | 计划性维护、资源重新分配 |
| **热迁移(Live Migration)** | VM运行中迁移 | 极短(秒级) | 无停机维护、负载均衡 |
| **块迁移(Block Migration)** | 仅迁移磁盘数据 | 取决于磁盘大小 | 存储迁移、存储维护 |

### 热迁移流程架构图

```
迁移请求
    │
    ▼
Nova API → 验证请求
    │
    ▼
Nova Scheduler → 选择目标节点
    │
    ▼
Nova Conductor → 协调迁移
    │
    ┌─────────────┴─────────────┐
    │                           │
    ▼                           ▼
源节点Compute            目标节点Compute
    │                           │
    ├─► 冻结VM状态              ├─► 准备接收
    │                           │
    ├─► 传输内存数据            ├─► 接收内存数据
    │                           │
    ├─► 增量同步内存            ├─► 增量接收
    │                           │
    ├─► 传输设备状态            ├─► 恢复设备状态
    │                           │
    ├─► 切换到目标节点          ├─► 启动VM
    │                           │
    └─► 清理源节点资源          └─► 更新状态
```

### 热迁移流程详解

**阶段1：准备阶段**
```bash
# 检查迁移条件
1. 源节点和目标节点必须在同一集群
2. 目标节点有足够资源(CPU/内存/磁盘)
3. 网络连通性正常
4. 存储可访问(共享存储或块迁移)

# 执行迁移命令
openstack server migrate --live <instance_id> --target-host <target_host>
```

**阶段2：预拷贝阶段(Pre-copy)**
```python
# 预拷贝流程
1. 源节点暂停VM写入(短暂)
2. 生成内存快照
3. 传输内存数据到目标节点
4. 恢复源节点VM运行
5. 重复增量同步(脏页追踪)
```

**阶段3：停止与切换阶段(Stop-and-Copy)**
```python
# 最终切换流程
1. 源节点冻结VM
2. 传输剩余脏页
3. 传输设备状态(网卡、磁盘等)
4. 目标节点启动VM
5. 更新网络路由
6. 源节点清理资源
```

**阶段4：Post-copy模式(可选)**
```python
# Post-copy模式
1. 先启动目标节点VM
2. VM运行时按需拉取内存页
3. 适用于内存大、网络带宽高的场景
4. 可能产生性能抖动
```

### 冷迁移流程

```bash
# 冷迁移步骤
1. 关闭虚拟机
openstack server stop <instance_id>

2. 执行冷迁移
openstack server migrate <instance_id>

3. 启动虚拟机(可选)
openstack server start <instance_id>

# 冷迁移特点
- 需要关闭VM
- 无共享存储要求
- 迁移整个磁盘镜像
- 适合计划性维护
```

---

## 常见问题与故障排查

### 问题1：虚拟机创建失败 - No valid host was found

**现象**：
```
Error: No valid host was found. No available host suitable for scheduling
```

**排查步骤**：
```bash
# 1. 检查计算节点状态
openstack compute service list

# 2. 检查节点资源
nova hypervisor-list
nova hypervisor-stats

# 3. 检查调度器日志
tail -f /var/log/nova/nova-scheduler.log

# 4. 检查实例规格
openstack flavor show m1.small

# 5. 检查可用域
openstack availability zone list
```

**解决方案**：
1. 启动离线的计算节点服务
2. 检查资源配额
3. 调整实例规格
4. 检查调度器过滤器配置

### 问题2：热迁移失败

**现象**：
```
Error: Live migration failed
```

**排查步骤**：
```bash
# 1. 检查源节点和目标节点连通性
ping <target_host>
ssh <target_host>

# 2. 检查libvirt配置
virsh capabilities

# 3. 检查存储访问
ls -la /var/lib/nova/instances/

# 4. 检查迁移日志
tail -f /var/log/nova/nova-compute.log
```

**解决方案**：
1. 确保节点间SSH免密登录
2. 配置共享存储(NFS/Ceph)
3. 检查libvirt版本兼容性
4. 增加迁移超时时间

### 问题3：虚拟机无法启动

**现象**：
```
Error: Instance failed to start
```

**排查步骤**：
```bash
# 1. 检查实例状态
openstack server show <instance_id>

# 2. 检查libvirt状态
virsh list --all
virsh domstate <instance_id>

# 3. 检查计算节点日志
tail -f /var/log/nova/nova-compute.log

# 4. 检查磁盘文件
ls -la /var/lib/nova/instances/<instance_id>/

# 5. 尝试手动启动
virsh start <instance_id>
```

**解决方案**：
1. 修复损坏的磁盘镜像
2. 检查网络配置
3. 重启计算节点服务
4. 重建虚拟机

---

## 场景面试题

### 基础问题

1. **OpenStack的核心组件有哪些？各自的作用是什么？**
   - Nova: 计算服务，管理虚拟机生命周期
   - Neutron: 网络服务，管理虚拟网络
   - Glance: 镜像服务，管理虚拟机镜像
   - Cinder: 块存储服务，管理持久化卷
   - Keystone: 认证服务，管理用户和权限
   - Horizon: 仪表板，提供Web管理界面

2. **虚拟机创建流程中，Scheduler的作用是什么？**
   - Scheduler负责选择合适的计算节点来创建虚拟机
   - 通过过滤器(filter)排除不满足条件的节点
   - 通过权重计算(weighting)选择最优节点
   - 支持多种调度策略(如资源利用率、负载均衡)

3. **热迁移和冷迁移有什么区别？**
   - 热迁移：虚拟机运行中迁移，停机时间极短
   - 冷迁移：需要先关闭虚拟机，停机时间较长
   - 热迁移需要共享存储或块迁移支持
   - 冷迁移不需要共享存储

4. **什么是预拷贝(Pre-copy)迁移？**
   - 预拷贝是热迁移的一种方式
   - 在VM运行时先传输大部分内存数据
   - 通过脏页追踪进行增量同步
   - 最后短暂冻结VM传输剩余数据

5. **Nova Conductor的作用是什么？**
   - 协调Nova组件之间的通信
   - 处理数据库操作，避免Compute直接访问数据库
   - 执行异步任务
   - 提高系统安全性和可扩展性

### 复杂场景问题

6. **如何设计支持500-1000台VM的OpenStack平台？**
   - **计算层**：多个Compute节点，配置足够的CPU/内存
   - **网络层**：Spine-Leaf架构，VXLAN虚拟网络
   - **存储层**：Ceph分布式存储，支持块存储和对象存储
   - **控制层**：多节点HA部署，负载均衡
   - **监控**：集成Prometheus+Grafana监控

7. **热迁移失败可能有哪些原因？如何排查？**
   - **原因**：网络不通、存储不可访问、资源不足、版本不兼容
   - **排查**：
     1. 检查节点间网络连通性
     2. 检查共享存储挂载
     3. 检查目标节点资源
     4. 查看nova-compute日志
     5. 检查libvirt配置

8. **如何实现OpenStack高可用性？**
   - **控制节点**：多节点部署，使用Pacemaker/Corosync
   - **数据库**：MySQL/MariaDB主从复制或Galera集群
   - **消息队列**：RabbitMQ集群
   - **网络**：Neutron多节点部署，冗余网络设备
   - **存储**：分布式存储(Ceph)，多副本

9. **虚拟机启动慢可能是什么原因？**
   - **存储问题**：磁盘IO慢，镜像下载慢
   - **网络问题**：Neutron配置复杂，DHCP响应慢
   - **计算节点问题**：负载高，资源不足
   - **镜像问题**：镜像过大，启动脚本复杂

10. **如何优化虚拟机创建速度？**
    - 使用本地缓存镜像
    - 优化存储IO(使用SSD)
    - 简化网络配置
    - 使用预配置的镜像模板
    - 调整Scheduler策略

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

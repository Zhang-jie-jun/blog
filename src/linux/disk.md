# 磁盘管理完整指南

## 概述

磁盘管理是Linux系统管理的重要部分，包括磁盘发现、分区、格式化、挂载等操作。本指南详细介绍从新增磁盘到配置自动挂载的完整流程。

---

## 一、磁盘发现与识别

### 查看磁盘设备

```bash
# 查看所有块设备
lsblk

# 查看磁盘详细信息
fdisk -l

# 查看磁盘分区表
parted -l

# 查看磁盘UUID
blkid
```

### 识别新增磁盘

```bash
# 扫描SCSI总线，发现新磁盘
echo "- - -" > /sys/class/scsi_host/hostX/scan

# 重新读取分区表
partprobe /dev/sdb

# 查看新磁盘信息
lsblk /dev/sdb
```

---

## 二、GPT分区模式

### GPT vs MBR

| 特性 | MBR | GPT |
|------|-----|-----|
| 支持磁盘大小 | 最大2TB | 最大9.4ZB |
| 主分区数量 | 最多4个 | 最多128个 |
| 分区表备份 | 无 | 有 |
| UEFI支持 | 不支持 | 支持 |

### 创建GPT分区

```bash
# 使用parted创建GPT分区表
parted /dev/sdb mklabel gpt

# 创建主分区（占用全部空间）
parted /dev/sdb mkpart primary 0% 100%

# 创建多个分区
parted /dev/sdb mkpart primary 0% 50%
parted /dev/sdb mkpart primary 50% 100%

# 查看分区结果
parted -l /dev/sdb
```

### 查看GPT分区信息

```bash
# 查看GPT分区详情
gdisk -l /dev/sdb

# 查看分区类型
lsblk -f /dev/sdb
```

---

## 三、完整流程：新增磁盘配置

### 步骤1：发现磁盘

```bash
# 查看系统中的磁盘
lsblk

# 假设新磁盘为 /dev/sdb
```

### 步骤2：创建分区

```bash
# 创建GPT分区表
parted /dev/sdb mklabel gpt

# 创建分区（全部空间）
parted /dev/sdb mkpart primary 0% 100%

# 验证分区
lsblk /dev/sdb
```

### 步骤3：格式化分区

```bash
# 格式化为ext4
mkfs.ext4 /dev/sdb1

# 格式化为xfs
mkfs.xfs /dev/sdb1

# 格式化为NTFS
mkfs.ntfs /dev/sdb1

# 查看格式化结果
blkid /dev/sdb1
```

### 步骤4：挂载分区

```bash
# 创建挂载点
mkdir -p /mnt/data

# 临时挂载
mount /dev/sdb1 /mnt/data

# 验证挂载
df -h /mnt/data
```

### 步骤5：配置自动挂载

```bash
# 获取分区UUID
blkid /dev/sdb1

# 编辑fstab
vim /etc/fstab

# 添加以下内容（替换UUID为实际值）
UUID=12345678-1234-5678-90ab-cdef01234567 /mnt/data ext4 defaults 0 2

# 测试挂载
mount -a

# 验证自动挂载配置
df -h /mnt/data
```

---

## 四、磁盘分区管理工具

### parted命令

```bash
# 创建分区表
parted /dev/sdb mklabel gpt

# 创建分区
parted /dev/sdb mkpart primary 0% 100%

# 删除分区
parted /dev/sdb rm 1

# 调整分区大小
parted /dev/sdb resizepart 1 0% 70%
```

### gdisk命令

```bash
# 进入交互模式
gdisk /dev/sdb

# 命令：
# n - 创建新分区
# d - 删除分区
# w - 保存并退出
# q - 退出不保存
```

### fdisk命令（MBR模式）

```bash
# 创建MBR分区表
fdisk /dev/sdb

# 命令：
# n - 创建新分区
# p - 主分区
# e - 扩展分区
# w - 保存并退出
```

---

## 五、常见问题

### 问题1：磁盘无法识别

```bash
# 检查磁盘连接
dmesg | grep -i sdb

# 重新扫描SCSI总线
echo "- - -" > /sys/class/scsi_host/host0/scan
echo "- - -" > /sys/class/scsi_host/host1/scan
echo "- - -" > /sys/class/scsi_host/host2/scan
```

### 问题2：分区格式化失败

```bash
# 检查磁盘状态
smartctl -a /dev/sdb

# 检查是否有坏道
badblocks -v /dev/sdb1
```

### 问题3：挂载失败

```bash
# 检查fstab语法
mount -a

# 检查文件系统
fsck /dev/sdb1
```

---

## 六、面试题

**Q1：如何创建GPT分区？**
```bash
parted /dev/sdb mklabel gpt
parted /dev/sdb mkpart primary 0% 100%
```

**Q2：如何配置开机自动挂载？**
```bash
# 在/etc/fstab中添加
UUID=xxx /mnt/data ext4 defaults 0 2
```

**Q3：GPT和MBR有什么区别？**
- GPT支持更大磁盘（最大9.4ZB）
- GPT支持更多分区（最多128个）
- GPT有分区表备份
- GPT支持UEFI启动

**Q4：如何重新扫描磁盘？**
```bash
echo "- - -" > /sys/class/scsi_host/hostX/scan
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

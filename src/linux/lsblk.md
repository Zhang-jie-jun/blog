# lsblk - 列出块设备

## 概述

`lsblk`命令用于列出系统中所有块设备的信息，包括磁盘、分区、挂载点等。它是了解系统存储结构的重要工具。

## 基本语法

```bash
lsblk [选项] [设备]
```

## 常用选项

| 选项 | 说明 |
|------|------|
| `-a` | 显示所有设备（包括空设备） |
| `-b` | 以字节为单位显示大小 |
| `-d` | 只显示磁盘设备，不显示分区 |
| `-f` | 显示文件系统信息（类型、UUID、挂载点） |
| `-m` | 显示权限和所有者信息 |
| `-o` | 自定义输出字段 |
| `-p` | 显示完整路径 |
| `-t` | 显示设备类型信息 |

## 输出字段说明

### 默认输出

| 字段 | 说明 |
|------|------|
| `NAME` | 设备名称 |
| `MAJ:MIN` | 主设备号:次设备号 |
| `RM` | 是否可移除设备（1=是，0=否） |
| `SIZE` | 设备大小 |
| `RO` | 是否只读（1=是，0=否） |
| `TYPE` | 设备类型（disk、part、lvm等） |
| `MOUNTPOINT` | 挂载点（未挂载显示空） |

### 文件系统信息（-f选项）

| 字段 | 说明 |
|------|------|
| `FSTYPE` | 文件系统类型 |
| `LABEL` | 文件系统标签 |
| `UUID` | 文件系统唯一标识符 |
| `MOUNTPOINT` | 挂载点 |

## 使用场景

### 场景1：查看系统存储结构

```bash
# 查看所有块设备
lsblk

# 查看磁盘和分区
lsblk -a

# 只查看磁盘
lsblk -d
```

### 场景2：查看文件系统信息

```bash
# 显示文件系统类型、UUID和挂载点
lsblk -f

# 显示指定设备的文件系统信息
lsblk -f /dev/sda
```

### 场景3：自定义输出

```bash
# 只显示名称、大小、类型和挂载点
lsblk -o NAME,SIZE,TYPE,MOUNTPOINT

# 显示完整路径和UUID
lsblk -p -o NAME,SIZE,UUID,MOUNTPOINT
```

### 场景4：查看权限信息

```bash
lsblk -m
```

## 输出示例

```bash
$ lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda           8:0    0 232.9G  0 disk 
├─sda1        8:1    0   512M  0 part /boot/efi
├─sda2        8:2    0    32G  0 part /
└─sda3        8:3    0 199.4G  0 part /home
sr0          11:0    1  1024M  0 rom  
```

## 问题分析原理

### 常见问题

**问题1：找不到磁盘**

```bash
# 检查磁盘是否被识别
lsblk -d

# 检查磁盘状态
cat /proc/partitions
```

**问题2：分区未显示**

```bash
# 重新读取分区表
partprobe /dev/sda

# 或
hdparm -z /dev/sda
```

**问题3：挂载点为空**

```bash
# 检查是否已挂载
mount | grep /dev/sda1

# 如果未挂载，手动挂载
mkdir -p /mnt/data
mount /dev/sda1 /mnt/data
```

## 面试题

**Q1：如何查看系统中所有磁盘设备？**
```bash
lsblk -d
```

**Q2：如何查看分区的UUID？**
```bash
lsblk -f
```

**Q3：如何查看指定设备的详细信息？**
```bash
lsblk /dev/sda
```

**Q4：lsblk和fdisk有什么区别？**
- `lsblk`：显示块设备的树形结构，更直观
- `fdisk`：用于创建和管理磁盘分区

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

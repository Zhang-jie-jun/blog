# mount - 挂载命令

## 概述

`mount`命令用于将文件系统挂载到Linux系统的指定目录。挂载是Linux系统访问存储设备的方式。

## 基本语法

```bash
mount [选项] 设备 挂载点
```

## 常用选项

| 选项 | 说明 |
|------|------|
| `-t` | 指定文件系统类型 |
| `-o` | 指定挂载选项 |
| `-r` | 只读挂载 |
| `-w` | 读写挂载（默认） |
| `-a` | 挂载/etc/fstab中所有未挂载的文件系统 |
| `-l` | 显示已挂载的文件系统（带卷标） |

## 文件系统类型

| 类型 | 说明 |
|------|------|
| `ext4` | Linux常用文件系统 |
| `xfs` | 高性能日志文件系统 |
| `ntfs` | Windows NT文件系统 |
| `fat32` | FAT32文件系统 |
| `iso9660` | CD/DVD文件系统 |
| `nfs` | 网络文件系统 |
| `cifs` | SMB/CIFS协议（Windows共享） |

## 挂载选项（-o）

| 选项 | 说明 |
|------|------|
| `defaults` | 默认选项（rw,suid,dev,exec,auto,nouser,async） |
| `rw` | 读写模式 |
| `ro` | 只读模式 |
| `noexec` | 不允许执行二进制文件 |
| `nosuid` | 不允许setuid/setgid |
| `nodev` | 不允许访问设备文件 |
| `noatime` | 不更新文件访问时间（提升性能） |
| `nodiratime` | 不更新目录访问时间 |
| `sync` | 同步写入（数据安全） |
| `async` | 异步写入（性能更好） |
| `user` | 允许普通用户挂载 |
| `remount` | 重新挂载已挂载的文件系统 |

## 使用场景

### 场景1：挂载磁盘分区

```bash
# 挂载ext4分区
mount /dev/sda1 /mnt/data

# 指定文件系统类型
mount -t ext4 /dev/sda1 /mnt/data

# 只读挂载
mount -o ro /dev/sda1 /mnt/data
```

### 场景2：挂载U盘

```bash
# 查看U盘设备名
lsblk

# 创建挂载点
mkdir -p /mnt/usb

# 挂载U盘
mount /dev/sdb1 /mnt/usb
```

### 场景3：挂载ISO镜像

```bash
# 创建挂载点
mkdir -p /mnt/cdrom

# 挂载ISO文件
mount -o loop /path/to/image.iso /mnt/cdrom
```

### 场景4：挂载NFS共享

```bash
# 安装nfs客户端
apt install nfs-common  # Debian/Ubuntu
yum install nfs-utils    # CentOS/RHEL

# 挂载NFS共享
mount server:/share /mnt/nfs
```

### 场景5：重新挂载文件系统

```bash
# 将根文件系统重新挂载为读写模式
mount -o remount,rw /

# 添加noatime选项提升性能
mount -o remount,noatime /
```

## 卸载文件系统

```bash
# 卸载挂载点
umount /mnt/data

# 如果设备正忙，强制卸载（慎用）
umount -f /mnt/data

# 延迟卸载
umount -l /mnt/data
```

## 查看已挂载的文件系统

```bash
# 查看所有挂载
mount

# 查看指定类型
mount -t ext4

# 更清晰的格式
df -h
```

## 问题分析原理

### 常见问题

**问题1：挂载时提示"device is busy"**

```bash
# 查找占用挂载点的进程
lsof /mnt/data

# 或者
fuser -v /mnt/data

# 终止占用进程后再卸载
fuser -k /mnt/data
```

**问题2：无法挂载NTFS分区**

```bash
# 需要安装ntfs-3g
apt install ntfs-3g
mount /dev/sda1 /mnt/windows
```

**问题3：挂载后权限问题**

```bash
# 使用uid和gid选项指定用户权限
mount -o uid=1000,gid=1000 /dev/sda1 /mnt/data

# 或者使用fmask和dmask设置权限
mount -o fmask=113,dmask=002 /dev/sda1 /mnt/data
```

## 面试题

**Q1：如何挂载一个ext4分区？**
```bash
mount -t ext4 /dev/sda1 /mnt/data
```

**Q2：如何将文件系统重新挂载为只读模式？**
```bash
mount -o remount,ro /mnt/data
```

**Q3：如何挂载ISO镜像文件？**
```bash
mount -o loop image.iso /mnt/cdrom
```

**Q4：卸载时提示设备忙怎么办？**
```bash
# 查找并终止占用进程
lsof /mnt/data
fuser -k /mnt/data
umount /mnt/data
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

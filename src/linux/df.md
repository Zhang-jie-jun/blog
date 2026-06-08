# df - 磁盘空间使用情况

## 概述

`df`命令用于查看文件系统的磁盘空间使用情况，包括总容量、已用空间、可用空间和挂载点。

## 基本语法

```bash
df [选项] [文件/目录]
```

## 常用选项

| 选项 | 说明 |
|------|------|
| `-a` | 显示所有文件系统（包括特殊文件系统） |
| `-h` | 以人类可读格式显示大小（KB、MB、GB） |
| `-H` | 以1000为单位显示（而非1024） |
| `-i` | 显示inode使用情况 |
| `-k` | 以KB为单位显示 |
| `-m` | 以MB为单位显示 |
| `-T` | 显示文件系统类型 |
| `-t` | 只显示指定类型的文件系统 |
| `-x` | 排除指定类型的文件系统 |

## 输出字段说明

| 字段 | 说明 |
|------|------|
| `Filesystem` | 文件系统设备名 |
| `Size` | 总容量 |
| `Used` | 已使用空间 |
| `Avail` | 可用空间 |
| `Use%` | 使用率 |
| `Mounted on` | 挂载点 |

## 使用场景

### 场景1：查看所有文件系统空间

```bash
# 人类可读格式
df -h

# 显示文件系统类型
df -hT

# 显示所有文件系统（包括特殊的）
df -a
```

### 场景2：查看指定目录所在的文件系统

```bash
df -h /home
df -h /
```

### 场景3：查看inode使用情况

```bash
# 查看inode使用
df -i

# 查看指定文件系统的inode
df -i /
```

### 场景4：过滤特定类型的文件系统

```bash
# 只显示ext4文件系统
df -h -t ext4

# 排除tmpfs
df -h -x tmpfs
```

## 问题分析原理

### 常见问题

**问题1：磁盘空间满了**

```bash
# 查找大文件
find / -type f -size +100M 2>/dev/null

# 查找占用空间最大的目录
du -sh /* 2>/dev/null | sort -rh

# 清理日志文件
rm -rf /var/log/*.log
```

**问题2：inode满了**

```bash
# 查看inode使用
df -i

# 查找大量小文件的目录
find / -type f | awk -F '/' '{print $3}' | sort | uniq -c | sort -nr

# 删除临时文件
find /tmp -type f -mtime +7 -delete
```

**问题3：根分区满了**

```bash
# 检查根分区
df -h /

# 查看各目录占用
du -sh /home /var /usr 2>/dev/null

# 清理旧的内核文件
apt autoremove --purge  # Debian/Ubuntu
yum remove kernel-*     # CentOS/RHEL
```

## 面试题

**Q1：如何查看磁盘空间使用情况？**
```bash
df -h
```

**Q2：如何查看inode使用情况？**
```bash
df -i
```

**Q3：如何只查看ext4文件系统？**
```bash
df -h -t ext4
```

**Q4：磁盘空间满了如何排查？**
```bash
# 查看各目录占用
du -sh /* | sort -rh
# 查找大文件
find / -type f -size +100M
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

# fstab - 文件系统挂载配置

## 概述

`/etc/fstab`文件用于定义系统启动时自动挂载的文件系统。它包含了所有需要挂载的设备、挂载点、文件系统类型和挂载选项。

## 文件格式

```
设备/UUID    挂载点    文件系统类型    挂载选项    dump    pass
```

### 字段说明

| 字段 | 说明 |
|------|------|
| **设备** | 可以是设备路径（/dev/sda1）或UUID |
| **挂载点** | 文件系统挂载的目录路径 |
| **文件系统类型** | ext4、xfs、ntfs、nfs等 |
| **挂载选项** | 用逗号分隔的挂载参数 |
| **dump** | 是否备份（0=不备份，1=备份） |
| **pass** | 开机自检顺序（0=不检查，1=优先检查，2=次要检查） |

## 常用挂载选项

| 选项 | 说明 |
|------|------|
| `defaults` | 默认选项（rw,suid,dev,exec,auto,nouser,async） |
| `noauto` | 不自动挂载（需要手动mount） |
| `auto` | 自动挂载 |
| `rw` | 读写模式 |
| `ro` | 只读模式 |
| `noexec` | 禁止执行二进制文件 |
| `nosuid` | 禁止setuid/setgid |
| `nodev` | 禁止访问设备文件 |
| `noatime` | 不更新访问时间 |
| `nodiratime` | 不更新目录访问时间 |
| `relatime` | 相对访问时间（平衡性能和功能） |
| `user` | 允许普通用户挂载 |
| `users` | 允许所有用户挂载 |
| `_netdev` | 网络设备（如NFS、CIFS） |

## 配置示例

### 示例1：挂载本地磁盘

```
# 使用设备路径
/dev/sda1    /mnt/data    ext4    defaults    0    2

# 使用UUID（推荐）
UUID=12345678-1234-5678-90ab-cdef01234567    /mnt/data    ext4    defaults    0    2
```

### 示例2：挂载swap分区

```
/dev/sda2    none    swap    sw    0    0

# 或使用UUID
UUID=87654321-4321-8765-abcd-ef0123456789    none    swap    sw    0    0
```

### 示例3：挂载NFS共享

```
server:/share    /mnt/nfs    nfs    defaults,_netdev    0    0
```

### 示例4：挂载ISO文件

```
/path/to/image.iso    /mnt/cdrom    iso9660    loop,ro    0    0
```

### 示例5：挂载Windows共享（CIFS）

```
//server/share    /mnt/smb    cifs    username=user,password=pass,_netdev    0    0
```

## 获取UUID

```bash
# 查看所有设备的UUID
blkid

# 查看指定设备的UUID
blkid /dev/sda1
```

## 测试fstab配置

```bash
# 检查fstab语法
mount -a

# 挂载单个条目
mount /mnt/data

# 验证挂载
df -h /mnt/data
```

## 问题分析原理

### 常见问题

**问题1：系统启动时挂载失败**

```bash
# 检查fstab语法错误
mount -a

# 检查UUID是否正确
blkid
```

**问题2：NFS挂载超时**

```bash
# 添加_netdev选项确保网络就绪后再挂载
# 在fstab中添加：
server:/share    /mnt/nfs    nfs    defaults,_netdev    0    0
```

**问题3：权限问题导致无法挂载**

```bash
# 确保挂载点目录存在且权限正确
mkdir -p /mnt/data
chmod 755 /mnt/data
```

**问题4：挂载后无法写入**

```bash
# 检查挂载选项是否为rw
mount | grep /mnt/data

# 如果是ro，重新挂载
mount -o remount,rw /mnt/data
```

## 面试题

**Q1：如何设置开机自动挂载磁盘？**
```bash
# 在/etc/fstab中添加条目
UUID=xxx /mnt/data ext4 defaults 0 2
```

**Q2：为什么推荐使用UUID而不是设备路径？**
- 设备路径可能会变化（如添加新磁盘后设备名改变）
- UUID是唯一的，不会因硬件变化而改变

**Q3：fstab中的dump和pass字段是什么意思？**
- `dump`：是否使用dump命令备份（0=不备份）
- `pass`：fsck检查顺序（0=不检查，1=根分区，2=其他分区）

**Q4：如何测试fstab配置是否正确？**
```bash
mount -a
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

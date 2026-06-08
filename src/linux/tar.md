# tar - 压缩解压工具

## 概述

`tar`是一个用于打包和压缩文件的工具，可以将多个文件或目录打包成一个文件，并支持多种压缩格式。

## 基本语法

```bash
tar [选项] [目标文件] [源文件/目录]
```

## 常用选项

| 选项 | 说明 |
|------|------|
| `-c` | 创建新的归档文件 |
| `-x` | 从归档文件中提取文件 |
| `-v` | 显示详细信息 |
| `-f` | 指定归档文件名 |
| `-z` | 使用gzip压缩 |
| `-j` | 使用bzip2压缩 |
| `-J` | 使用xz压缩 |
| `-t` | 列出归档文件内容 |
| `-p` | 保留文件权限 |
| `-C` | 指定解压目录 |

## 使用场景

### 场景1：创建归档文件

```bash
# 创建tar文件（不压缩）
tar -cvf archive.tar /path/to/files

# 创建gzip压缩文件
tar -czvf archive.tar.gz /path/to/files

# 创建bzip2压缩文件
tar -cjvf archive.tar.bz2 /path/to/files

# 创建xz压缩文件
tar -cJvf archive.tar.xz /path/to/files
```

### 场景2：查看归档内容

```bash
# 查看tar文件
tar -tvf archive.tar

# 查看压缩文件
tar -tzvf archive.tar.gz
```

### 场景3：解压归档文件

```bash
# 解压到当前目录
tar -xzvf archive.tar.gz

# 解压到指定目录
tar -xzvf archive.tar.gz -C /path/to/destination

# 解压特定文件
tar -xzvf archive.tar.gz file1.txt
```

### 场景4：添加文件到归档

```bash
# 添加文件到已存在的tar文件
tar -rvf archive.tar newfile.txt
```

### 场景5：压缩速度对比

| 压缩方式 | 速度 | 压缩率 | 扩展名 |
|---------|------|--------|--------|
| gzip | 快 | 中等 | .tar.gz |
| bzip2 | 较慢 | 较高 | .tar.bz2 |
| xz | 最慢 | 最高 | .tar.xz |

## 问题分析原理

### 常见问题

**问题1：解压时权限丢失**

```bash
# 使用-p选项保留权限
tar -xzvpf archive.tar.gz
```

**问题2：解压到指定目录**

```bash
# 创建目录并解压
mkdir -p /tmp/extract
tar -xzvf archive.tar.gz -C /tmp/extract
```

**问题3：分卷压缩**

```bash
# 创建分卷压缩
tar -czvf - /path/to/files | split -b 100M - archive.tar.gz.part

# 合并分卷
cat archive.tar.gz.part* > archive.tar.gz

# 解压
tar -xzvf archive.tar.gz
```

**问题4：增量备份**

```bash
# 创建增量归档
tar -czvf backup.tar.gz -g snapshot.snar /path/to/files

# 下次备份只备份变化的文件
tar -czvf backup2.tar.gz -g snapshot.snar /path/to/files
```

## 面试题

**Q1：如何创建gzip压缩文件？**
```bash
tar -czvf archive.tar.gz /path/to/files
```

**Q2：如何解压到指定目录？**
```bash
tar -xzvf archive.tar.gz -C /path/to/dir
```

**Q3：如何查看归档内容？**
```bash
tar -tvf archive.tar.gz
```

**Q4：gzip、bzip2、xz有什么区别？**
- `gzip`：速度最快，压缩率中等
- `bzip2`：速度较慢，压缩率较高
- `xz`：速度最慢，压缩率最高

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

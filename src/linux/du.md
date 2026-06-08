# du - 目录大小统计

## 概述

`du`命令用于统计目录或文件的磁盘使用空间，帮助用户了解哪些文件或目录占用了大量空间。

## 基本语法

```bash
du [选项] [文件/目录]
```

## 常用选项

| 选项 | 说明 |
|------|------|
| `-a` | 显示所有文件和目录的大小 |
| `-h` | 以人类可读格式显示（KB、MB、GB） |
| `-H` | 以1000为单位显示 |
| `-s` | 只显示总计大小 |
| `-k` | 以KB为单位显示 |
| `-m` | 以MB为单位显示 |
| `-c` | 显示总计大小 |
| `-d` | 指定显示深度 |
| `-x` | 只统计当前文件系统 |
| `--exclude` | 排除指定模式的文件 |

## 使用场景

### 场景1：查看目录大小

```bash
# 查看当前目录大小
du -sh

# 查看指定目录大小
du -sh /home

# 查看多个目录
du -sh /home /var /usr
```

### 场景2：查看目录下各子目录大小

```bash
# 查看一级子目录大小
du -h --max-depth=1

# 查看两级子目录
du -h -d 2 /var
```

### 场景3：查看文件大小

```bash
# 显示所有文件大小
du -ah /etc

# 只显示文件（不显示目录总计）
find /var -type f -exec du -h {} \;
```

### 场景4：按大小排序

```bash
# 按大小降序排列
du -sh /* | sort -rh

# 查找最大的前10个目录
du -a /home | sort -nr | head -10
```

### 场景5：排除特定文件

```bash
# 排除日志文件
du -sh --exclude="*.log" /var

# 排除多个模式
du -sh --exclude="*.log" --exclude="*.tmp" /var
```

## 问题分析原理

### 常见问题

**问题1：找出占用空间最大的目录**

```bash
# 查找根目录下最大的目录
du -sh /* | sort -rh | head -5

# 递归查找最大的目录
du -a / | sort -nr | head -20
```

**问题2：清理磁盘空间**

```bash
# 查找旧文件并删除
find /tmp -type f -mtime +30 -exec rm {} \;

# 查找大日志文件
find /var/log -type f -size +50M

# 清理缓存
rm -rf /var/cache/*
```

**问题3：统计特定类型文件大小**

```bash
# 统计所有.log文件大小
find /var -name "*.log" -exec du -ch {} + | grep total

# 统计所有图片文件
find /home -type f \( -name "*.jpg" -o -name "*.png" \) -exec du -ch {} +
```

## 面试题

**Q1：如何查看目录大小？**
```bash
du -sh /path/to/directory
```

**Q2：如何按大小排序显示目录？**
```bash
du -sh /* | sort -rh
```

**Q3：如何只显示一级子目录大小？**
```bash
du -h --max-depth=1
```

**Q4：如何统计特定类型文件的总大小？**
```bash
find /path -name "*.log" -exec du -ch {} + | grep total
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

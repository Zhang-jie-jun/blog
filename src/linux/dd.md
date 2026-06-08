# dd命令详解

## 概述

`dd`是一个强大的命令行工具，用于复制文件和转换数据。它可以用来测试磁盘性能、创建镜像文件、清空磁盘等。

---

## 一、基本语法

```bash
dd if=输入文件 of=输出文件 [选项]
```

### 常用选项

| 选项 | 说明 |
|------|------|
| `if` | 输入文件（input file） |
| `of` | 输出文件（output file） |
| `bs` | 块大小（block size） |
| `count` | 复制的块数 |
| `seek` | 输出文件开头跳过的块数 |
| `skip` | 输入文件开头跳过的块数 |
| `conv` | 转换选项（如sync、fsync） |
| `status` | 显示进度信息 |

### 块大小单位

| 单位 | 大小 |
|------|------|
| `b` | 512字节 |
| `k` | 1KB |
| `K` | 1024字节 |
| `m` | 1MB |
| `M` | 1024KB |
| `g` | 1GB |
| `G` | 1024MB |

---

## 二、磁盘性能测试

### 测试顺序写性能

```bash
# 测试大块顺序写（1GB数据，1MB块大小）
dd if=/dev/zero of=/tmp/test_write bs=1M count=1024 conv=fdatasync

# 测试小块顺序写（1GB数据，4KB块大小）
dd if=/dev/zero of=/tmp/test_write bs=4K count=262144 conv=fdatasync

# 测试带缓存写
dd if=/dev/zero of=/tmp/test_write bs=1M count=1024

# 测试直接IO写（绕过缓存）
dd if=/dev/zero of=/tmp/test_write bs=1M count=1024 oflag=direct
```

### 测试顺序读性能

```bash
# 测试大块顺序读
dd if=/tmp/test_write of=/dev/null bs=1M count=1024

# 测试小块顺序读
dd if=/tmp/test_write of=/dev/null bs=4K count=262144

# 测试直接IO读
dd if=/tmp/test_write of=/dev/null bs=1M count=1024 iflag=direct
```

### 测试随机IO性能

```bash
# 使用fio进行随机写测试
fio --name=randwrite --ioengine=sync --rw=randwrite --bs=4k --numjobs=1 --size=1G --iodepth=1 --runtime=60 --time_based

# 使用fio进行随机读测试
fio --name=randread --ioengine=sync --rw=randread --bs=4k --numjobs=1 --size=1G --iodepth=1 --runtime=60 --time_based
```

### 性能测试对比

| 测试类型 | 命令 | 适用场景 |
|----------|------|----------|
| 大块顺序写 | `dd if=/dev/zero of=test bs=1M count=1024 conv=fdatasync` | 大文件写入 |
| 小块顺序写 | `dd if=/dev/zero of=test bs=4K count=262144 conv=fdatasync` | 小文件写入 |
| 大块顺序读 | `dd if=test of=/dev/null bs=1M count=1024` | 大文件读取 |
| 小块顺序读 | `dd if=test of=/dev/null bs=4K count=262144` | 小文件读取 |
| 直接IO写 | `dd if=/dev/zero of=test bs=1M count=1024 oflag=direct` | 绕过缓存写 |

---

## 三、数据创建

### 创建大块数据文件

```bash
# 创建1GB文件（全零）
dd if=/dev/zero of=large_file bs=1M count=1024

# 创建10GB文件
dd if=/dev/zero of=large_file bs=1G count=10

# 创建指定大小的空文件（稀疏文件）
dd if=/dev/zero of=sparse_file bs=1G count=1 seek=100
```

### 创建小文件数据

```bash
# 创建多个小文件
for i in {1..1000}; do
    dd if=/dev/urandom of=file_$i bs=1K count=1 > /dev/null 2>&1
done

# 创建指定内容的文件
echo "Hello World" | dd of=hello.txt

# 创建随机内容文件
dd if=/dev/urandom of=random_data bs=1M count=10
```

### 创建测试数据

```bash
# 创建二进制测试数据
dd if=/dev/urandom of=binary_data bs=1M count=10

# 创建文本测试数据
yes "test line" | head -10000 | dd of=text_data.txt

# 创建重复模式数据
printf 'A%.0s' {1..1000} | dd of=pattern_data.txt
```

---

## 四、数据清理

### 清空磁盘数据

```bash
# 安全擦除磁盘（用零填充）
dd if=/dev/zero of=/dev/sdb bs=1M

# 安全擦除磁盘（用随机数据填充）
dd if=/dev/urandom of=/dev/sdb bs=1M

# 快速擦除（只写前1MB）
dd if=/dev/zero of=/dev/sdb bs=1M count=1

# 擦除分区
dd if=/dev/zero of=/dev/sdb1 bs=1M
```

### 清空文件内容

```bash
# 清空文件（保留文件）
dd if=/dev/null of=file.txt

# 替换文件内容
dd if=/dev/zero of=file.txt bs=1M count=1
```

### 安全删除文件

```bash
# 用零覆盖文件
dd if=/dev/zero of=secret_file bs=1M count=1 conv=notrunc

# 用随机数据覆盖
dd if=/dev/urandom of=secret_file bs=1M count=1 conv=notrunc

# 多次覆盖（更安全）
shred -vfz -n 3 secret_file
```

---

## 五、磁盘镜像

### 创建磁盘镜像

```bash
# 创建整个磁盘的镜像
dd if=/dev/sda of=/backup/sda.img bs=1M

# 创建分区镜像
dd if=/dev/sda1 of=/backup/sda1.img bs=1M

# 压缩镜像
dd if=/dev/sda bs=1M | gzip > /backup/sda.img.gz

# 备份MBR
dd if=/dev/sda of=/backup/mbr.img bs=512 count=1
```

### 恢复磁盘镜像

```bash
# 恢复整个磁盘
dd if=/backup/sda.img of=/dev/sda bs=1M

# 恢复分区
dd if=/backup/sda1.img of=/dev/sda1 bs=1M

# 从压缩镜像恢复
gunzip -c /backup/sda.img.gz | dd of=/dev/sda bs=1M
```

### 克隆磁盘

```bash
# 直接克隆到另一个磁盘
dd if=/dev/sda of=/dev/sdb bs=1M

# 克隆到远程磁盘
dd if=/dev/sda bs=1M | ssh user@remote 'dd of=/dev/sdb bs=1M'
```

---

## 六、数据转换

### 格式转换

```bash
# 转换大小写
dd if=input.txt of=output.txt conv=ucase

# 转换换行符（DOS→Unix）
dd if=dos.txt of=unix.txt conv=unix

# 转换换行符（Unix→DOS）
dd if=unix.txt of=dos.txt conv=dos

# ASCII→EBCDIC转换
dd if=ascii.txt of=ebcdic.txt conv=ebcdic
```

### 数据修复

```bash
# 跳过错误继续复制
dd if=/dev/sda of=/backup/sda.img bs=1M conv=noerror,sync

# 只复制有效数据块
dd if=/dev/sda of=/backup/sda.img bs=1M conv=noerror
```

---

## 七、实用技巧

### 查看dd进度

```bash
# 使用status选项（GNU dd 8.24+）
dd if=/dev/zero of=/tmp/test bs=1M count=1024 status=progress

# 使用pv命令
pv /dev/zero | dd of=/tmp/test bs=1M count=1024

# 使用watch查看进度
watch -n 1 'ls -la /tmp/test'
```

### 限制速度

```bash
# 限制写入速度为1MB/s
dd if=/dev/zero of=/tmp/test bs=1M count=1024 | pv -qL 1M

# 使用ddrescue限制速度
ddrescue --max-rate=1M /dev/zero /tmp/test
```

### 并行复制

```bash
# 使用pigz并行压缩
dd if=/dev/sda bs=1M | pigz -9 > /backup/sda.img.gz

# 使用pbzip2并行压缩
dd if=/dev/sda bs=1M | pbzip2 > /backup/sda.img.bz2
```

---

## 八、面试题

**Q1：如何用dd测试磁盘顺序写性能？**
```bash
dd if=/dev/zero of=/tmp/test bs=1M count=1024 conv=fdatasync
```

**Q2：如何创建一个1GB的文件？**
```bash
dd if=/dev/zero of=1gb_file bs=1M count=1024
```

**Q3：如何安全擦除磁盘数据？**
```bash
dd if=/dev/urandom of=/dev/sdb bs=1M
```

**Q4：如何创建磁盘镜像？**
```bash
dd if=/dev/sda of=/backup/sda.img bs=1M
```

**Q5：dd中的conv=fdatasync是什么作用？**
- 强制将数据写入磁盘后再返回
- 确保测试结果反映真实磁盘性能，而不是缓存性能

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

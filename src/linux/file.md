# 文件操作命令详解

## 概述

文件操作是Linux系统中最常用的操作之一，包括文件的创建、复制、移动、删除、查找等。本文介绍常用的文件操作命令。

---

## 一、文件创建与删除

### 1. touch

```bash
# 创建空文件
touch file.txt

# 创建多个文件
touch file1.txt file2.txt file3.txt

# 更新文件时间戳
touch -d "2024-01-01" file.txt
```

### 2. rm

```bash
# 删除文件
rm file.txt

# 删除目录
rm -r directory

# 强制删除（不提示）
rm -f file.txt

# 删除目录（强制）
rm -rf directory

# 删除所有txt文件
rm *.txt
```

### 3. mkdir/rmdir

```bash
# 创建目录
mkdir directory

# 创建多级目录
mkdir -p /path/to/directory

# 删除空目录
rmdir directory

# 删除目录树
rm -rf directory
```

---

## 二、文件复制与移动

### 1. cp

```bash
# 复制文件
cp source.txt dest.txt

# 复制目录
cp -r source_dir dest_dir

# 保留属性
cp -a source.txt dest.txt

# 交互式复制
cp -i source.txt dest.txt

# 强制复制（覆盖不提示）
cp -f source.txt dest.txt
```

### 2. mv

```bash
# 移动文件
mv source.txt dest.txt

# 移动目录
mv source_dir dest_dir

# 重命名文件
mv oldname.txt newname.txt

# 移动到上级目录
mv file.txt ..

# 交互式移动
mv -i source.txt dest.txt
```

---

## 三、文件查看

### 1. cat

```bash
# 查看文件内容
cat file.txt

# 显示行号
cat -n file.txt

# 显示不可见字符
cat -A file.txt

# 合并文件
cat file1.txt file2.txt > combined.txt
```

### 2. less/more

```bash
# 分页查看（向前翻页）
less file.txt

# 搜索
/keyword

# 退出
q
```

### 3. head/tail

```bash
# 查看文件头部（默认前10行）
head file.txt

# 查看前20行
head -n 20 file.txt

# 查看文件尾部（默认后10行）
tail file.txt

# 实时监控日志
tail -f /var/log/syslog

# 查看最后20行
tail -n 20 file.txt
```

---

## 四、文件查找

### 1. find

```bash
# 在当前目录查找文件
find . -name "*.txt"

# 查找所有txt文件
find / -name "*.txt" 2>/dev/null

# 按类型查找
find . -type f -name "*.txt"

# 按大小查找（大于100MB）
find . -size +100M

# 按时间查找（最近7天）
find . -mtime -7

# 查找并删除
find . -name "*.log" -delete

# 查找并执行命令
find . -name "*.txt" -exec ls -la {} \;
```

### 2. locate

```bash
# 快速查找文件
locate file.txt

# 更新数据库
updatedb
```

### 3. which

```bash
# 查找命令位置
which ls

# 查找所有匹配
which -a python
```

---

## 五、文件比较

### 1. diff

```bash
# 比较两个文件
diff file1.txt file2.txt

# 并排显示
diff -y file1.txt file2.txt

# 统一格式
diff -u file1.txt file2.txt

# 递归比较目录
diff -r dir1 dir2
```

### 2. cmp

```bash
# 比较两个文件
cmp file1.txt file2.txt

# 显示差异位置
cmp -l file1.txt file2.txt
```

---

## 六、文件链接

### 1. ln

```bash
# 创建硬链接
ln file.txt hardlink.txt

# 创建软链接（符号链接）
ln -s file.txt softlink.txt

# 创建目录软链接
ln -s /path/to/directory link_name
```

---

## 七、文件权限

### 1. chmod

```bash
# 设置权限（数字模式）
chmod 755 file.txt

# 设置权限（符号模式）
chmod u+rwx,g+rx,o+r file.txt

# 递归修改
chmod -R 755 directory
```

### 2. chown

```bash
# 修改所有者
chown user:group file.txt

# 递归修改
chown -R user:group directory
```

---

## 八、文件压缩与解压

### 1. tar

```bash
# 打包
tar -cvf archive.tar directory

# 打包并压缩（gzip）
tar -czvf archive.tar.gz directory

# 打包并压缩（bzip2）
tar -cjvf archive.tar.bz2 directory

# 解压
tar -xvf archive.tar

# 解压gz文件
tar -xzvf archive.tar.gz

# 解压到指定目录
tar -xzvf archive.tar.gz -C /path/to/directory
```

### 2. zip/unzip

```bash
# 压缩
zip archive.zip file1.txt file2.txt

# 压缩目录
zip -r archive.zip directory

# 解压
unzip archive.zip

# 解压到指定目录
unzip archive.zip -d /path/to/directory
```

---

## 九、实战案例

### 案例1：批量重命名

```bash
# 将所有txt文件重命名为bak文件
for file in *.txt; do
    mv "$file" "${file%.txt}.bak"
done
```

### 案例2：查找大文件

```bash
# 查找大于1GB的文件
find / -size +1G 2>/dev/null | head -10
```

### 案例3：备份文件

```bash
# 创建备份
cp -a /data /data_backup_$(date +%Y%m%d)
```

---

## 十、面试题

**Q1：如何创建目录？**
```bash
mkdir directory
mkdir -p /path/to/directory
```

**Q2：如何复制文件？**
```bash
cp source.txt dest.txt
cp -r source_dir dest_dir
```

**Q3：如何查找文件？**
```bash
find . -name "*.txt"
locate file.txt
```

**Q4：如何查看文件内容？**
```bash
cat file.txt
less file.txt
head file.txt
tail file.txt
```

**Q5：如何创建软链接？**
```bash
ln -s source link_name
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

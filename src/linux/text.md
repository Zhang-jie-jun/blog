# 文本处理命令详解

## 概述

文本处理是Linux系统中最常用的操作之一，包括文本搜索、替换、排序、统计等。本文介绍常用的文本处理命令。

---

## 一、文本搜索

### 1. grep

```bash
# 搜索文本
grep "pattern" file.txt

# 忽略大小写
grep -i "pattern" file.txt

# 显示行号
grep -n "pattern" file.txt

# 显示匹配前后的行
grep -A 2 -B 2 "pattern" file.txt

# 递归搜索目录
grep -r "pattern" directory

# 反向匹配
grep -v "pattern" file.txt

# 正则表达式
grep -E "^[a-z]+" file.txt

# 统计匹配次数
grep -c "pattern" file.txt
```

### 2. egrep

```bash
# 扩展正则表达式
egrep "pattern1|pattern2" file.txt

# 等同于 grep -E
grep -E "pattern1|pattern2" file.txt
```

---

## 二、文本替换

### 1. sed

```bash
# 替换文本
sed 's/old/new/' file.txt

# 全局替换
sed 's/old/new/g' file.txt

# 替换并保存
sed -i 's/old/new/g' file.txt

# 删除空行
sed '/^$/d' file.txt

# 删除指定行
sed '5d' file.txt

# 在指定行后插入
sed '5a new line' file.txt

# 在指定行前插入
sed '5i new line' file.txt

# 正则表达式替换
sed 's/^[0-9]+//' file.txt
```

### 2. awk

```bash
# 打印指定字段
awk '{print $1}' file.txt

# 指定分隔符
awk -F ',' '{print $1}' file.txt

# 条件过滤
awk '$3 > 100' file.txt

# 计算总和
awk '{sum += $1} END {print sum}' file.txt

# 统计行数
awk 'END {print NR}' file.txt

# 格式化输出
awk '{printf "%-10s %d\n", $1, $2}' file.txt
```

---

## 三、文本排序

### 1. sort

```bash
# 排序
sort file.txt

# 反向排序
sort -r file.txt

# 按数值排序
sort -n file.txt

# 按第二列排序
sort -k 2 file.txt

# 去除重复行
sort -u file.txt

# 按分隔符排序
sort -t ',' -k 2 file.txt
```

### 2. uniq

```bash
# 去除重复行（需先排序）
sort file.txt | uniq

# 显示重复次数
sort file.txt | uniq -c

# 只显示重复行
sort file.txt | uniq -d
```

---

## 四、文本统计

### 1. wc

```bash
# 统计行数、单词数、字节数
wc file.txt

# 只统计行数
wc -l file.txt

# 只统计单词数
wc -w file.txt

# 只统计字节数
wc -c file.txt

# 统计字符数
wc -m file.txt
```

### 2. cut

```bash
# 提取指定字段
cut -d ',' -f 1,3 file.txt

# 提取指定字符范围
cut -c 1-10 file.txt

# 去除指定字段
cut -d ',' --complement -f 2 file.txt
```

### 3. paste

```bash
# 合并文件（列合并）
paste file1.txt file2.txt

# 指定分隔符
paste -d ',' file1.txt file2.txt
```

---

## 五、文本转换

### 1. tr

```bash
# 字符替换
tr 'a-z' 'A-Z' < file.txt

# 删除指定字符
tr -d '0-9' < file.txt

# 压缩重复字符
tr -s ' ' < file.txt
```

### 2. split

```bash
# 按行数分割
split -l 100 file.txt

# 按大小分割
split -b 10M file.txt

# 指定输出前缀
split -l 100 file.txt output_
```

---

## 六、文本连接

### 1. join

```bash
# 按字段连接文件
join file1.txt file2.txt

# 指定连接字段
join -1 1 -2 2 file1.txt file2.txt

# 指定分隔符
join -t ',' file1.txt file2.txt
```

---

## 七、实战案例

### 案例1：统计日志中访问次数最多的IP

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -10
```

### 案例2：提取CSV文件中的特定列

```bash
cut -d ',' -f 1,3,5 data.csv > extracted.csv
```

### 案例3：批量替换文件中的文本

```bash
sed -i 's/old_domain.com/new_domain.com/g' *.html
```

### 案例4：统计文件中每个单词出现的次数

```bash
tr ' ' '\n' < file.txt | sort | uniq -c | sort -nr
```

---

## 八、面试题

**Q1：如何在文件中搜索文本？**
```bash
grep "pattern" file.txt
```

**Q2：如何替换文件中的文本？**
```bash
sed -i 's/old/new/g' file.txt
```

**Q3：如何提取文件中的指定字段？**
```bash
awk '{print $1}' file.txt
cut -d ',' -f 1 file.txt
```

**Q4：如何统计文件的行数？**
```bash
wc -l file.txt
awk 'END {print NR}' file.txt
```

**Q5：如何去除文件中的重复行？**
```bash
sort file.txt | uniq
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

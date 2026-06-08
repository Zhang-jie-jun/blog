# Linux三剑客：grep、sed、awk

## 概述

Linux三剑客是文本处理领域最强大的三个命令行工具：
- **grep**：文本搜索工具，用于在文件中查找匹配的字符串
- **sed**：流编辑器，用于对文本进行替换、删除、插入等操作
- **awk**：文本处理语言，用于数据提取和报告生成

---

## 一、grep - 文本搜索利器

### 1. 基本语法

```bash
grep [选项] "搜索模式" 文件名
```

### 2. 常用选项

| 选项 | 说明 | 示例 |
|------|------|------|
| `-i` | 忽略大小写 | `grep -i "hello" file.txt` |
| `-v` | 反向匹配（不包含） | `grep -v "error" log.txt` |
| `-r` | 递归搜索目录 | `grep -r "TODO" ./src` |
| `-l` | 只显示匹配的文件名 | `grep -l "pattern" *.txt` |
| `-n` | 显示行号 | `grep -n "error" log.txt` |
| `-c` | 只显示匹配行数 | `grep -c "success" result.txt` |
| `-A n` | 显示匹配行及后面n行 | `grep -A 3 "error" log.txt` |
| `-B n` | 显示匹配行及前面n行 | `grep -B 2 "error" log.txt` |
| `-E` | 使用扩展正则表达式 | `grep -E "pattern1|pattern2" file.txt` |

### 3. 正则表达式

```bash
# 匹配以abc开头的行
grep "^abc" file.txt

# 匹配以xyz结尾的行
grep "xyz$" file.txt

# 匹配包含数字的行
grep "[0-9]" file.txt

# 匹配任意字符
grep "a.c" file.txt

# 匹配0个或多个字符
grep "a*b" file.txt

# 匹配1个或多个字符
grep -E "a+b" file.txt

# 匹配0个或1个字符
grep -E "colou?r" file.txt

# 或匹配
grep -E "cat|dog" file.txt
```

### 4. 使用场景

**场景1：查找日志中的错误信息**
```bash
# 查找所有包含ERROR的行
grep "ERROR" app.log

# 查找ERROR并显示上下文
grep -A 2 -B 2 "ERROR" app.log
```

**场景2：查找包含特定IP的访问日志**
```bash
grep "192.168.1.100" access.log
```

**场景3：排除注释行和空行**
```bash
grep -v "^#" config.conf | grep -v "^$"
```

### 5. 面试题

**Q1：如何在当前目录及其子目录中查找包含"TODO"的文件？**
```bash
grep -r "TODO" .
```

**Q2：如何统计日志文件中ERROR出现的次数？**
```bash
grep -c "ERROR" app.log
```

**Q3：如何显示匹配行的行号？**
```bash
grep -n "pattern" file.txt
```

---

## 二、sed - 流编辑器

### 1. 基本语法

```bash
sed [选项] '命令' 文件名
```

### 2. 常用选项

| 选项 | 说明 |
|------|------|
| `-i` | 直接修改原文件（原地编辑） |
| `-e` | 执行多个编辑命令 |
| `-n` | 静默模式，只显示匹配的行 |

### 3. 常用命令

```bash
# 替换操作（s表示substitute）
sed 's/旧字符串/新字符串/' file.txt

# 全局替换（g表示global）
sed 's/old/new/g' file.txt

# 删除指定行
sed '3d' file.txt          # 删除第3行
sed '1,5d' file.txt        # 删除第1-5行
sed '/pattern/d' file.txt  # 删除包含pattern的行

# 插入操作
sed '3i\new line' file.txt     # 在第3行前插入
sed '3a\new line' file.txt     # 在第3行后追加

# 打印指定行
sed -n '5,10p' file.txt    # 打印第5-10行
sed -n '/pattern/p' file.txt  # 打印包含pattern的行
```

### 4. 使用场景

**场景1：批量替换配置文件中的IP地址**
```bash
sed -i 's/192.168.1.1/10.0.0.1/g' config.conf
```

**场景2：删除日志文件中的空行**
```bash
sed '/^$/d' app.log > cleaned.log
```

**场景3：在文件末尾添加内容**
```bash
sed -i '$a\export PATH=$PATH:/opt/bin' ~/.bashrc
```

**场景4：提取特定行**
```bash
# 提取第10行
sed -n '10p' file.txt

# 提取包含"error"的行
sed -n '/error/p' app.log
```

### 5. 面试题

**Q1：如何将文件中所有的"foo"替换为"bar"？**
```bash
sed 's/foo/bar/g' file.txt
```

**Q2：如何删除文件中的第5行？**
```bash
sed '5d' file.txt
```

**Q3：如何在原文件中进行替换操作？**
```bash
sed -i 's/old/new/g' file.txt
```

---

## 三、awk - 文本处理语言

### 1. 基本语法

```bash
awk [选项] 'pattern {action}' 文件名
```

### 2. 常用内置变量

| 变量 | 说明 |
|------|------|
| `$0` | 整行内容 |
| `$1-$n` | 第1到第n个字段 |
| `NF` | 字段总数 |
| `NR` | 当前行号 |
| `FS` | 字段分隔符（默认空格） |
| `OFS` | 输出字段分隔符 |

### 3. 常用操作

```bash
# 打印指定字段
awk '{print $1, $3}' file.txt  # 打印第1和第3字段

# 指定字段分隔符
awk -F ':' '{print $1}' /etc/passwd  # 使用冒号分隔

# 条件过滤
awk '$3 > 100' file.txt  # 第3字段大于100的行

# 计算操作
awk '{sum += $2} END {print sum}' file.txt  # 求和

# 格式化输出
awk '{printf "%-10s %5d\n", $1, $2}' file.txt
```

### 4. 使用场景

**场景1：统计日志中各IP的访问次数**
```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr
```

**场景2：计算文件中数字的总和**
```bash
awk '{sum += $1} END {print "Total:", sum}' numbers.txt
```

**场景3：格式化输出/etc/passwd**
```bash
awk -F ':' '{print "用户名:", $1, "UID:", $3}' /etc/passwd
```

**场景4：过滤特定条件的行**
```bash
# 过滤第2字段大于1000的行
awk '$2 > 1000' data.txt
```

### 5. 面试题

**Q1：如何提取文件的第一列？**
```bash
awk '{print $1}' file.txt
```

**Q2：如何统计文件的行数？**
```bash
awk 'END {print NR}' file.txt
```

**Q3：如何计算文件中所有数字的平均值？**
```bash
awk '{sum += $1; count++} END {print sum/count}' numbers.txt
```

---

## 综合实战

### 实战1：分析Nginx访问日志

```bash
# 统计访问最多的前10个IP
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -10

# 统计各状态码的数量
awk '{print $9}' access.log | sort | uniq -c

# 查找访问时间超过5秒的请求
awk '$NF > 5.0' access.log
```

### 实战2：处理CSV数据

```bash
# 提取CSV文件的第2列和第5列
awk -F ',' '{print $2, $5}' data.csv

# 计算第3列的总和和平均值
awk -F ',' '{sum += $3; count++} END {print "Sum:", sum, "Avg:", sum/count}' data.csv
```

### 实战3：批量重命名文件

```bash
# 将所有.txt文件重命名为.bak
ls *.txt | sed 's/\(.*\)\.txt/mv \1.txt \1.bak/' | bash
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

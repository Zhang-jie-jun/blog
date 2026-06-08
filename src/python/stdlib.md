# Python 标准库

## 标准库简介

Python 标准库包含了大量实用的模块，覆盖文件操作、网络通信、数据处理等多个领域。

---

## 1. 文件与目录操作

### os 模块

```python
import os

# 获取当前目录
print(os.getcwd())

# 创建目录
os.makedirs("new_dir/subdir", exist_ok=True)

# 列出目录内容
print(os.listdir("."))

# 文件路径操作
path = os.path.join("dir", "file.txt")
print(os.path.abspath(path))
print(os.path.exists(path))
```

### pathlib 模块

```python
from pathlib import Path

# 创建路径对象
p = Path("example.txt")

# 检查路径类型
print(p.exists())
print(p.is_file())
print(p.is_dir())

# 获取文件信息
print(p.stat())

# 遍历目录
for file in Path(".").glob("*.txt"):
    print(file)
```

---

## 2. 数据处理

### collections 模块

```python
from collections import defaultdict, Counter, deque

# defaultdict：默认值字典
d = defaultdict(list)
d['key'].append(1)
d['key'].append(2)
print(d['key'])  # [1, 2]

# Counter：计数
c = Counter("hello world")
print(c)  # Counter({'l': 3, 'o': 2, 'h': 1, 'e': 1, ' ': 1, 'w': 1, 'r': 1, 'd': 1})

# deque：双端队列
q = deque([1, 2, 3])
q.append(4)
q.appendleft(0)
print(q)  # deque([0, 1, 2, 3, 4])
```

### itertools 模块

```python
import itertools

# 无限迭代器
for i in itertools.count(1, 2):
    if i > 10:
        break
    print(i)  # 1, 3, 5, 7, 9

# 组合工具
permutations = itertools.permutations([1, 2, 3], 2)
for p in permutations:
    print(p)  # (1,2), (1,3), (2,1), (2,3), (3,1), (3,2)
```

---

## 3. 网络通信

### socket 模块

```python
import socket

# 创建 TCP 服务器
server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.bind(('localhost', 12345))
server.listen(5)

# 创建 TCP 客户端
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect(('localhost', 12345))
client.send(b"Hello, Server!")
data = client.recv(1024)
print(data.decode())
client.close()
```

### urllib 模块

```python
from urllib import request, parse

# 发送 HTTP 请求
response = request.urlopen("https://www.example.com")
html = response.read().decode()

# 带参数的请求
params = {'key1': 'value1', 'key2': 'value2'}
query_string = parse.urlencode(params)
url = f"https://api.example.com/data?{query_string}"
```

---

## 4. 日期与时间

### datetime 模块

```python
from datetime import datetime, timedelta

# 获取当前时间
now = datetime.now()
print(now)

# 格式化输出
print(now.strftime("%Y-%m-%d %H:%M:%S"))

# 时间运算
tomorrow = now + timedelta(days=1)
print(tomorrow)

# 时间差
diff = tomorrow - now
print(diff.days)
```

### time 模块

```python
import time

# 获取时间戳
timestamp = time.time()
print(timestamp)

# 休眠
time.sleep(1)

# 格式化时间
print(time.strftime("%Y-%m-%d", time.localtime()))
```

---

## 5. 正则表达式

### re 模块

```python
import re

# 匹配
pattern = r"\d+"
text = "There are 123 apples and 456 oranges"
matches = re.findall(pattern, text)
print(matches)  # ['123', '456']

# 替换
result = re.sub(r"\d+", "NUMBER", text)
print(result)  # "There are NUMBER apples and NUMBER oranges"

# 分组
pattern = r"(\w+)@(\w+)\.(\w+)"
email = "user@example.com"
match = re.match(pattern, email)
print(match.group(1))  # user
print(match.group(2))  # example
```

---

## 6. 数据序列化

### json 模块

```python
import json

# 序列化
data = {"name": "Alice", "age": 30}
json_str = json.dumps(data)
print(json_str)

# 反序列化
data = json.loads(json_str)
print(data["name"])

# 文件操作
with open("data.json", "w") as f:
    json.dump(data, f)

with open("data.json", "r") as f:
    data = json.load(f)
```

### pickle 模块

```python
import pickle

# 序列化任意对象
data = {"key": "value", "list": [1, 2, 3]}
pickled = pickle.dumps(data)

# 反序列化
unpickled = pickle.loads(pickled)
print(unpickled)
```

---

## 常见面试题

### 1. os 和 pathlib 的区别？

**答案：**
- `os`：传统的函数式接口
- `pathlib`：面向对象的路径操作，更直观

### 2. 如何发送 HTTP 请求？

**答案：**
使用 `urllib` 模块或第三方库 `requests`。

### 3. 什么是正则表达式？

**答案：**
正则表达式是一种文本匹配模式，用于搜索、替换和验证字符串。

### 4. json 和 pickle 的区别？

**答案：**
| 特性 | json | pickle |
|------|------|--------|
| 可读性 | 高 | 低 |
| 支持类型 | 基本类型 | 任意Python对象 |
| 跨语言 | 是 | 否 |

### 5. collections 模块有哪些常用数据结构？

**答案：**
- `defaultdict`：默认值字典
- `Counter`：计数器
- `deque`：双端队列
- `namedtuple`：命名元组

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
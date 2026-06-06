# Python 第三方库

## 第三方库简介

Python 拥有庞大的第三方库生态，涵盖数据科学、Web 开发、机器学习等多个领域。

---

## 1. 数据科学

### NumPy

```python
import numpy as np

# 创建数组
arr = np.array([1, 2, 3, 4, 5])
print(arr)

# 数组运算
print(arr + 2)      # [3 4 5 6 7]
print(arr * 2)      # [ 2  4  6  8 10]
print(arr.mean())   # 3.0

# 矩阵运算
matrix = np.array([[1, 2], [3, 4]])
print(matrix @ matrix)  # 矩阵乘法
```

### Pandas

```python
import pandas as pd

# 创建 DataFrame
data = {
    'Name': ['Alice', 'Bob', 'Charlie'],
    'Age': [25, 30, 35],
    'City': ['New York', 'London', 'Paris']
}
df = pd.DataFrame(data)
print(df)

# 数据操作
print(df['Name'])          # 选择列
print(df.loc[0])           # 选择行
print(df[df['Age'] > 30])  # 过滤

# 数据统计
print(df.describe())
```

### Matplotlib

```python
import matplotlib.pyplot as plt

# 折线图
x = [1, 2, 3, 4, 5]
y = [1, 4, 9, 16, 25]
plt.plot(x, y)
plt.xlabel('X')
plt.ylabel('Y')
plt.title('Square Function')
plt.show()

# 柱状图
categories = ['A', 'B', 'C', 'D']
values = [3, 7, 2, 5]
plt.bar(categories, values)
plt.show()
```

---

## 2. 机器学习

### Scikit-learn

```python
from sklearn import datasets
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_squared_error

# 加载数据集
boston = datasets.load_boston()
X = boston.data
y = boston.target

# 划分数据集
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

# 训练模型
model = LinearRegression()
model.fit(X_train, y_train)

# 预测
y_pred = model.predict(X_test)
print(f"MSE: {mean_squared_error(y_test, y_pred)}")
```

### TensorFlow

```python
import tensorflow as tf

# 创建简单神经网络
model = tf.keras.Sequential([
    tf.keras.layers.Dense(64, activation='relu', input_shape=(784,)),
    tf.keras.layers.Dense(10, activation='softmax')
])

model.compile(optimizer='adam',
              loss='sparse_categorical_crossentropy',
              metrics=['accuracy'])

# 训练模型
model.fit(x_train, y_train, epochs=10)
```

---

## 3. Web 开发

### Flask

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello, World!"

@app.route('/api/data', methods=['GET', 'POST'])
def data():
    if request.method == 'POST':
        data = request.get_json()
        return jsonify({'status': 'success', 'data': data})
    return jsonify({'message': 'GET request'})

if __name__ == '__main__':
    app.run(debug=True)
```

### Django

```python
# views.py
from django.http import JsonResponse
from django.views import View

class DataView(View):
    def get(self, request):
        return JsonResponse({'message': 'GET request'})
    
    def post(self, request):
        data = request.POST
        return JsonResponse({'status': 'success'})

# urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('data/', views.DataView.as_view()),
]
```

---

## 4. 异步编程

### requests（同步）

```python
import requests

response = requests.get('https://api.example.com/data')
print(response.json())

payload = {'key': 'value'}
response = requests.post('https://api.example.com/data', json=payload)
```

### aiohttp（异步）

```python
import aiohttp
import asyncio

async def fetch(session, url):
    async with session.get(url) as response:
        return await response.json()

async def main():
    async with aiohttp.ClientSession() as session:
        data = await fetch(session, 'https://api.example.com/data')
        print(data)

asyncio.run(main())
```

---

## 5. 数据库

### SQLAlchemy

```python
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

Base = declarative_base()

class User(Base):
    __tablename__ = 'users'
    id = Column(Integer, primary_key=True)
    name = Column(String)
    age = Column(Integer)

engine = create_engine('sqlite:///example.db')
Base.metadata.create_all(engine)

Session = sessionmaker(bind=engine)
session = Session()

# 添加数据
user = User(name='Alice', age=30)
session.add(user)
session.commit()

# 查询数据
users = session.query(User).all()
for user in users:
    print(user.name, user.age)
```

---

## 常见面试题

### 1. NumPy 和 Pandas 的区别？

**答案：**
- **NumPy**：处理数值数组，擅长数学运算
- **Pandas**：处理表格数据，擅长数据清洗和分析

### 2. Flask 和 Django 的区别？

**答案：**
| 特性 | Flask | Django |
|------|-------|--------|
| 大小 | 轻量级 | 重量级 |
| 功能 | 灵活 | 全功能 |
| 适合场景 | 小型项目 | 大型项目 |

### 3. 什么是机器学习？

**答案：**
机器学习是让计算机从数据中学习模式，无需明确编程。

### 4. 如何发送 HTTP 请求？

**答案：**
使用 `requests` 库（同步）或 `aiohttp` 库（异步）。

### 5. SQLAlchemy 是什么？

**答案：**
SQLAlchemy 是 Python 的 ORM（对象关系映射）库，用于数据库操作。

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
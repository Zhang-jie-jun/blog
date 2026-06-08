# Postman使用方法

## Postman是什么？
&emsp;&emsp;Postman是一款流行的API测试工具，用于发送HTTP请求、测试API接口、自动化测试等。

## 基本操作

### 创建请求

1. 打开Postman
2. 点击「New」→「Request」
3. 输入请求名称和描述
4. 选择请求方法（GET、POST、PUT、DELETE等）
5. 输入URL

### 发送请求

```bash
# GET请求示例
GET https://api.example.com/users

# POST请求示例
POST https://api.example.com/users
Content-Type: application/json

{
"name": "John",
"email": "john@example.com"
}
```

## 请求类型

### GET请求
- 用于获取资源
- 参数通过URL传递

### POST请求
- 用于创建资源
- 参数通过请求体传递
- 常用Content-Type:
    - application/json
    - application/x-www-form-urlencoded
    - multipart/form-data

### PUT请求
- 用于更新资源
- 替换整个资源

### PATCH请求
- 用于部分更新资源

### DELETE请求
- 用于删除资源

## 请求参数

### URL参数
- Params标签页
- 键值对形式

### 请求头
- Headers标签页
- 常用Headers:
    - Content-Type
    - Authorization
    - Accept

### 请求体
- Body标签页
- 支持格式:
    - raw (JSON, XML, Text)
    - form-data
    - x-www-form-urlencoded
    - binary

## 响应处理

### 查看响应
- Body标签页: 响应内容
- Headers标签页: 响应头
- Status: HTTP状态码
- Time: 响应时间
- Size: 响应大小

### 响应状态码
- 200 OK: 请求成功
- 201 Created: 资源创建成功
- 400 Bad Request: 请求参数错误
- 401 Unauthorized: 未授权
- 403 Forbidden: 禁止访问
- 404 Not Found: 资源未找到
- 500 Internal Server Error: 服务器错误

## 环境变量

### 创建环境
1. 点击右上角环境选择器
2. 点击「Add」
3. 输入环境名称
4. 添加变量（key-value）

### 使用环境变量
```bash
# 在URL中使用
https://{{base_url}}/api/users

# 在请求头中使用
Authorization: Bearer {{access_token}}

# 在请求体中使用
{
"api_key": "{{api_key}}"
}
```

## 集合（Collection）

### 创建集合
1. 点击「New」→「Collection」
2. 输入名称和描述
3. 添加请求到集合

### 运行集合
1. 选择集合
2. 点击「Run」
3. 配置运行选项
4. 查看结果

## 自动化测试

### 添加测试脚本
```javascript
// 状态码测试
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// 响应时间测试
pm.test("Response time is less than 200ms", function () {
    pm.expect(pm.response.responseTime).to.be.below(200);
});

// JSON响应测试
pm.test("Response has correct data", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.name).to.eql("John");
});
```

### 前置脚本
```javascript
// 设置环境变量
pm.environment.set("timestamp", Date.now());

// 生成随机ID
pm.environment.set("random_id", Math.random().toString(36).substring(2, 9));
```

## Mock Server

### 创建Mock
1. 选择集合
2. 点击「Mock」
3. 配置Mock响应
4. 获取Mock URL

## 常用快捷键

```bash
Ctrl+N          # 新建请求
Ctrl+S          # 保存
Ctrl+Enter      # 发送请求
Ctrl+D          # 复制请求
Ctrl+Shift+D    # 删除请求
Ctrl+F          # 搜索
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

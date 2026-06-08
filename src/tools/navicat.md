# Navicat使用方法

## Navicat是什么？
&emsp;&emsp;Navicat是一款强大的数据库管理工具，支持多种数据库（MySQL、PostgreSQL、SQLite、SQL Server等）。

## 连接数据库

### MySQL连接

1. 打开Navicat
2. 点击「连接」→「MySQL」
3. 输入连接信息:
    - 连接名: 自定义名称
    - 主机名: localhost或IP地址
    - 端口: 3306（默认）
    - 用户名: 数据库用户名
    - 密码: 数据库密码
4. 点击「测试连接」验证
5. 点击「确定」保存连接

### PostgreSQL连接

1. 点击「连接」→「PostgreSQL」
2. 输入连接信息:
    - 主机名: localhost或IP地址
    - 端口: 5432（默认）
    - 数据库: 数据库名
    - 用户名: 数据库用户名
    - 密码: 数据库密码

### SQLite连接

1. 点击「连接」→「SQLite」
2. 选择数据库文件路径
3. 输入连接名

## 数据库操作

### 创建数据库

1. 右键连接 →「新建数据库」
2. 输入数据库名称
3. 选择字符集和排序规则

### 创建表

1. 打开数据库
2. 右键「表」→「新建表」
3. 添加字段:
    - 字段名
    - 类型
    - 长度
    - 是否允许为空
    - 默认值
    - 主键/外键
4. 点击「保存」

### 修改表结构

1. 右键表 →「设计表」
2. 修改字段信息
3. 点击「保存」

## 数据操作

### 查看数据

1. 双击表名打开数据视图
2. 使用筛选器过滤数据
3. 点击列标题排序

### 添加数据

1. 打开数据表
2. 点击「+」添加新行
3. 输入数据
4. 点击「√」保存

### 修改数据

1. 直接在单元格中修改
2. 点击「√」保存

### 删除数据

1. 选中行
2. 点击「-」删除
3. 确认删除

## SQL查询

### 执行SQL

1. 点击「查询」→「新建查询」
2. 输入SQL语句
3. 点击「运行」执行

### 查询示例

```sql
-- 查询数据
SELECT * FROM users WHERE status = 1;

-- 插入数据
INSERT INTO users (name, email, created_at)
VALUES ('John', 'john@example.com', NOW());

-- 更新数据
UPDATE users SET name = 'Jane' WHERE id = 1;

-- 删除数据
DELETE FROM users WHERE id = 1;

-- 聚合查询
SELECT COUNT(*) as total FROM users;
SELECT AVG(age) as avg_age FROM users;
```

## 数据导入导出

### 导出数据

1. 右键表 →「导出向导」
2. 选择导出格式（Excel、CSV、SQL等）
3. 选择导出路径
4. 点击「开始」

### 导入数据

1. 右键表 →「导入向导」
2. 选择导入文件
3. 配置导入选项
4. 点击「开始」

## 备份与恢复

### 备份数据库

1. 右键连接 →「备份」
2. 选择备份类型（完整备份、差异备份等）
3. 选择备份路径
4. 点击「开始」

### 恢复数据库

1. 右键连接 →「恢复」
2. 选择备份文件
3. 配置恢复选项
4. 点击「开始」

## 工具功能

### 数据同步

1. 点击「工具」→「数据同步」
2. 选择源和目标数据库
3. 选择要同步的表
4. 点击「比较」查看差异
5. 点击「同步」执行同步

### 结构同步

1. 点击「工具」→「结构同步」
2. 选择源和目标数据库
3. 点击「比较」查看结构差异
4. 点击「同步」执行同步

### 查询构建器

1. 点击「查询」→「查询构建器」
2. 选择表
3. 选择字段
4. 设置筛选条件
5. 点击「运行」

## 快捷键

```bash
Ctrl+N          # 新建连接/查询
Ctrl+S          # 保存
Ctrl+R          # 运行查询
Ctrl+F          # 查找
Ctrl+D          # 复制
Ctrl+X          # 剪切
Ctrl+V          # 粘贴
Ctrl+Z          # 撤销
Ctrl+Y          # 重做
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

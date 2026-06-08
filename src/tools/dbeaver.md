# DBeaver使用方法

## DBeaver是什么？
&emsp;&emsp;DBeaver是一款免费开源的数据库管理工具，支持多种数据库，功能强大且跨平台。

## 连接数据库

### 创建连接

1. 打开DBeaver
2. 点击「数据库」→「新建连接」
3. 选择数据库类型（MySQL、PostgreSQL、SQLite等）
4. 配置连接参数:
    - 主机名
    - 端口
    - 数据库名
    - 用户名和密码
5. 点击「测试连接」验证
6. 点击「完成」保存连接

### 连接管理

- 连接列表显示所有已配置的连接
- 双击连接打开/关闭连接
- 右键连接可编辑、删除、复制连接

## 数据库操作

### 创建数据库

1. 右键连接 →「创建数据库」
2. 输入数据库名称
3. 配置数据库参数
4. 点击「确定」

### 创建表

1. 展开数据库 → 右键「表」→「创建表」
2. 在编辑器中添加字段:
    - 名称
    - 类型
    - 长度/精度
    - 约束（主键、外键、唯一、非空）
    - 默认值
3. 点击「保存」

### 修改表

1. 右键表 →「编辑表」
2. 修改字段或添加新字段
3. 点击「保存」

## 数据操作

### 浏览数据

1. 双击表名打开数据视图
2. 使用过滤功能筛选数据
3. 点击列标题排序
4. 使用分页导航浏览大量数据

### 添加数据

1. 在数据视图中点击「+」添加新行
2. 输入字段值
3. 点击「保存」按钮

### 修改数据

1. 直接在单元格中编辑数据
2. 点击「保存」确认修改

### 删除数据

1. 选中要删除的行
2. 点击「-」删除
3. 确认删除操作

## SQL编辑器

### 打开编辑器

1. 点击「SQL」→「新建SQL脚本」
2. 或右键连接/表 →「新建SQL脚本」

### 执行SQL

```sql
-- 查询数据
SELECT * FROM users LIMIT 100;

-- 插入数据
INSERT INTO users (name, email) VALUES ('Alice', 'alice@example.com');

-- 更新数据
UPDATE users SET name = 'Bob' WHERE id = 1;

-- 删除数据
DELETE FROM users WHERE id = 1;
```

### 快捷键

```bash
Ctrl+Enter      # 执行选中的SQL或全部SQL
Ctrl+/          # 注释/取消注释
Ctrl+D          # 复制行
Ctrl+Shift+F    # 格式化SQL
Ctrl+Space      # 自动补全
```

## 数据导入导出

### 导出数据

1. 右键表 →「导出数据」
2. 选择导出格式（CSV、Excel、JSON等）
3. 配置导出选项
4. 点击「下一步」→「完成」

### 导入数据

1. 右键表 →「导入数据」
2. 选择导入文件
3. 配置导入选项（字段映射、数据类型等）
4. 点击「下一步」→「完成」

## 数据库比较

### 比较数据库结构

1. 点击「数据库」→「比较数据库」
2. 选择源数据库和目标数据库
3. 选择要比较的对象类型
4. 点击「比较」查看差异
5. 点击「应用」同步差异

### 比较数据表

1. 选择两个表
2. 右键 →「比较数据」
3. 查看数据差异
4. 选择同步方向

## ER图

### 生成ER图

1. 右键数据库 →「ER图」→「新建ER图」
2. 添加表到ER图
3. 调整布局
4. 导出ER图为图片或PDF

## 快捷键汇总

```bash
Ctrl+N          # 新建连接/脚本
Ctrl+S          # 保存
Ctrl+R          # 运行SQL
Ctrl+F          # 查找
Ctrl+H          # 替换
Ctrl+Z          # 撤销
Ctrl+Y          # 重做
Ctrl+Tab        # 切换标签页
Ctrl+W          # 关闭标签页
```

## 实用功能

### 元数据搜索

- 使用工具栏搜索框搜索数据库对象
- 支持模糊搜索

### 连接配置导出

1. 点击「文件」→「导出连接」
2. 选择要导出的连接
3. 选择导出格式
4. 保存配置文件

### 驱动管理

1. 点击「数据库」→「驱动管理器」
2. 添加或更新数据库驱动
3. 配置驱动路径

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

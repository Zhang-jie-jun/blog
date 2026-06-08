# SVN使用方法

## SVN是什么？
&emsp;&emsp;SVN（Subversion）是一个开源的集中式版本控制系统，用于管理文件和目录的变更历史。

## SVN与Git的最主要区别？
&emsp;&emsp;SVN是集中式版本控制系统，版本库集中存放在中央服务器，用户必须联网才能进行大部分操作。工作流程通常是：从服务器checkout代码 -> 修改 -> commit到服务器。

&emsp;&emsp;Git是分布式版本控制系统，每个用户的电脑都有完整的版本库，可以离线工作。用户之间通过push/pull来同步代码。

## 如何操作？

### 创建版本库（服务端）
&emsp;&emsp;在服务器上创建SVN版本库：
```bash
# 创建版本库目录
mkdir -p /svn/repos/project
# 创建版本库
svnadmin create /svn/repos/project
```

### 检出代码（Checkout）
&emsp;&emsp;从服务器检出代码到本地工作目录：
```bash
svn checkout svn://server_ip/repos/project /local/path
# 或者使用http协议
svn checkout http://server_ip/svn/project /local/path
```

### 更新代码（Update）
&emsp;&emsp;获取服务器上的最新代码：
```bash
# 更新当前目录
svn update
# 更新指定文件或目录
svn update file.txt
```

### 添加文件（Add）
&emsp;&emsp;将新文件添加到版本控制：
```bash
# 添加单个文件
svn add newfile.txt
# 添加目录及其内容
svn add newdir/
# 添加所有未版本化的文件
svn add *
```

### 提交修改（Commit）
&emsp;&emsp;将本地修改提交到服务器：
```bash
# 提交所有修改
svn commit -m "提交说明：修复了登录bug"
# 提交指定文件
svn commit -m "更新配置" config.ini
```

### 查看状态（Status）
&emsp;&emsp;查看工作目录的状态：
```bash
svn status
# 显示详细信息
svn status -v
```

状态符号说明：
- `A`: 已添加
- `D`: 已删除
- `M`: 已修改
- `?`: 未版本化
- `!`: 缺失或被删除

### 查看日志（Log）
&emsp;&emsp;查看提交历史：
```bash
# 查看所有日志
svn log
# 查看最近N条日志
svn log -l 10
# 查看指定文件的日志
svn log file.txt
```

### 版本回退
&emsp;&emsp;撤销本地修改或回滚到历史版本：

**撤销本地修改（未commit）：**
```bash
# 撤销单个文件
svn revert file.txt
# 撤销目录
svn revert dir/ -R
```

**回滚到历史版本：**
```bash
# 回滚到指定版本
svn merge -r HEAD:123 .
# 然后提交回滚
svn commit -m "回滚到版本123"
```

### 分支管理

#### 创建分支
```bash
# 在服务器上创建分支
svn copy http://server/svn/project/trunk \
         http://server/svn/project/branches/feature_branch \
         -m "创建功能分支"
```

#### 切换分支
```bash
# 检出分支
svn checkout http://server/svn/project/branches/feature_branch
```

#### 合并分支
```bash
# 切换到主干
cd trunk
# 将分支合并到主干
svn merge http://server/svn/project/branches/feature_branch
# 解决冲突后提交
svn commit -m "合并feature_branch到主干"
```

#### 删除分支
```bash
svn delete http://server/svn/project/branches/feature_branch \
           -m "删除分支"
```

## SVN目录结构规范
&emsp;&emsp;标准的SVN仓库结构：
```
project/
├── trunk/          # 主干开发
├── branches/       # 分支目录
│   ├── feature1/
│   └── bugfix/
└── tags/           # 标签目录
    ├── v1.0/
    └── v1.1/
```

## 常见问题

### 如何解决冲突？
&emsp;&emsp;当多人修改同一文件的同一部分时，会产生冲突。SVN会在冲突文件中标记冲突区域：
```
<<<<<<< .mine
我的修改内容
=======
其他人的修改内容
>>>>>>> .r123
```
&emsp;&emsp;手动编辑文件解决冲突后，标记冲突已解决：
```bash
svn resolved file.txt
```

### 如何忽略文件？
&emsp;&emsp;创建 `.svnignore` 文件，列出需要忽略的文件模式：
```
*.log
*.tmp
build/
.DS_Store
```
&emsp;&emsp;或者使用命令：
```bash
svn propset svn:ignore "*.log" .
```

### 如何查看文件差异？
```bash
# 查看本地修改与基础版本的差异
svn diff file.txt
# 查看两个版本之间的差异
svn diff -r 123:124 file.txt
```

## SVN常用命令
```
svn checkout (co):    检出代码
svn update (up):      更新代码
svn add:              添加文件
svn commit (ci):      提交修改
svn status (st):      查看状态
svn log:              查看日志
svn revert:           撤销修改
svn merge:            合并分支
svn copy (cp):        复制文件/创建分支
svn delete (del):     删除文件
svn diff (di):        查看差异
svn info:             查看信息
svn list (ls):        列出目录
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

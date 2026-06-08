# pip使用方法

## pip是什么？
&emsp;&emsp;pip是Python的包管理工具，用于安装、管理和发布Python包。

## 基本命令

### 安装包

```bash
# 安装指定包
pip install <package-name>

# 安装指定版本
pip install <package-name>==1.0.0

# 安装最低版本
pip install "<package-name>>=1.0.0"

# 安装到用户目录（无需管理员权限）
pip install --user <package-name>

# 安装开发版本
pip install git+https://github.com/user/repo.git
```

### 卸载包

```bash
# 卸载指定包
pip uninstall <package-name>

# 强制卸载（不提示确认）
pip uninstall -y <package-name>
```

### 更新包

```bash
# 更新指定包
pip install --upgrade <package-name>

# 更新pip自身
pip install --upgrade pip
```

### 查看已安装的包

```bash
# 列出所有已安装的包
pip list

# 列出已安装包的详细信息
pip show <package-name>

# 检查可更新的包
pip list --outdated
```

## requirements.txt

### 生成requirements.txt

```bash
# 导出当前环境的包列表
pip freeze > requirements.txt

# 导出指定格式
pip freeze --format=freeze > requirements.txt
```

### 从requirements.txt安装

```bash
# 安装所有依赖
pip install -r requirements.txt

# 升级依赖
pip install --upgrade -r requirements.txt

# 安装到指定目录
pip install -r requirements.txt --target=/path/to/dir
```

### requirements.txt格式示例

```txt
# 指定版本
requests==2.25.1
flask>=1.1.2
django~=3.1.0

# 从git安装
git+https://github.com/user/repo.git@branch

# 从本地安装
./path/to/package

# 条件依赖
docopt>=0.6.2;python_version<"3.0"
```

## 配置文件

### pip.conf / pip.ini

**Linux/macOS**: `~/.config/pip/pip.conf`
**Windows**: `%APPDATA%\pip\pip.ini`

```ini
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
trusted-host = pypi.tuna.tsinghua.edu.cn
timeout = 60
```

### 使用命令行配置

```bash
# 设置镜像源
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

# 查看配置
pip config list

# 重置配置
pip config unset global.index-url
```

## 虚拟环境

### 创建虚拟环境

```bash
# 使用venv（Python 3.3+）
python -m venv myenv

# 使用virtualenv
virtualenv myenv

# 指定Python版本
virtualenv -p python3.8 myenv
```

### 激活虚拟环境

```bash
# Linux/macOS
source myenv/bin/activate

# Windows（Command Prompt）
myenv\Scripts\activate.bat

# Windows（PowerShell）
myenv\Scripts\Activate.ps1
```

### 退出虚拟环境

```bash
deactivate
```

## 常用命令汇总

```bash
pip install <pkg>          # 安装包
pip uninstall <pkg>        # 卸载包
pip install --upgrade <pkg> # 更新包
pip list                   # 列出已安装的包
pip show <pkg>             # 查看包信息
pip freeze                # 导出依赖列表
pip install -r requirements.txt # 安装依赖文件
pip search <pkg>           # 搜索包（已弃用）
pip check                  # 检查依赖问题
pip cache purge            # 清理缓存
```

## 常见问题

### 安装速度慢
- 使用国内镜像源：
    ```bash
    pip install <package> -i https://pypi.tuna.tsinghua.edu.cn/simple
    ```

### 权限问题
- 使用 `--user` 选项安装到用户目录
- 或使用虚拟环境

### SSL错误
- 使用 `--trusted-host` 参数：
    ```bash
    pip install <package> --trusted-host pypi.org
    ```

### 编译错误
- 安装编译依赖（以Ubuntu为例）：
    ```bash
    sudo apt-get install python3-dev gcc
    ```

### 版本冲突
- 使用虚拟环境隔离不同项目的依赖
- 使用 `pip check` 检查依赖冲突

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

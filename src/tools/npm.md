# npm使用方法

## npm是什么？
&emsp;&emsp;npm（Node Package Manager）是Node.js的包管理工具，用于安装、管理和发布JavaScript包。

## 基本命令

### 初始化项目

```bash
# 初始化项目（交互式）
npm init

# 初始化项目（默认配置）
npm init -y

# 初始化项目并指定模板
npm init <template>
```

### 安装包

```bash
# 安装包到项目依赖
npm install <package-name>

# 安装指定版本
npm install <package-name>@1.0.0

# 安装到开发依赖
npm install <package-name> --save-dev

# 安装全局包
npm install -g <package-name>

# 安装所有依赖（根据package.json）
npm install
```

### 卸载包

```bash
# 卸载项目依赖
npm uninstall <package-name>

# 卸载开发依赖
npm uninstall <package-name> --save-dev

# 卸载全局包
npm uninstall -g <package-name>
```

### 更新包

```bash
# 更新指定包
npm update <package-name>

# 更新所有包
npm update

# 检查可更新的包
npm outdated
```

## package.json

### 基本结构

```json
{
"name": "my-project",
"version": "1.0.0",
"description": "My project description",
"main": "index.js",
"scripts": {
    "start": "node index.js",
    "test": "jest",
    "build": "webpack"
},
"dependencies": {
    "express": "^4.17.1",
    "lodash": "^4.17.20"
},
"devDependencies": {
    "webpack": "^5.0.0",
    "jest": "^26.0.0"
},
"keywords": ["node", "express"],
"author": "John Doe",
"license": "MIT"
}
```

### 版本号规则

```bash
^1.0.0  # 兼容1.x.x，不包括2.0.0
~1.0.0  # 兼容1.0.x，不包括1.1.0
1.x     # 匹配任何1.x版本
latest  # 安装最新版本
```

## 脚本命令

### 运行脚本

```bash
npm run <script-name>

# 示例
npm run start
npm run test
npm run build
```

### 常用脚本示例

```json
{
"scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "test": "mocha",
    "lint": "eslint .",
    "build": "babel src -d dist",
    "prestart": "npm run build",
    "postinstall": "npm run build"
}
}
```

## 配置文件

### .npmrc

```bash
# 设置npm仓库
registry=https://registry.npmmirror.com/

# 设置代理
proxy=http://proxy.example.com:8080

# 设置缓存目录
cache=/path/to/cache

# 保存时不生成package-lock.json
package-lock=false
```

### .gitignore

```bash
node_modules/
npm-debug.log
.DS_Store
dist/
build/
```

## 发布包

### 注册账号

```bash
# 注册npm账号
npm adduser

# 登录npm
npm login

# 查看当前用户
npm whoami
```

### 发布步骤

```bash
# 更新版本号
npm version patch   # 1.0.0 → 1.0.1
npm version minor   # 1.0.0 → 1.1.0
npm version major   # 1.0.0 → 2.0.0

# 发布包
npm publish

# 发布到指定仓库
npm publish --registry=https://registry.npmmirror.com/
```

## 常用命令汇总

```bash
npm init           # 初始化项目
npm install        # 安装依赖
npm install <pkg>  # 安装指定包
npm uninstall <pkg> # 卸载包
npm update         # 更新包
npm run <script>   # 运行脚本
npm test           # 运行测试
npm version        # 更新版本
npm publish        # 发布包
npm cache clean    # 清理缓存
npm config list    # 查看配置
npm ls             # 查看已安装的包
npm search <pkg>   # 搜索包
npm view <pkg>     # 查看包信息
```

## 常见问题

### 安装速度慢
- 使用国内镜像：`npm config set registry https://registry.npmmirror.com/`

### 权限问题
- 使用 `npm install -g` 时权限不足，可使用 `sudo` 或修改权限

### 版本冲突
- 使用 `npm ls` 查看依赖树
- 使用 `npm dedupe` 去重依赖

### 缓存问题
- 使用 `npm cache clean --force` 清理缓存

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

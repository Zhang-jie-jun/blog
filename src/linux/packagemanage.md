# Linux包管理命令详解

## 概述

Linux包管理是系统管理的重要部分，不同发行版使用不同的包管理工具。本文介绍Debian/Ubuntu（apt/dpkg）和CentOS/RHEL（yum/rpm）两大主流包管理系统。

---

## 一、Debian/Ubuntu包管理

### 1. apt命令

#### 更新软件源

```bash
# 更新软件源列表
sudo apt update

# 升级所有已安装的包
sudo apt upgrade

# 升级系统（包括内核）
sudo apt dist-upgrade
```

#### 安装软件

```bash
# 安装单个包
sudo apt install nginx

# 安装多个包
sudo apt install nginx mysql-server php

# 安装指定版本
sudo apt install nginx=1.18.0-0ubuntu1

# 安装时不提示
sudo apt install -y nginx
```

#### 卸载软件

```bash
# 卸载包（保留配置）
sudo apt remove nginx

# 彻底卸载（删除配置）
sudo apt purge nginx

# 清理无用的依赖
sudo apt autoremove

# 清理缓存
sudo apt clean
```

#### 搜索软件

```bash
# 搜索包
apt search nginx

# 查看包信息
apt show nginx

# 查看已安装的包
apt list --installed

# 查看可升级的包
apt list --upgradable
```

### 2. dpkg命令

#### 安装本地包

```bash
# 安装deb包
sudo dpkg -i package.deb

# 强制安装
sudo dpkg --force-all -i package.deb
```

#### 管理已安装的包

```bash
# 查看已安装的包
dpkg -l

# 查看包安装的文件
dpkg -L nginx

# 查看文件属于哪个包
dpkg -S /usr/bin/nginx

# 配置包
sudo dpkg-reconfigure nginx
```

#### 修复依赖问题

```bash
# 修复损坏的包
sudo dpkg --configure -a

# 修复依赖
sudo apt -f install
```

---

## 二、CentOS/RHEL包管理

### 1. yum命令

#### 更新软件

```bash
# 检查更新
yum check-update

# 升级所有包
sudo yum update

# 升级指定包
sudo yum update nginx
```

#### 安装软件

```bash
# 安装包
sudo yum install nginx

# 安装多个包
sudo yum install nginx mysql-server

# 安装指定版本
sudo yum install nginx-1.18.0
```

#### 卸载软件

```bash
# 卸载包
sudo yum remove nginx

# 卸载包及其依赖
sudo yum autoremove nginx

# 清理缓存
sudo yum clean all
```

#### 搜索软件

```bash
# 搜索包
yum search nginx

# 查看包信息
yum info nginx

# 查看已安装的包
yum list installed

# 查看包提供的文件
yum provides nginx
```

### 2. rpm命令

#### 安装本地包

```bash
# 安装rpm包
sudo rpm -ivh package.rpm

# 升级包
sudo rpm -Uvh package.rpm

# 强制安装
sudo rpm -ivh --force package.rpm
```

#### 管理已安装的包

```bash
# 查看已安装的包
rpm -qa

# 查看指定包
rpm -qa | grep nginx

# 查看包信息
rpm -qi nginx

# 查看包安装的文件
rpm -ql nginx

# 验证包
rpm -V nginx
```

---

## 三、常见问题

### 问题1：依赖冲突

```bash
# Debian/Ubuntu
sudo apt-get -f install

# CentOS/RHEL
sudo yum deplist nginx
sudo yum install --skip-broken
```

### 问题2：包损坏

```bash
# Debian/Ubuntu
sudo dpkg --configure -a
sudo apt clean
sudo apt update

# CentOS/RHEL
sudo yum clean all
sudo yum update
```

### 问题3：PPA源问题

```bash
# 添加PPA
sudo add-apt-repository ppa:nginx/stable
sudo apt update

# 删除PPA
sudo add-apt-repository --remove ppa:nginx/stable
```

### 问题4：找不到包

```bash
# 检查软件源
cat /etc/apt/sources.list

# 启用 universe/multiverse 仓库
sudo add-apt-repository universe
sudo add-apt-repository multiverse
```

---

## 四、实战案例

### 案例1：安装LAMP环境

```bash
# Debian/Ubuntu
sudo apt update
sudo apt install apache2 mysql-server php libapache2-mod-php php-mysql

# CentOS/RHEL
sudo yum install httpd mariadb-server php php-mysql
sudo systemctl start httpd mariadb
```

### 案例2：升级内核

```bash
# Debian/Ubuntu
sudo apt update
sudo apt dist-upgrade

# CentOS/RHEL
sudo yum update kernel
sudo reboot
```

---

## 五、面试题

**Q1：apt和dpkg有什么区别？**
- `dpkg`：底层工具，直接操作deb包
- `apt`：高层工具，自动处理依赖关系

**Q2：yum和rpm有什么区别？**
- `rpm`：底层工具，直接操作rpm包
- `yum`：高层工具，自动处理依赖关系

**Q3：如何修复依赖问题？**
```bash
# Debian/Ubuntu
sudo apt -f install

# CentOS/RHEL
sudo yum deplist <package>
```

**Q4：如何清理系统缓存？**
```bash
# Debian/Ubuntu
sudo apt clean

# CentOS/RHEL
sudo yum clean all
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

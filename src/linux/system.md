# Linux系统管理命令详解

## 概述

系统管理是Linux运维的核心，包括用户管理、权限管理、服务管理、定时任务等。本文介绍常用的系统管理命令及其使用场景。

---

## 一、用户管理

### 1. useradd/usermod/userdel

```bash
# 创建用户
sudo useradd username

# 创建用户并指定home目录
sudo useradd -m -d /home/username username

# 创建系统用户
sudo useradd -r username

# 修改用户信息
sudo usermod -l newname oldname

# 添加用户到组
sudo usermod -aG groupname username

# 删除用户
sudo userdel username

# 删除用户及home目录
sudo userdel -r username
```

### 2. groupadd/groupmod/groupdel

```bash
# 创建组
sudo groupadd groupname

# 修改组名
sudo groupmod -n newname oldname

# 删除组
sudo groupdel groupname
```

### 3. passwd

```bash
# 修改当前用户密码
passwd

# 修改其他用户密码
sudo passwd username

# 锁定用户
sudo passwd -l username

# 解锁用户
sudo passwd -u username

# 设置密码过期
sudo passwd -e username
```

---

## 二、权限管理

### 1. chmod

```bash
# 符号模式
chmod u+rwx,g+rx,o+r file.txt

# 数字模式
chmod 755 file.txt

# 递归修改目录权限
chmod -R 755 directory

# 添加执行权限
chmod +x script.sh
```

### 2. chown

```bash
# 修改所有者
sudo chown user:group file.txt

# 递归修改
sudo chown -R user:group directory

# 只修改所有者
sudo chown user file.txt

# 只修改组
sudo chgrp group file.txt
```

### 3. umask

```bash
# 查看当前umask
umask

# 设置umask
umask 022

# 临时设置（当前会话）
umask 002

# 永久设置
echo "umask 022" >> ~/.bashrc
```

---

## 三、服务管理

### 1. systemctl

```bash
# 启动服务
sudo systemctl start nginx

# 停止服务
sudo systemctl stop nginx

# 重启服务
sudo systemctl restart nginx

# 查看服务状态
sudo systemctl status nginx

# 设置开机自启
sudo systemctl enable nginx

# 禁用开机自启
sudo systemctl disable nginx

# 查看所有服务
sudo systemctl list-units --type=service

# 查看失败的服务
sudo systemctl list-units --type=service --state=failed
```

### 2. service（旧版）

```bash
# 启动服务
sudo service nginx start

# 停止服务
sudo service nginx stop

# 重启服务
sudo service nginx restart

# 查看状态
sudo service nginx status
```

---

## 四、定时任务

### 1. crontab

```bash
# 编辑定时任务
crontab -e

# 查看定时任务
crontab -l

# 删除定时任务
crontab -r

# 指定用户
sudo crontab -u username -e
```

### 2. cron表达式

```
*    *    *    *    *    command
-    -    -    -    -
|    |    |    |    |
|    |    |    |    +----- 星期 (0-7) (0和7代表周日)
|    |    |    +---------- 月份 (1-12)
|    |    +--------------- 日期 (1-31)
|    +-------------------- 小时 (0-23)
+------------------------- 分钟 (0-59)
```

### 3. 示例

```bash
# 每天凌晨2点执行
0 2 * * * /path/to/script.sh

# 每周一早上8点执行
0 8 * * 1 /path/to/script.sh

# 每小时执行
0 * * * * /path/to/script.sh

# 每30分钟执行
*/30 * * * * /path/to/script.sh

# 工作日的9点到17点，每小时执行
0 9-17 * * 1-5 /path/to/script.sh
```

---

## 五、系统信息

### 1. uname

```bash
# 查看内核版本
uname -r

# 查看系统信息
uname -a

# 查看操作系统
uname -s

# 查看主机名
uname -n
```

### 2. hostnamectl

```bash
# 查看系统信息
hostnamectl

# 设置主机名
sudo hostnamectl set-hostname myserver

# 设置静态主机名
sudo hostnamectl set-hostname --static myserver
```

### 3. lsb_release

```bash
# 查看发行版信息
lsb_release -a

# 查看发行版名称
lsb_release -d
```

---

## 六、系统维护

### 1. shutdown/reboot

```bash
# 立即关机
sudo shutdown -h now

# 10分钟后关机
sudo shutdown -h +10

# 定时关机
sudo shutdown -h 22:00

# 取消关机
sudo shutdown -c

# 重启
sudo reboot

# 立即重启
sudo shutdown -r now
```

### 2. sync

```bash
# 同步磁盘缓存
sync

# 关机前同步
sudo sync && sudo shutdown -h now
```

---

## 七、常见问题

### 问题1：忘记root密码

```bash
# 重启进入单用户模式
# 在GRUB菜单按e进入编辑模式
# 修改linux行，添加init=/bin/bash
# 按Ctrl+X启动

# 挂载根分区为可读写
mount -o remount,rw /

# 修改密码
passwd root

# 重启
exec /sbin/init
```

### 问题2：服务启动失败

```bash
# 查看服务状态
sudo systemctl status nginx

# 查看日志
sudo journalctl -u nginx

# 查看详细日志
sudo journalctl -u nginx -f
```

### 问题3：用户无法登录

```bash
# 检查用户是否锁定
passwd -S username

# 解锁用户
sudo passwd -u username

# 检查home目录权限
ls -la /home/username
```

---

## 八、面试题

**Q1：如何创建用户？**
```bash
sudo useradd -m username
```

**Q2：如何设置文件权限？**
```bash
chmod 755 file.txt
```

**Q3：如何设置定时任务？**
```bash
crontab -e
# 添加：0 2 * * * /path/to/script.sh
```

**Q4：如何管理系统服务？**
```bash
sudo systemctl start|stop|restart|status service_name
```

**Q5：如何修改主机名？**
```bash
sudo hostnamectl set-hostname newname
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

# 安全相关命令详解

## 概述

Linux系统安全是运维的重要部分，包括用户管理、权限控制、加密解密、日志审计等。本文介绍常用的安全相关命令。

---

## 一、用户与权限

### 1. sudo

```bash
# 以root权限执行命令
sudo command

# 切换到root
sudo -i

# 执行前询问密码
sudo -k command

# 查看sudo配置
sudo -l
```

### 2. su

```bash
# 切换到root
su -

# 切换到其他用户
su - username

# 执行单个命令
su -c "command" username
```

### 3. chmod/chown

```bash
# 设置文件权限
chmod 700 file.txt

# 设置所有者
chown user:group file.txt

# 递归修改目录权限
chmod -R 755 directory
```

---

## 二、加密与解密

### 1. openssl

```bash
# 生成RSA密钥对
openssl genrsa -out private.key 2048

# 生成公钥
openssl rsa -in private.key -pubout -out public.key

# 加密文件
openssl enc -aes-256-cbc -salt -in file.txt -out file.txt.enc

# 解密文件
openssl enc -aes-256-cbc -d -in file.txt.enc -out file.txt

# 生成MD5哈希
openssl md5 file.txt

# 生成SHA256哈希
openssl sha256 file.txt

# 生成自签名证书
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365
```

### 2. gpg

```bash
# 生成密钥对
gpg --gen-key

# 加密文件
gpg -e -r recipient@example.com file.txt

# 解密文件
gpg -d file.txt.gpg > file.txt

# 列出密钥
gpg --list-keys

# 导出公钥
gpg --armor --export username@example.com > public.key

# 导入密钥
gpg --import public.key
```

---

## 三、SSH安全

### 1. ssh-keygen

```bash
# 生成SSH密钥对
ssh-keygen -t rsa -b 4096

# 指定文件名
ssh-keygen -t rsa -b 4096 -f ~/.ssh/mykey

# 生成ED25519密钥
ssh-keygen -t ed25519
```

### 2. ssh-copy-id

```bash
# 复制公钥到远程服务器
ssh-copy-id user@hostname

# 指定端口
ssh-copy-id -p 2222 user@hostname

# 指定密钥文件
ssh-copy-id -i ~/.ssh/mykey user@hostname
```

### 3. SSH配置

```bash
# 禁止密码登录
echo "PasswordAuthentication no" >> /etc/ssh/sshd_config

# 禁止root登录
echo "PermitRootLogin no" >> /etc/ssh/sshd_config

# 重启SSH服务
sudo systemctl restart sshd
```

---

## 四、防火墙

### 1. iptables

```bash
# 查看规则
sudo iptables -L

# 允许SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# 允许HTTP
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# 允许HTTPS
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# 允许回环接口
sudo iptables -A INPUT -i lo -j ACCEPT

# 允许已建立的连接
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# 默认拒绝所有入站
sudo iptables -P INPUT DROP

# 保存规则
sudo iptables-save > /etc/iptables/rules.v4
```

### 2. ufw

```bash
# 启用防火墙
sudo ufw enable

# 允许SSH
sudo ufw allow ssh

# 允许HTTP
sudo ufw allow 80/tcp

# 允许HTTPS
sudo ufw allow 443/tcp

# 拒绝指定IP
sudo ufw deny from 192.168.1.100

# 查看状态
sudo ufw status
```

---

## 五、日志审计

### 1. journalctl

```bash
# 查看所有日志
journalctl

# 查看错误日志
journalctl -p err

# 查看指定服务日志
journalctl -u sshd

# 实时监控日志
journalctl -f

# 查看登录日志
journalctl _COMM=sshd
```

### 2. last/lastb

```bash
# 查看登录历史
last

# 查看失败登录
lastb

# 查看最近登录
last -n 10

# 查看指定用户登录
last username
```

### 3. auditd

```bash
# 查看审计日志
sudo auditctl -l

# 添加审计规则
sudo auditctl -w /etc/passwd -p wa -k passwd_changes

# 查看审计日志
ausearch -k passwd_changes

# 查看审计状态
sudo auditctl -s
```

---

## 六、SELinux

### 1. setenforce

```bash
# 查看SELinux状态
getenforce

# 临时禁用SELinux
sudo setenforce 0

# 临时启用SELinux
sudo setenforce 1

# 查看SELinux策略
sestatus
```

### 2. chcon

```bash
# 设置SELinux上下文
sudo chcon -t httpd_sys_content_t /var/www/html

# 永久设置上下文
sudo semanage fcontext -a -t httpd_sys_content_t "/var/www/html(/.*)?"
sudo restorecon -Rv /var/www/html
```

---

## 七、安全扫描

### 1. chkrootkit

```bash
# 检测rootkit
sudo chkrootkit
```

### 2. rkhunter

```bash
# 检测rootkit
sudo rkhunter --check
```

### 3. nmap

```bash
# 扫描本地端口
nmap localhost

# 扫描指定IP
nmap 192.168.1.1

# 扫描端口范围
nmap -p 1-1000 192.168.1.1

# 全面扫描
nmap -A 192.168.1.1
```

---

## 八、安全加固建议

### 1. 最小权限原则

```bash
# 禁止不必要的SUID/SGID权限
find / -type f -perm /6000 -exec ls -la {} \;

# 删除不必要的SUID权限
chmod -s /path/to/file
```

### 2. 定期更新系统

```bash
# Debian/Ubuntu
sudo apt update && sudo apt upgrade

# CentOS/RHEL
sudo yum update
```

### 3. 日志监控

```bash
# 监控登录失败
grep "Failed password" /var/log/auth.log
```

---

## 九、实战案例

### 案例1：配置SSH安全

```bash
# 生成密钥对
ssh-keygen -t ed25519

# 复制公钥到服务器
ssh-copy-id user@hostname

# 禁用密码登录
sudo sed -i 's/PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo systemctl restart sshd
```

### 案例2：配置防火墙

```bash
# 启用ufw
sudo ufw enable

# 允许必要的端口
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# 查看状态
sudo ufw status
```

---

## 十、面试题

**Q1：如何配置SSH免密登录？**
```bash
ssh-keygen -t rsa
ssh-copy-id user@hostname
```

**Q2：如何禁止root远程登录？**
```bash
echo "PermitRootLogin no" >> /etc/ssh/sshd_config
sudo systemctl restart sshd
```

**Q3：如何使用iptables开放端口？**
```bash
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
```

**Q4：如何生成SSL证书？**
```bash
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365
```

**Q5：如何检测rootkit？**
```bash
sudo chkrootkit
sudo rkhunter --check
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

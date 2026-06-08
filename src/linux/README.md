# Linux常用命令

## 目录结构

| 文档 | 内容 |
|------|------|
| [Linux三剑客](grep.md) | grep、sed、awk文本处理工具详解 |
| [磁盘I/O统计](iostat.md) | iostat命令使用方法及性能分析 |
| [挂载命令](mount.md) | mount命令使用及文件系统挂载 |
| [fstab配置](fstab.md) | /etc/fstab文件系统自动挂载配置 |
| [块设备信息](lsblk.md) | lsblk查看磁盘分区信息 |
| [磁盘空间](df.md) | df查看磁盘空间使用情况 |
| [目录大小](du.md) | du统计目录占用空间 |
| [内存使用](free.md) | free查看内存使用情况 |
| [进程监控](top.md) | top实时进程监控工具 |
| [网络状态](netstat.md) | netstat查看网络连接状态 |
| [进程状态](ps.md) | ps查看进程状态信息 |
| [进程终止](kill.md) | kill命令终止进程 |
| [压缩解压](tar.md) | tar打包压缩工具 |
| [磁盘管理](disk.md) | 磁盘分区、GPT分区模式、完整磁盘配置流程 |
| [文件句柄](filehandle.md) | 文件句柄查看、配置限制、问题定位 |
| [dd命令](dd.md) | 磁盘性能测试、数据创建、数据清理 |
| [包管理](packagemanage.md) | apt、yum、dpkg、rpm包管理工具 |
| [网络管理](network.md) | 网络配置、测试、故障诊断 |
| [系统管理](system.md) | 用户、权限、服务、定时任务管理 |
| [系统监控](monitor.md) | CPU、内存、磁盘、进程监控 |
| [文件操作](file.md) | 文件创建、复制、移动、查找 |
| [文本处理](text.md) | grep、sed、awk、sort等文本工具 |
| [安全相关](security.md) | SSH、加密、防火墙、审计 |

## 命令分类

### 文件与文本处理命令
- **cp**：文件复制
- **mv**：文件移动/重命名
- **rm**：文件删除
- **mkdir**：创建目录
- **find**：文件查找
- **locate**：快速文件定位
- **head/tail**：文件查看
- **grep**：文本搜索工具
- **sed**：流编辑器
- **awk**：文本处理语言
- **sort**：文本排序
- **uniq**：去重
- **cut**：字段提取
- **paste**：列合并
- **tr**：字符转换

### 包管理命令
- **apt**：Debian/Ubuntu包管理
- **yum**：CentOS/RHEL包管理
- **dpkg**：Debian包操作
- **rpm**：RPM包操作

### 磁盘管理命令
- **iostat**：磁盘I/O统计
- **mount**：挂载文件系统
- **df**：磁盘空间使用
- **du**：目录大小统计
- **lsblk**：块设备信息
- **disk**：磁盘分区与GPT配置
- **dd**：磁盘性能测试与数据操作

### 网络管理命令
- **ip**：网络接口配置
- **curl**：HTTP客户端
- **wget**：文件下载
- **ssh**：安全远程登录
- **scp**：安全文件传输
- **rsync**：文件同步
- **dig**：DNS查询
- **nslookup**：域名解析

### 系统管理命令
- **systemctl**：系统服务管理
- **crontab**：定时任务管理
- **useradd/usermod/userdel**：用户管理
- **groupadd/groupmod/groupdel**：组管理
- **chmod**：权限管理
- **chown**：所有权管理
- **hostnamectl**：主机名管理

### 系统监控命令
- **top**：进程实时监控
- **ps**：进程状态查看
- **free**：内存使用情况
- **netstat**：网络连接状态
- **lsof**：文件句柄管理
- **iostat**：磁盘I/O统计
- **vmstat**：虚拟内存统计
- **mpstat**：CPU统计
- **tcpdump**：网络抓包
- **journalctl**：系统日志

### 安全相关命令
- **openssl**：加密工具
- **gpg**：加密工具
- **iptables**：防火墙配置
- **ufw**：简化防火墙
- **last/lastb**：登录审计

### 其他工具命令
- **kill**：终止进程
- **tar**：打包压缩工具

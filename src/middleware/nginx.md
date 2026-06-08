# Nginx详解

## Nginx是什么？
&emsp;&emsp;Nginx是一个高性能的HTTP和反向代理服务器，由Igor Sysoev开发。它以高并发、低内存消耗、稳定性强著称，广泛应用于Web服务器、负载均衡、反向代理等场景。

## Nginx的特点

| 特性 | 说明 |
|------|------|
| **高性能** | 支持数万级并发连接 |
| **低内存** | 内存占用低 |
| **稳定性** | 高可靠运行 |
| **模块化** | 丰富的模块支持 |
| **热部署** | 支持配置热更新 |
| **负载均衡** | 内置多种负载均衡策略 |

---

## 一、Nginx核心原理

### 1.1 Nginx架构

```
                    ┌─────────────────────────────────────────────────────────┐
                    │                    客户端请求                            │
                    └───────────────────────────┬─────────────────────────────┘
                                                │
                                                ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                          Nginx Server                                     │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐                  │
│  │   Master     │    │   Worker 1   │    │   Worker N   │                  │
│  │  (管理进程)  │    │  (工作进程)  │    │  (工作进程)  │                  │
│  │  配置加载    │    │  处理请求    │    │  处理请求    │                  │
│  │  信号管理    │    │  连接处理    │    │  连接处理    │                  │
│  └──────┬───────┘    └──────┬───────┘    └──────┬───────┘                  │
│         │                   │                   │                          │
│         │ 信号              │ 共享内存          │ 共享内存                   │
│         ▼                   ▼                   ▼                          │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐                  │
│  │   配置文件    │    │   连接池     │    │   缓存       │                  │
│  │   nginx.conf │    │              │    │              │                  │
│  └──────────────┘    └──────────────┘    └──────────────┘                  │
└─────────────────────────────────────────────────────────────────────────────┘
                                                │
                                                ▼
                    ┌─────────────────────────────────────────────────────────┐
                    │                   后端服务器                            │
                    │  ┌──────────┐  ┌──────────┐  ┌──────────┐              │
                    │  │  Server1 │  │  Server2 │  │  Server3 │              │
                    │  └──────────┘  └──────────┘  └──────────┘              │
                    └─────────────────────────────────────────────────────────┘
```

### 1.2 Nginx进程模型

**Master进程**：
- 读取和验证配置文件
- 管理Worker进程
- 处理信号
- 不处理请求

**Worker进程**：
- 处理实际的请求
- 多个Worker进程实现并发
- 通过共享内存共享数据

**配置示例**：
```nginx
worker_processes 4;  # Worker进程数，通常等于CPU核心数

events {
    worker_connections 1024;  # 每个Worker最大连接数
    use epoll;  # 使用epoll事件模型
}
```

### 1.3 事件驱动模型

**事件模型对比**：

| 模型 | 说明 | 适用场景 |
|------|------|----------|
| **select** | 轮询所有连接 | 连接数少 |
| **poll** | 轮询所有连接 | 连接数少 |
| **epoll** | 事件通知 | 高并发 |
| **kqueue** | 事件通知 | BSD系统 |

**epoll工作原理**：
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   epoll_create  │───►│   epoll_ctl     │───►│   epoll_wait    │
│   (创建实例)     │    │   (注册事件)     │    │   (等待事件)     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### 1.4 HTTP请求处理流程

```
请求 ──► 接收连接 ──► 解析请求 ──► 查找配置 ──► 处理请求 ──► 响应客户端
                                              │
                                              ▼
                                        反向代理/静态文件/动态内容
```

**配置示例**：
```nginx
http {
    server {
        listen 80;
        server_name example.com;
        
        location / {
            root /var/www/html;
            index index.html;
        }
        
        location /api {
            proxy_pass http://backend;
            proxy_set_header Host $host;
        }
    }
}
```

---

## 二、Nginx常见问题分析与排查

### 问题1：Nginx无法启动

**现象**：
```
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
```

**排查步骤**：
```bash
# 1. 检查端口占用
netstat -tlnp | grep :80

# 2. 检查配置文件
nginx -t

# 3. 检查日志
cat /var/log/nginx/error.log
```

**解决方案**：
1. 释放占用端口的进程
2. 修改监听端口
3. 修复配置文件错误

---

### 问题2：404错误

**现象**：
```
404 Not Found
```

**排查步骤**：
```bash
# 1. 检查配置文件
cat /etc/nginx/nginx.conf

# 2. 检查文件路径
ls -la /var/www/html

# 3. 检查访问日志
cat /var/log/nginx/access.log | grep 404
```

**解决方案**：
1. 检查root和index配置
2. 确认文件存在
3. 检查location匹配规则

---

### 问题3：502 Bad Gateway

**现象**：
```
502 Bad Gateway
```

**排查步骤**：
```bash
# 1. 检查后端服务器状态
curl http://backend:8080

# 2. 检查代理配置
cat /etc/nginx/nginx.conf | grep proxy_pass

# 3. 检查错误日志
cat /var/log/nginx/error.log | grep -i error
```

**解决方案**：
1. 确保后端服务器正常运行
2. 检查代理配置正确
3. 增加超时时间

---

### 问题4：性能问题

**现象**：
- 响应慢
- CPU使用率高
- 内存占用高

**排查步骤**：
```bash
# 1. 检查进程状态
top -p $(pgrep nginx)

# 2. 检查连接数
netstat -tlnp | grep nginx | wc -l

# 3. 检查日志
cat /var/log/nginx/access.log | tail -100
```

**解决方案**：
1. 增加Worker进程数
2. 优化事件模型
3. 启用缓存

---

### 问题5：配置文件错误

**现象**：
```
nginx: [emerg] invalid parameter "xxx" in /etc/nginx/nginx.conf:xx
```

**排查步骤**：
```bash
# 1. 验证配置文件
nginx -t

# 2. 检查语法错误
nginx -t -c /etc/nginx/nginx.conf

# 3. 查看错误详情
cat /var/log/nginx/error.log
```

**解决方案**：
1. 修复语法错误
2. 检查模块是否加载
3. 查看官方文档

---

### 问题6：SSL证书问题

**现象**：
```
SSL certificate error
```

**排查步骤**：
```bash
# 1. 检查证书文件
openssl x509 -in /etc/nginx/cert.crt -text -noout

# 2. 检查配置
cat /etc/nginx/nginx.conf | grep ssl

# 3. 测试SSL连接
openssl s_client -connect example.com:443
```

**解决方案**：
1. 确保证书文件正确
2. 配置正确的SSL参数
3. 更新过期证书

---

### 问题7：连接超时

**现象**：
```
504 Gateway Timeout
```

**排查步骤**：
```bash
# 1. 检查超时配置
cat /etc/nginx/nginx.conf | grep timeout

# 2. 检查后端响应时间
curl -w "%{time_total}\n" http://backend:8080

# 3. 检查错误日志
cat /var/log/nginx/error.log | grep timeout
```

**解决方案**：
1. 增加超时时间配置
2. 优化后端响应
3. 使用异步处理

---

### 问题8：缓存问题

**现象**：
- 页面不更新
- 缓存不生效

**排查步骤**：
```bash
# 1. 检查缓存配置
cat /etc/nginx/nginx.conf | grep cache

# 2. 检查缓存目录
ls -la /var/cache/nginx

# 3. 测试缓存效果
curl -I http://example.com/static/js/app.js
```

**解决方案**：
1. 配置正确的缓存规则
2. 检查缓存目录权限
3. 设置合适的缓存时间

---

### 问题9：负载均衡问题

**现象**：
- 请求分布不均
- 部分服务器负载高

**排查步骤**：
```bash
# 1. 检查负载均衡配置
cat /etc/nginx/nginx.conf | grep upstream

# 2. 检查后端服务器状态
curl http://example.com/health

# 3. 查看访问日志统计
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c
```

**解决方案**：
1. 选择合适的负载均衡策略
2. 配置健康检查
3. 调整权重

---

### 问题10：文件上传问题

**现象**：
- 文件上传失败
- 413 Request Entity Too Large

**排查步骤**：
```bash
# 1. 检查上传大小限制
cat /etc/nginx/nginx.conf | grep client_max_body_size

# 2. 检查后端配置
cat /etc/php/php.ini | grep upload_max_filesize

# 3. 检查错误日志
cat /var/log/nginx/error.log | grep 413
```

**解决方案**：
1. 增加`client_max_body_size`
2. 调整后端上传限制
3. 配置临时文件目录

---

## 三、Nginx面试题

### 基础问题

**Q1：Nginx是什么？有什么特点？**

**A1：**
Nginx是一个高性能的HTTP和反向代理服务器，具有以下特点：
- 高性能：支持数万级并发连接
- 低内存：内存占用低
- 稳定性：高可靠运行
- 模块化：丰富的模块支持
- 热部署：支持配置热更新
- 负载均衡：内置多种负载均衡策略

---

**Q2：Nginx的进程模型是怎样的？**

**A2：**
- **Master进程**：管理配置和Worker进程，不处理请求
- **Worker进程**：处理实际请求，多个Worker实现并发
- 通过共享内存共享数据

---

**Q3：Nginx支持哪些事件模型？**

**A3：**
- **select**：轮询模型，适用于连接数少的场景
- **poll**：轮询模型，比select稍好
- **epoll**：事件通知模型，适用于高并发场景
- **kqueue**：BSD系统的事件通知模型

---

**Q4：Nginx的负载均衡策略有哪些？**

**A4：**
- **round_robin**：轮询（默认）
- **least_conn**：最少连接
- **ip_hash**：基于IP哈希
- **fair**：基于响应时间
- **url_hash**：基于URL哈希

---

**Q5：Nginx和Apache有什么区别？**

**A5：**

| 特性 | Nginx | Apache |
|------|-------|--------|
| 并发性能 | 高 | 中等 |
| 内存占用 | 低 | 高 |
| 配置复杂度 | 中等 | 复杂 |
| 模块扩展 | 静态编译 | 动态加载 |
| 适用场景 | 高并发、反向代理 | 动态内容、.htaccess |

---

**Q6：什么是反向代理？Nginx如何配置反向代理？**

**A6：**
反向代理是代理服务器接收客户端请求，转发给后端服务器，然后将响应返回给客户端。

```nginx
location /api {
    proxy_pass http://backend;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
}
```

---

**Q7：Nginx如何实现HTTPS？**

**A7：**
```nginx
server {
    listen 443 ssl;
    server_name example.com;
    
    ssl_certificate /etc/nginx/cert.crt;
    ssl_certificate_key /etc/nginx/cert.key;
    
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
}
```

---

**Q8：Nginx的缓存机制是怎样的？**

**A8：**
Nginx支持多级缓存：
- 浏览器缓存：通过`expires`和`Cache-Control`头控制
- 代理缓存：通过`proxy_cache`配置
- FastCGI缓存：通过`fastcgi_cache`配置

---

**Q9：Nginx如何处理静态文件？**

**A9：**
```nginx
location /static {
    root /var/www/html;
    expires 30d;  # 设置缓存时间
    add_header Cache-Control "public, immutable";
}
```

---

**Q10：Nginx如何实现限流？**

**A10：**
```nginx
limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;

server {
    location /api {
        limit_req zone=one burst=5;
    }
}
```

---

### 复杂场景问题

**Q11：如何设计一个高可用的Nginx架构？**

**A11：**

**架构设计：**
```
                    ┌─────────────────────────────┐
                    │           VIP               │
                    └─────────────┬───────────────┘
                                  │
        ┌─────────────────────────┼─────────────────────────┐
        ▼                         ▼                         ▼
┌──────────────┐          ┌──────────────┐          ┌──────────────┐
│   Nginx 1    │          │   Nginx 2    │          │   Nginx 3    │
│  (Active)    │◄─────────►│  (Standby)  │◄─────────►│  (Standby)  │
│  Keepalived  │          │  Keepalived  │          │  Keepalived  │
└──────────────┘          └──────────────┘          └──────────────┘
        │                         │                         │
        └─────────────────────────┼─────────────────────────┘
                                  │
                                  ▼
                    ┌─────────────────────────────┐
                    │        后端服务器集群         │
                    │  ┌──────────┐  ┌──────────┐ │
                    │  │ Server1  │  │ Server2  │ │
                    │  └──────────┘  └──────────┘ │
                    └─────────────────────────────┘
```

**关键配置：**
```nginx
upstream backend {
    server server1:8080 weight=3;
    server server2:8080 weight=2;
    server server3:8080 backup;
    
    ip_hash;  # 保持会话一致性
    keepalive 100;  # 连接复用
}
```

**故障切换流程：**
1. Keepalived检测Nginx状态
2. 主Nginx故障时，VIP切换到备用节点
3. 备用节点接管流量
4. 通知运维人员处理

---

**Q12：如何处理Nginx的性能瓶颈？**

**A12：**

**性能分析：**
```bash
# 查看连接数
netstat -tlnp | grep nginx | wc -l

# 查看Worker状态
ps aux | grep nginx

# 查看请求响应时间
curl -w "%{time_total}\n" http://example.com
```

**优化方案：**

1. **调整Worker进程数**：
```nginx
worker_processes auto;  # 自动设置为CPU核心数
```

2. **优化事件模型**：
```nginx
events {
    worker_connections 10240;
    use epoll;
    multi_accept on;
}
```

3. **启用压缩**：
```nginx
gzip on;
gzip_types text/html text/css application/json;
gzip_comp_level 6;
```

4. **启用缓存**：
```nginx
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m max_size=10g inactive=60m;

location /api {
    proxy_cache my_cache;
    proxy_cache_valid 200 30m;
}
```

---

**Q13：如何实现Nginx的WAF（Web应用防火墙）？**

**A13：**

**方案一：使用ngx_http_limit_req_module限流**
```nginx
limit_req_zone $binary_remote_addr zone=rate_limit:10m rate=10r/s;

location /api {
    limit_req zone=rate_limit burst=20 nodelay;
}
```

**方案二：使用ngx_http_secure_link_module防盗链**
```nginx
location /download {
    secure_link $arg_md5,$arg_expires;
    secure_link_md5 "$secure_link_expires$uri$remote_addr secret";
    
    if ($secure_link = "") {
        return 403;
    }
    if ($secure_link = "0") {
        return 410;
    }
}
```

**方案三：使用ModSecurity模块**
```nginx
location / {
    modsecurity on;
    modsecurity_rules_file /etc/nginx/modsecurity.conf;
}
```

**方案四：自定义规则**
```nginx
# 阻止SQL注入
if ($request_uri ~* "(select|union|insert|delete|update|drop)") {
    return 403;
}

# 阻止XSS攻击
if ($request_body ~* "<script.*>") {
    return 403;
}
```

---

**Q14：如何设计Nginx的监控体系？**

**A14：**

**监控指标：**

| 类别 | 指标 |
|------|------|
| **连接** | 活跃连接数、请求数 |
| **性能** | 请求响应时间、吞吐量 |
| **错误** | 错误率、5xx错误数 |
| **资源** | CPU、内存、磁盘I/O |

**监控工具：**
- **Prometheus + Grafana**：指标采集和可视化
- **nginx-exporter**：Nginx指标导出
- **ELK**：日志分析
- **Zabbix/Nagios**：告警监控

**配置示例：**
```nginx
server {
    location /metrics {
        stub_status on;
        access_log off;
        allow 127.0.0.1;
        deny all;
    }
}
```

**告警规则：**
```yaml
- alert: HighErrorRate
  expr: sum(rate(nginx_http_requests_total{status=~"5.."}[5m])) / sum(rate(nginx_http_requests_total[5m])) > 0.05
  for: 5m
  labels:
    severity: critical
```

---

**Q15：如何实现Nginx的灰度发布？**

**A15：**

**方案一：基于IP的灰度发布**
```nginx
map $remote_addr $group {
    default normal;
    192.168.1.0/24 canary;
}

upstream normal {
    server server1:8080;
}

upstream canary {
    server server2:8080;
}

server {
    location / {
        proxy_pass http://$group;
    }
}
```

**方案二：基于Cookie的灰度发布**
```nginx
map $cookie_canary $group {
    default normal;
    1 canary;
}

server {
    location / {
        proxy_pass http://$group;
    }
}
```

**方案三：基于Header的灰度发布**
```nginx
map $http_x_canary $group {
    default normal;
    true canary;
}

server {
    location / {
        proxy_pass http://$group;
    }
}
```

**方案四：基于权重的灰度发布**
```nginx
upstream backend {
    server server1:8080 weight=9;  # 90%流量
    server server2:8080 weight=1;  # 10%流量
}

server {
    location / {
        proxy_pass http://backend;
    }
}
```

---

**Q16：如何处理Nginx的日志管理？**

**A16：**

**日志配置：**
```nginx
http {
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';
    
    access_log /var/log/nginx/access.log main;
    error_log /var/log/nginx/error.log warn;
}
```

**日志轮转：**
```bash
# /etc/logrotate.d/nginx
/var/log/nginx/*.log {
    daily
    missingok
    rotate 52
    compress
    delaycompress
    notifempty
    create 0640 www-data adm
    sharedscripts
    postrotate
        [ -f /var/run/nginx.pid ] && kill -USR1 `cat /var/run/nginx.pid`
    endscript
}
```

**日志分析：**
```bash
# 统计访问量
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -nr | head -10

# 统计状态码
awk '{print $9}' /var/log/nginx/access.log | sort | uniq -c

# 查找慢请求
awk '$NF > 10' /var/log/nginx/access.log
```

---

**Q17：如何实现Nginx的HTTPS优化？**

**A17：**

**配置优化：**

1. **启用TLS 1.2+**：
```nginx
ssl_protocols TLSv1.2 TLSv1.3;
ssl_prefer_server_ciphers on;
ssl_ciphers "EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH";
```

2. **启用OCSP Stapling**：
```nginx
ssl_stapling on;
ssl_stapling_verify on;
ssl_trusted_certificate /etc/nginx/ca-bundle.crt;
```

3. **启用HTTP/2**：
```nginx
listen 443 ssl http2;
```

4. **启用HSTS**：
```nginx
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
```

5. **启用缓存**：
```nginx
ssl_session_cache shared:SSL:10m;
ssl_session_timeout 10m;
```

---

**Q18：如何处理Nginx的大文件下载？**

**A18：**

**配置优化：**

1. **增加超时时间**：
```nginx
proxy_connect_timeout 60s;
proxy_send_timeout 60s;
proxy_read_timeout 60s;
```

2. **启用分块传输**：
```nginx
proxy_buffering off;
chunked_transfer_encoding on;
```

3. **配置sendfile**：
```nginx
sendfile on;
tcp_nopush on;
tcp_nodelay on;
```

4. **设置合适的缓冲区**：
```nginx
proxy_buffer_size 128k;
proxy_buffers 4 256k;
proxy_busy_buffers_size 256k;
```

---

**Q19：如何实现Nginx的负载均衡健康检查？**

**A19：**

**方案一：使用ngx_http_proxy_module**
```nginx
upstream backend {
    server server1:8080 max_fails=3 fail_timeout=30s;
    server server2:8080 max_fails=3 fail_timeout=30s;
}

server {
    location / {
        proxy_pass http://backend;
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
    }
}
```

**方案二：使用ngx_http_upstream_check_module**
```nginx
upstream backend {
    server server1:8080;
    server server2:8080;
    
    check interval=3000 rise=2 fall=3 timeout=1000;
    check_http_send "HEAD /health HTTP/1.0\r\n\r\n";
    check_http_expect_alive http_2xx http_3xx;
}
```

**方案三：使用第三方健康检查**
```bash
# 使用keepalived或haproxy进行健康检查
```

---

**Q20：如何进行Nginx的配置管理和版本控制？**

**A20：**

**配置管理策略：**

1. **配置文件结构**：
```
/etc/nginx/
├── nginx.conf          # 主配置文件
├── conf.d/             # 站点配置
│   ├── example.com.conf
│   └── api.example.com.conf
├── ssl/                # SSL证书
│   ├── cert.crt
│   └── cert.key
└── snippets/           # 可复用配置
    ├── ssl.conf
    └── proxy.conf
```

2. **版本控制**：
```bash
# 使用Git管理配置
git init /etc/nginx
git add .
git commit -m "Initial commit"
```

3. **配置测试**：
```bash
# 在应用前测试配置
nginx -t

# 热更新配置
nginx -s reload
```

4. **配置备份**：
```bash
# 定期备份配置
tar -czf nginx-config-$(date +%Y%m%d).tar.gz /etc/nginx
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

# CDN详细教程

## 概述

CDN（Content Delivery Network）是一种分布式网络架构，通过在全球各地部署边缘节点，将内容缓存到离用户最近的位置，从而加速内容分发，降低延迟，提高用户体验。

---

## 一、CDN基本概念

### 1. CDN定义

CDN是一个分布式系统，包含以下核心组件：

| 组件 | 说明 |
|------|------|
| **源站** | 原始内容服务器，存储原始数据 |
| **边缘节点** | 分布在各地的缓存服务器 |
| **DNS解析** | 将域名解析到最近的边缘节点 |
| **缓存策略** | 决定内容如何缓存和更新 |

### 2. CDN工作原理

```
用户请求 → DNS解析 → 最近的边缘节点 → 缓存命中 → 返回内容
                                    ↓
                              缓存未命中 → 回源获取 → 缓存 → 返回内容
```

**详细流程**：

1. **用户发起请求**：用户访问网站，请求静态资源
2. **DNS解析**：权威DNS将域名解析到CDN的负载均衡器
3. **智能路由**：负载均衡器根据用户位置、节点负载等因素选择最优边缘节点
4. **缓存查询**：边缘节点检查请求的资源是否在缓存中
5. **缓存命中**：直接返回缓存的内容
6. **缓存未命中**：边缘节点向源站请求内容，缓存后返回给用户

---

## 二、CDN架构

### 1. 架构层次

```
┌─────────────────────────────────────────────────────────────┐
│                    用户层 (User)                            │
│  - 浏览器、移动应用、API客户端                               │
├─────────────────────────────────────────────────────────────┤
│                    DNS层 (DNS)                             │
│  - 权威DNS、递归DNS、CDN DNS                                │
├─────────────────────────────────────────────────────────────┤
│                    负载均衡层 (Load Balancer)               │
│  - 全局负载均衡(GSLB)、本地负载均衡                         │
├─────────────────────────────────────────────────────────────┤
│                    边缘节点层 (Edge Nodes)                  │
│  - 缓存服务器、SSL终止、内容优化                            │
├─────────────────────────────────────────────────────────────┤
│                    源站层 (Origin Server)                   │
│  - 原始内容存储、动态内容处理                               │
└─────────────────────────────────────────────────────────────┘
```

### 2. 核心组件详解

#### 全局负载均衡（GSLB）

**功能**：
- 根据用户IP判断地理位置
- 检测各边缘节点的健康状态和负载
- 选择最优节点响应用户请求

**算法**：
- **基于地理位置**：选择距离最近的节点
- **基于负载**：选择负载最低的节点
- **基于延迟**：选择响应最快的节点

#### 边缘节点

**功能**：
- 缓存静态资源（图片、CSS、JS、视频等）
- 处理HTTPS请求（SSL终止）
- 内容优化（压缩、图片优化）
- 日志记录和统计

#### 缓存策略

| 策略 | 说明 |
|------|------|
| **时间过期** | 根据TTL（Time To Live）自动过期 |
| **内容更新** | 源站更新后主动推送或PURGE |
| **缓存键** | 根据URL、查询参数等生成唯一键 |
| **缓存层次** | 多级缓存（浏览器缓存、CDN缓存、源站缓存） |

---

## 三、CDN缓存机制

### 1. 缓存类型

| 类型 | 说明 |
|------|------|
| **浏览器缓存** | 用户浏览器本地缓存 |
| **CDN边缘缓存** | 边缘节点的缓存 |
| **源站缓存** | 源站服务器的缓存 |

### 2. 缓存控制头部

```http
# 设置缓存时间（1小时）
Cache-Control: max-age=3600

# 不缓存
Cache-Control: no-store

# 必须验证
Cache-Control: no-cache

# 公共缓存（可被CDN缓存）
Cache-Control: public

# 私有缓存（仅浏览器缓存）
Cache-Control: private
```

### 3. 缓存更新策略

#### 方式1：时间过期（TTL）

```http
# 设置资源缓存1小时
Cache-Control: max-age=3600
```

#### 方式2：主动刷新（PURGE）

```bash
# 清除单个URL缓存
curl -X PURGE https://cdn.example.com/image.jpg

# 清除目录下所有缓存
curl -X PURGE https://cdn.example.com/images/
```

#### 方式3：版本控制

```http
# 通过URL版本号更新
https://cdn.example.com/app.v2.js
https://cdn.example.com/style.v3.css
```

#### 方式4：查询参数

```http
# 通过查询参数缓存
https://cdn.example.com/image.jpg?v=1.0.0
```

---

## 四、CDN配置示例

### 1. Nginx作为CDN边缘节点

```nginx
http {
    # 缓存目录
    proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=cdn_cache:10m max_size=10g inactive=60m use_temp_path=off;

    server {
        listen 80;
        server_name cdn.example.com;

        location / {
            # 启用缓存
            proxy_cache cdn_cache;
            
            # 缓存键（包含查询参数）
            proxy_cache_key "$scheme$request_method$host$request_uri";
            
            # 缓存时间
            proxy_cache_valid 200 302 1h;
            proxy_cache_valid 404 1m;
            
            # 设置缓存控制头部
            add_header X-Cache-Status $upstream_cache_status;
            
            # 回源配置
            proxy_pass http://origin.example.com;
            
            # 设置代理请求头
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
```

### 2. CDN缓存规则配置

```json
{
    "rules": [
        {
            "path": "/images/*",
            "ttl": 86400,
            "cache": true
        },
        {
            "path": "/static/*",
            "ttl": 31536000,
            "cache": true
        },
        {
            "path": "/api/*",
            "ttl": 0,
            "cache": false
        },
        {
            "path": "*.html",
            "ttl": 3600,
            "cache": true,
            "purgeable": true
        }
    ]
}
```

---

## 五、CDN优化策略

### 1. 内容优化

#### 图片优化
- **格式转换**：WebP、AVIF格式
- **自适应图片**：根据设备分辨率提供合适尺寸
- **懒加载**：只加载可见区域的图片

#### 代码优化
- **压缩**：gzip、Brotli压缩
- **合并**：合并CSS和JS文件
- **Minify**：去除空白和注释

### 2. HTTPS优化

#### SSL/TLS配置
```nginx
server {
    listen 443 ssl http2;
    server_name cdn.example.com;

    # SSL证书
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    # 启用HTTP/2
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers off;

    # 启用OCSP Stapling
    ssl_stapling on;
    ssl_stapling_verify on;
}
```

### 3. 缓存策略优化

**最佳实践**：
- **静态资源**：长TTL（如1年）+ 版本控制
- **HTML页面**：短TTL（如1小时）+ ETag验证
- **API响应**：根据业务需求设置合理TTL
- **避免缓存敏感数据**：设置`Cache-Control: no-store`

---

## 六、CDN监控与调试

### 1. 常用调试命令

```bash
# 检查CDN缓存状态
curl -I https://cdn.example.com/image.jpg

# 查看缓存命中情况
curl -s -I https://cdn.example.com/style.css | grep X-Cache

# 测试不同节点响应
curl --resolve cdn.example.com:443:192.168.1.100 https://cdn.example.com/

# 清除缓存
curl -X PURGE https://cdn.example.com/image.jpg -H "X-Purge-Token: your-token"
```

### 2. 监控指标

| 指标 | 说明 |
|------|------|
| **缓存命中率** | 缓存命中请求数/总请求数 |
| **回源率** | 回源请求数/总请求数 |
| **响应时间** | 平均响应时间 |
| **带宽使用** | 入站/出站带宽 |
| **错误率** | 错误请求数/总请求数 |

### 3. 日志分析

```bash
# 分析访问日志
cat access.log | awk '{print $7}' | sort | uniq -c | sort -rn | head -10

# 统计缓存命中率
cat access.log | awk '{print $NF}' | grep -c HIT
cat access.log | awk '{print $NF}' | grep -c MISS
```

---

## 七、CDN常见问题

### 1. 缓存更新不及时

**问题描述**：源站更新了内容，但CDN仍然返回旧内容

**排查步骤**：
```bash
# 1. 检查缓存TTL设置
curl -I https://cdn.example.com/resource.js

# 2. 检查是否需要手动清除缓存
curl -X PURGE https://cdn.example.com/resource.js

# 3. 检查URL是否有版本号
# 推荐：https://cdn.example.com/resource.v2.js
```

### 2. HTTPS证书问题

**问题描述**：浏览器提示证书无效

**排查步骤**：
```bash
# 1. 检查证书有效期
openssl x509 -in cert.pem -text -noout | grep Not

# 2. 检查证书链
openssl s_client -connect cdn.example.com:443 -showcerts

# 3. 检查证书配置
nginx -t
```

### 3. 缓存命中率低

**问题描述**：大部分请求都回源，缓存命中率低于50%

**排查步骤**：
```bash
# 1. 分析未命中原因
cat access.log | awk '{print $7, $NF}' | grep MISS | head -20

# 2. 检查TTL设置是否合理
# 静态资源建议设置较长TTL

# 3. 检查缓存键配置
# 是否包含过多变化的参数
```

### 4. 跨域问题

**问题描述**：CDN资源跨域访问被阻止

**解决方案**：
```nginx
add_header Access-Control-Allow-Origin "*";
add_header Access-Control-Allow-Methods "GET, POST, OPTIONS";
add_header Access-Control-Allow-Headers "Content-Type";
```

---

## 八、CDN厂商对比

| 厂商 | 特点 |
|------|------|
| **Cloudflare** | 全球节点多，免费套餐可用 |
| **阿里云CDN** | 国内节点丰富，适合国内业务 |
| **腾讯云CDN** | 与腾讯生态整合好 |
| **Akamai** | 老牌CDN，企业级服务 |
| **Fastly** | 支持实时配置更新 |

---

## 九、CDN安全

### 1. DDoS防护

```nginx
# 限制请求速率
limit_req_zone $binary_remote_addr zone=one:10m rate=10r/s;

server {
    location / {
        limit_req zone=one burst=20;
    }
}
```

### 2. WAF（Web应用防火墙）

**防护功能**：
- SQL注入检测
- XSS攻击防护
- 恶意爬虫识别
- 速率限制

### 3. 访问控制

```nginx
# 限制特定IP访问
allow 192.168.1.0/24;
deny all;

# 限制Referer
valid_referers none blocked example.com *.example.com;
if ($invalid_referer) {
    return 403;
}
```

---

## 十、常见面试题

### 1. CDN的工作原理是什么？

**答案**：
1. 用户发起请求
2. DNS解析到最近的边缘节点
3. 边缘节点检查缓存
4. 缓存命中：直接返回
5. 缓存未命中：回源获取并缓存

### 2. CDN的优势有哪些？

**答案**：
- 降低延迟：内容离用户更近
- 减轻源站压力：减少回源请求
- 提高可用性：多节点冗余
- 增强安全性：DDoS防护、WAF

### 3. 如何配置CDN缓存策略？

**答案**：
- 设置合理的TTL
- 使用版本控制更新资源
- 配置缓存键规则
- 定期清理过期缓存

### 4. 缓存命中率低的原因有哪些？

**答案**：
- TTL设置过短
- 资源URL频繁变化
- 查询参数过多
- 缓存键配置不合理

### 5. CDN如何处理HTTPS请求？

**答案**：
1. 边缘节点终止SSL连接
2. 使用HTTP或HTTPS回源
3. 支持SNI多域名证书
4. 提供OCSP Stapling

### 6. 什么是GSLB？

**答案**：
GSLB（Global Server Load Balancing）是全局负载均衡，根据用户位置、节点负载等因素选择最优边缘节点。

### 7. CDN如何更新缓存？

**答案**：
- TTL过期自动更新
- 主动PURGE清除缓存
- 版本控制（URL带版本号）
- 源站推送更新

### 8. CDN和缓存的区别？

**答案**：
CDN是分布式缓存系统，包含多个边缘节点；缓存可以是本地的、单一的。CDN是缓存的一种实现方式。

### 9. 如何测试CDN是否生效？

**答案**：
- 检查响应头中的`X-Cache`字段
- 使用不同网络访问同一资源
- 清除浏览器缓存后重新访问

### 10. CDN的适用场景有哪些？

**答案**：
- 静态资源分发（图片、CSS、JS）
- 视频流媒体
- 大文件下载
- API响应缓存

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

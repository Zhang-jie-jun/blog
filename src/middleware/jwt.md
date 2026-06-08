# JWT详解

## JWT是什么？
&emsp;&emsp;JWT（JSON Web Token）是一种用于安全传输信息的开放标准（RFC 7519）。它将用户信息编码为JSON对象，通过数字签名保证信息的完整性和真实性。

## JWT的结构

JWT由三个部分组成，用点号(.)分隔：

```
Header.Payload.Signature
```

### Header（头部）

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### Payload（载荷）

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022,
  "exp": 1516239082
}
```

### Signature（签名）

```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret
)
```

## JWT的特点

| 特性 | 说明 |
|------|------|
| **无状态** | 服务端无需存储会话 |
| **自包含** | Token包含所有必要信息 |
| **跨语言** | 支持多种编程语言 |
| **安全** | 数字签名保证完整性 |

## JWT代码示例（Go）

### 安装依赖

```bash
go get github.com/golang-jwt/jwt/v4
```

### 生成Token

```go
package main

import (
    "fmt"
    "time"

    "github.com/golang-jwt/jwt/v4"
)

var secretKey = []byte("my_secret_key")

func GenerateToken(username string) (string, error) {
    // 创建声明
    claims := jwt.MapClaims{
        "username": username,
        "exp":      time.Now().Add(time.Hour * 24).Unix(),
        "iat":      time.Now().Unix(),
    }

    // 创建Token
    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)

    // 签名Token
    return token.SignedString(secretKey)
}
```

### 验证Token

```go
func ValidateToken(tokenString string) (bool, error) {
    token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
        // 验证算法
        if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
            return nil, fmt.Errorf("Unexpected signing method: %v", token.Header["alg"])
        }
        return secretKey, nil
    })

    if err != nil {
        return false, err
    }

    if claims, ok := token.Claims.(jwt.MapClaims); ok && token.Valid {
        fmt.Printf("Username: %s\n", claims["username"])
        return true, nil
    }

    return false, nil
}
```

### 使用自定义Claims

```go
type CustomClaims struct {
    Username string `json:"username"`
    Role     string `json:"role"`
    jwt.StandardClaims
}

func GenerateCustomToken(username, role string) (string, error) {
    claims := CustomClaims{
        Username: username,
        Role:     role,
        StandardClaims: jwt.StandardClaims{
            ExpiresAt: time.Now().Add(time.Hour * 24).Unix(),
            IssuedAt:  time.Now().Unix(),
        },
    }

    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    return token.SignedString(secretKey)
}

func ValidateCustomToken(tokenString string) (*CustomClaims, error) {
    token, err := jwt.ParseWithClaims(tokenString, &CustomClaims{}, func(token *jwt.Token) (interface{}, error) {
        return secretKey, nil
    })

    if err != nil {
        return nil, err
    }

    if claims, ok := token.Claims.(*CustomClaims); ok && token.Valid {
        return claims, nil
    }

    return nil, fmt.Errorf("Invalid token")
}
```

## JWT与Gin集成

```go
func JWTMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        // 获取Token
        tokenString := c.GetHeader("Authorization")
        if tokenString == "" {
            c.AbortWithStatusJSON(401, gin.H{"error": "Unauthorized"})
            return
        }

        // 验证Token
        claims, err := ValidateCustomToken(tokenString)
        if err != nil {
            c.AbortWithStatusJSON(401, gin.H{"error": "Invalid token"})
            return
        }

        // 将用户信息存入上下文
        c.Set("username", claims.Username)
        c.Set("role", claims.Role)
        
        c.Next()
    }
}

// 使用中间件
r.GET("/protected", JWTMiddleware(), func(c *gin.Context) {
    username := c.GetString("username")
    c.JSON(200, gin.H{"message": "Hello " + username})
})
```

## JWT的安全建议

| 建议 | 说明 |
|------|------|
| 使用HTTPS | 防止Token被窃取 |
| 设置过期时间 | 减少Token泄露风险 |
| 使用强密钥 | 密钥长度至少32位 |
| 存储方式 | 前端存储在HttpOnly Cookie |
| 避免敏感信息 | Payload是Base64编码，不是加密 |

## JWT刷新机制

```go
func RefreshToken(oldTokenString string) (string, error) {
    // 解析旧Token（不验证过期）
    token, err := jwt.ParseWithClaims(oldTokenString, &CustomClaims{}, func(token *jwt.Token) (interface{}, error) {
        return secretKey, nil
    })

    if err != nil {
        return "", err
    }

    if claims, ok := token.Claims.(*CustomClaims); ok {
        // 创建新Token，延长过期时间
        claims.ExpiresAt = time.Now().Add(time.Hour * 24).Unix()
        claims.IssuedAt = time.Now().Unix()
        
        newToken := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
        return newToken.SignedString(secretKey)
    }

    return "", fmt.Errorf("Invalid token")
}
```

---

## 一、JWT核心原理

### 1.1 签名算法

**对称加密（HS256/HS512）：**
```go
// 生成签名
HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
```

**非对称加密（RS256/RS512）：**
```go
// 使用私钥签名
RSASHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), privateKey)

// 使用公钥验证
RSASHA256Verify(base64UrlEncode(header) + "." + base64UrlEncode(payload), signature, publicKey)
```

**算法对比：**

| 算法 | 类型 | 安全性 | 适用场景 |
|------|------|--------|----------|
| **HS256** | 对称 | 中 | 内部系统 |
| **HS512** | 对称 | 高 | 内部系统 |
| **RS256** | 非对称 | 高 | 分布式系统 |
| **RS512** | 非对称 | 极高 | 高安全要求 |

### 1.2 Token验证流程

```
客户端请求 → 提取Token → 验证签名 → 检查过期时间 → 检查自定义Claims → 处理请求
```

**验证步骤：**

1. **解析Token结构**：分割Header、Payload、Signature
2. **验证签名**：使用密钥/公钥验证完整性
3. **检查过期时间**：验证`exp`字段
4. **检查签发时间**：验证`iat`字段
5. **检查其他Claims**：根据业务需求验证

### 1.3 Base64URL编码

JWT使用Base64URL编码而非标准Base64：

```
标准Base64: + / =
Base64URL:  - _ (无填充)
```

**编码示例：**
```go
// 标准Base64编码
SGVsbG8gV29ybGQ=

// Base64URL编码
SGVsbG8gV29ybGQ
```

---

## 二、JWT与Session对比

| 特性 | JWT | Session |
|------|-----|---------|
| **存储位置** | 客户端 | 服务端 |
| **状态管理** | 无状态 | 有状态 |
| **扩展性** | 好（水平扩展） | 差（需要共享Session） |
| **性能** | 无需数据库查询 | 需要查询Session |
| **安全性** | 签名验证 | Cookie安全 |
| **跨域** | 容易 | 困难 |
| **Token过期** | 自动过期 | 需要主动清理 |

---

## 三、JWT常见问题分析与排查

### 问题1：Token验证失败

**现象：**
```
token is invalid
```

**排查步骤：**

```go
// 1. 检查Token格式
tokenString := c.GetHeader("Authorization")
if !strings.HasPrefix(tokenString, "Bearer ") {
    return errors.New("invalid token format")
}
tokenString = strings.TrimPrefix(tokenString, "Bearer ")

// 2. 检查签名算法
token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
    if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
        return nil, fmt.Errorf("unexpected signing method: %v", token.Header["alg"])
    }
    return secretKey, nil
})

// 3. 检查Token是否过期
if claims, ok := token.Claims.(jwt.MapClaims); ok {
    if float64(time.Now().Unix()) > claims["exp"].(float64) {
        return errors.New("token expired")
    }
}
```

**解决方案：**
1. 确保Token格式正确（带Bearer前缀）
2. 使用正确的签名算法
3. 检查Token是否过期

---

### 问题2：Token被篡改

**现象：**
```
signature is invalid
```

**排查步骤：**

```bash
# 手动验证签名
echo "header.payload.signature" | cut -d'.' -f1,2 | base64 -d | openssl dgst -sha256 -hmac "secret"
```

**解决方案：**
1. 使用强密钥（至少32位）
2. 定期轮换密钥
3. 使用HTTPS传输

---

### 问题3：Token泄露

**现象：**
用户Token被窃取，导致账户被盗。

**排查步骤：**

```go
// 检查Token存储方式
// 前端应使用HttpOnly Cookie
// 或使用localStorage配合HTTPS
```

**解决方案：**
1. 使用HttpOnly Cookie存储Token
2. 设置Cookie的Secure和SameSite属性
3. 实现Token黑名单机制
4. 设置较短的过期时间

---

### 问题4：Token刷新问题

**现象：**
刷新Token时失败或Token过期无法刷新。

**排查步骤：**

```go
func RefreshToken(tokenString string) (string, error) {
    // 解析时不验证过期
    token, err := jwt.ParseWithClaims(tokenString, &CustomClaims{}, func(token *jwt.Token) (interface{}, error) {
        return secretKey, nil
    })
    
    if err != nil {
        return "", err
    }
    
    // 检查是否可以刷新（过期时间在合理范围内）
    if claims, ok := token.Claims.(*CustomClaims); ok {
        if time.Now().Unix() > claims.ExpiresAt+3600 { // 允许过期后1小时内刷新
            return "", errors.New("token cannot be refreshed")
        }
        // 生成新Token
    }
}
```

**解决方案：**
1. 设置合理的刷新窗口
2. 使用单独的Refresh Token
3. 实现Refresh Token的过期和轮换

---

### 问题5：Claims解析失败

**现象：**
无法正确解析自定义Claims。

**排查步骤：**

```go
// 检查Claims定义
type CustomClaims struct {
    Username string `json:"username"`  // 注意json标签
    Role     string `json:"role"`
    jwt.StandardClaims
}

// 检查解析方式
token, err := jwt.ParseWithClaims(tokenString, &CustomClaims{}, keyFunc)
if err != nil {
    return nil, err
}

claims, ok := token.Claims.(*CustomClaims)
if !ok {
    return nil, errors.New("invalid claims type")
}
```

**解决方案：**
1. 确保Claims结构正确
2. 使用`ParseWithClaims`而非`Parse`
3. 检查JSON标签是否正确

---

### 问题6：密钥管理问题

**现象：**
密钥泄露或密钥轮换导致验证失败。

**排查步骤：**

```go
// 检查密钥配置
var secretKey = []byte(os.Getenv("JWT_SECRET"))

// 确保密钥不为空
if len(secretKey) == 0 {
    panic("JWT_SECRET environment variable not set")
}
```

**解决方案：**
1. 使用环境变量存储密钥
2. 定期轮换密钥
3. 使用密钥管理服务（如Vault）

---

### 问题7：跨域认证问题

**现象：**
前端无法携带Token进行跨域请求。

**排查步骤：**

```go
// 检查CORS配置
func CORSMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        c.Header("Access-Control-Allow-Origin", "*")
        c.Header("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE, OPTIONS")
        c.Header("Access-Control-Allow-Headers", "Content-Type, Authorization")
        
        if c.Request.Method == "OPTIONS" {
            c.AbortWithStatus(204)
            return
        }
        
        c.Next()
    }
}
```

**解决方案：**
1. 配置正确的CORS响应头
2. 允许Authorization头
3. 处理OPTIONS预检请求

---

### 问题8：Token过期时间设置不合理

**现象：**
Token过期太快或太慢。

**排查步骤：**

```go
// 检查过期时间设置
claims := jwt.MapClaims{
    "exp": time.Now().Add(time.Hour * 24).Unix(), // 24小时过期
}
```

**解决方案：**
1. 根据业务场景设置合理过期时间
2. 短Token + Refresh Token模式
3. 敏感操作强制重新登录

---

### 问题9：Token未包含必要信息

**现象：**
Token中缺少用户ID、角色等信息。

**排查步骤：**

```go
// 检查Claims定义
type CustomClaims struct {
    UserID   uint   `json:"user_id"`
    Username string `json:"username"`
    Role     string `json:"role"`
    jwt.StandardClaims
}
```

**解决方案：**
1. 在Payload中包含必要的用户信息
2. 避免存储敏感信息（如密码）
3. 使用最小权限原则

---

### 问题10：JWT与CSRF攻击

**现象：**
跨站请求伪造攻击风险。

**排查步骤：**

```go
// 检查是否使用Cookie存储Token
// 如果使用Cookie，需要CSRF防护
func CSRFMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        csrfToken := c.GetHeader("X-CSRF-Token")
        sessionCSRFToken := c.GetCookie("csrf_token")
        
        if csrfToken != sessionCSRFToken {
            c.AbortWithStatusJSON(403, gin.H{"error": "CSRF token mismatch"})
            return
        }
        
        c.Next()
    }
}
```

**解决方案：**
1. 如果使用Cookie存储Token，实现CSRF防护
2. 使用Token存储在Authorization头
3. 设置SameSite=Strict

---

## 四、JWT面试题

### 基础问题

**Q1：JWT是什么？由哪几部分组成？**

**A1：**
JWT（JSON Web Token）是一种安全传输信息的开放标准。

由三部分组成：
- **Header**：包含算法和类型
- **Payload**：包含声明信息
- **Signature**：数字签名

格式：`Header.Payload.Signature`

---

**Q2：JWT的签名算法有哪些？区别是什么？**

**A2：**

| 算法 | 类型 | 说明 |
|------|------|------|
| **HS256** | 对称 | 使用相同密钥签名和验证 |
| **HS512** | 对称 | 更安全的对称加密 |
| **RS256** | 非对称 | 使用私钥签名，公钥验证 |
| **RS512** | 非对称 | 更安全的非对称加密 |

---

**Q3：JWT和Session有什么区别？**

**A3：**

| 特性 | JWT | Session |
|------|-----|---------|
| **存储** | 客户端 | 服务端 |
| **状态** | 无状态 | 有状态 |
| **扩展** | 易水平扩展 | 需要Session共享 |
| **性能** | 无需DB查询 | 需要查询Session |

---

**Q4：JWT的Payload可以包含哪些信息？**

**A4：**

**标准Claims：**
- `iss`（issuer）：签发者
- `sub`（subject）：主题
- `aud`（audience）：受众
- `exp`（expiration time）：过期时间
- `nbf`（not before）：生效时间
- `iat`（issued at）：签发时间
- `jti`（JWT ID）：唯一标识

**自定义Claims：**
- 用户ID、用户名、角色等业务信息

---

**Q5：为什么JWT的Payload不是加密的？**

**A5：**
JWT的Payload是Base64URL编码，不是加密。原因：
- **签名保证完整性**：签名验证确保数据未被篡改
- **Payload是公开信息**：设计上Payload就是可读的
- **如需加密**：可以使用JWE（JSON Web Encryption）

---

**Q6：如何验证JWT的有效性？**

**A6：**
验证步骤：
1. 解析Token结构
2. 验证签名（使用密钥/公钥）
3. 检查过期时间（`exp`）
4. 检查生效时间（`nbf`）
5. 验证其他自定义Claims

---

**Q7：JWT如何实现Token刷新？**

**A7：**
常见方案：
1. **短Token + Refresh Token**：Access Token短过期，Refresh Token长过期
2. **过期时间窗口**：允许Token过期后一段时间内刷新
3. **黑名单机制**：防止刷新后的旧Token被滥用

---

**Q8：JWT的安全注意事项有哪些？**

**A8：**
- 使用HTTPS传输
- 设置合理的过期时间
- 使用强密钥
- 避免存储敏感信息
- 使用HttpOnly Cookie存储
- 实现Token黑名单

---

**Q9：JWT如何实现注销登录？**

**A9：**
JWT本身无法主动注销，需要额外机制：
1. **Token黑名单**：维护已注销Token列表
2. **短过期时间**：让Token尽快过期
3. **双Token机制**：Refresh Token存储在服务端，可失效

---

**Q10：什么是JWE？与JWT有什么区别？**

**A10：**
JWE（JSON Web Encryption）是JWT的加密版本。

| 特性 | JWT（JWS） | JWE |
|------|-----------|-----|
| **Payload** | Base64编码（可读） | 加密（不可读） |
| **用途** | 完整性验证 | 保密性 + 完整性 |
| **复杂度** | 简单 | 复杂 |

---

### 复杂场景问题

**Q11：如何设计一个安全的JWT认证系统？**

**A11：**

**架构设计：**

```
                    ┌──────────────────────────────────────┐
                    │              客户端                   │
                    └──────────────────┬───────────────────┘
                                       │
                                       ▼
                    ┌──────────────────────────────────────┐
                    │              API网关                 │
                    │      (认证、限流、日志)              │
                    └──────────────────┬───────────────────┘
                                       │
        ┌──────────────────────────────┼──────────────────────────────┐
        ▼                              ▼                              ▼
┌──────────────┐              ┌──────────────┐              ┌──────────────┐
│   认证服务   │              │   用户服务   │              │   订单服务   │
│ (JWT生成/验证)│              │   (业务逻辑) │              │   (业务逻辑) │
└──────────────┘              └──────────────┘              └──────────────┘
        │                              │                              │
        └──────────────────────────────┼──────────────────────────────┘
                                       │
                                       ▼
                    ┌──────────────────────────────────────┐
                    │           Redis缓存                  │
                    │   (Token黑名单、Refresh Token)       │
                    └──────────────────────────────────────┘
```

**关键组件：**
1. **双Token机制**：Access Token（15分钟）+ Refresh Token（7天）
2. **Token黑名单**：Redis存储已注销Token
3. **密钥轮换**：定期更换签名密钥
4. **监控告警**：异常登录检测

**安全措施：**
- HTTPS传输
- HttpOnly + Secure Cookie
- CSRF防护
- 多因素认证

---

**Q12：如何处理JWT的密钥轮换？**

**A12：**

**方案：多密钥并行验证**

```go
var validKeys = []string{"key1", "key2", "key3"}

func validateToken(tokenString string) (*CustomClaims, error) {
    for _, key := range validKeys {
        token, err := jwt.ParseWithClaims(tokenString, &CustomClaims{}, func(token *jwt.Token) (interface{}, error) {
            return []byte(key), nil
        })
        
        if err == nil && token.Valid {
            return token.Claims.(*CustomClaims), nil
        }
    }
    return nil, errors.New("invalid token")
}
```

**轮换流程：**
1. 生成新密钥并添加到验证列表
2. 一段时间后（如24小时）删除旧密钥
3. 监控验证失败率

---

**Q13：如何实现分布式环境下的Token黑名单？**

**A13：**

**方案：Redis分布式黑名单**

```go
type TokenBlacklist struct {
    client *redis.Client
}

func (tb *TokenBlacklist) Add(token string, expiresAt int64) error {
    // 计算剩余过期时间
    ttl := time.Until(time.Unix(expiresAt, 0))
    return tb.client.Set(context.Background(), "blacklist:"+token, "1", ttl).Err()
}

func (tb *TokenBlacklist) Contains(token string) (bool, error) {
    exists, err := tb.client.Exists(context.Background(), "blacklist:"+token).Result()
    return exists == 1, err
}
```

**优化策略：**
- 使用Redis集群保证高可用
- 设置合理的TTL自动清理
- 使用布隆过滤器减少内存占用

---

**Q14：如何处理JWT的性能问题？**

**A14：**

**性能优化策略：**

1. **减少Payload大小**：
```go
// 不好：包含大量信息
claims := jwt.MapClaims{
    "user_id": 123,
    "name": "John Doe",
    "email": "john@example.com",
    "role": "admin",
    "permissions": []string{"read", "write", "delete"},
}

// 好：只包含必要信息
claims := jwt.MapClaims{
    "sub": "123",
    "role": "admin",
}
```

2. **缓存验证结果**：
```go
var tokenCache = sync.Map{}

func validateToken(tokenString string) (*CustomClaims, error) {
    if cached, ok := tokenCache.Load(tokenString); ok {
        return cached.(*CustomClaims), nil
    }
    
    // 验证逻辑...
    
    tokenCache.Store(tokenString, claims)
    return claims, nil
}
```

3. **使用高效算法**：优先使用HS256而非RS256

---

**Q15：如何设计多租户系统的JWT认证？**

**A15：**

**方案：租户级密钥管理**

```go
type TenantKeyManager struct {
    keys map[string][]byte // tenantID -> secretKey
}

func (m *TenantKeyManager) GetKey(tenantID string) ([]byte, error) {
    key, ok := m.keys[tenantID]
    if !ok {
        return nil, errors.New("tenant not found")
    }
    return key, nil
}

func validateToken(tokenString, tenantID string) (*CustomClaims, error) {
    manager := NewTenantKeyManager()
    key, err := manager.GetKey(tenantID)
    if err != nil {
        return nil, err
    }
    
    token, err := jwt.ParseWithClaims(tokenString, &CustomClaims{}, func(token *jwt.Token) (interface{}, error) {
        return key, nil
    })
    
    if err != nil {
        return nil, err
    }
    
    claims, ok := token.Claims.(*CustomClaims)
    if !ok || !token.Valid {
        return nil, errors.New("invalid token")
    }
    
    // 验证租户ID
    if claims.TenantID != tenantID {
        return nil, errors.New("tenant mismatch")
    }
    
    return claims, nil
}
```

---

**Q16：如何实现JWT的权限控制？**

**A16：**

**方案：基于角色的访问控制（RBAC）**

```go
func RequireRole(role string) gin.HandlerFunc {
    return func(c *gin.Context) {
        claims, exists := c.Get("claims")
        if !exists {
            c.AbortWithStatusJSON(401, gin.H{"error": "Unauthorized"})
            return
        }
        
        customClaims := claims.(*CustomClaims)
        if customClaims.Role != role {
            c.AbortWithStatusJSON(403, gin.H{"error": "Forbidden"})
            return
        }
        
        c.Next()
    }
}

// 使用
r.GET("/admin", JWTMiddleware(), RequireRole("admin"), adminHandler)
```

**细粒度权限控制：**
```go
func RequirePermission(permission string) gin.HandlerFunc {
    return func(c *gin.Context) {
        claims := c.MustGet("claims").(*CustomClaims)
        
        // 检查权限列表
        for _, p := range claims.Permissions {
            if p == permission {
                c.Next()
                return
            }
        }
        
        c.AbortWithStatusJSON(403, gin.H{"error": "Forbidden"})
    }
}
```

---

**Q17：如何处理JWT的时区问题？**

**A17：**

**问题分析：**
JWT的`exp`和`iat`使用Unix时间戳（UTC），时区处理不当会导致验证失败。

**解决方案：**

```go
// 正确：使用UTC时间
claims := jwt.MapClaims{
    "exp": time.Now().UTC().Add(time.Hour * 24).Unix(),
    "iat": time.Now().UTC().Unix(),
}

// 验证时也使用UTC
func validateExpiry(exp int64) bool {
    return time.Now().UTC().Unix() <= exp
}
```

**注意事项：**
- 始终使用UTC时间
- 避免使用本地时间
- 在前端显示时转换为本地时区

---

**Q18：如何实现JWT的多设备登录管理？**

**A18：**

**方案：设备级Token管理**

```go
type DeviceClaims struct {
    DeviceID string `json:"device_id"`
    CustomClaims
}

func GenerateDeviceToken(userID, deviceID string) (string, error) {
    claims := DeviceClaims{
        DeviceID: deviceID,
        CustomClaims: CustomClaims{
            UserID: userID,
            StandardClaims: jwt.StandardClaims{
                ExpiresAt: time.Now().Add(time.Hour * 24 * 30).Unix(),
            },
        },
    }
    
    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    return token.SignedString(secretKey)
}

// 管理用户设备
func RevokeDevice(userID, deviceID string) error {
    // 将deviceID添加到用户的黑名单
    return redisClient.SAdd(context.Background(), "user:"+userID+":devices:blocked", deviceID).Err()
}
```

**登录流程：**
1. 用户登录时生成设备ID
2. 将设备信息存储到Redis
3. 验证时检查设备是否被拉黑

---

**Q19：如何实现JWT的审计日志？**

**A19：**

**方案：拦截器记录Token使用日志**

```go
func AuditMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        claims, exists := c.Get("claims")
        if !exists {
            c.Next()
            return
        }
        
        customClaims := claims.(*CustomClaims)
        
        // 记录审计日志
        logEntry := AuditLog{
            UserID:    customClaims.UserID,
            Timestamp: time.Now(),
            IP:        c.ClientIP(),
            Method:    c.Request.Method,
            Path:      c.Request.URL.Path,
            UserAgent: c.Request.UserAgent(),
        }
        
        // 异步写入日志存储
        go saveAuditLog(logEntry)
        
        c.Next()
    }
}
```

**审计日志内容：**
- 用户ID
- 时间戳
- 请求IP
- 请求方法和路径
- 用户代理
- 操作结果

---

**Q20：如何处理JWT与OAuth2的集成？**

**A20：**

**方案：OAuth2 + JWT组合使用**

```
                    OAuth2流程                           JWT流程
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  授权码获取 │ ──► │  Access Token │ ──► │  JWT验证    │
│  (OAuth2)   │     │   (JWT格式)   │     │  (业务逻辑) │
└─────────────┘     └─────────────┘     └─────────────┘
```

**实现示例：**

```go
// OAuth2回调处理
func OAuthCallback(c *gin.Context) {
    // 1. 获取授权码
    code := c.Query("code")
    
    // 2. 换取Access Token（JWT格式）
    token, err := oauthClient.Exchange(context.Background(), code)
    if err != nil {
        c.AbortWithStatusJSON(400, gin.H{"error": err.Error()})
        return
    }
    
    // 3. 验证JWT Token
    claims, err := ValidateToken(token.AccessToken)
    if err != nil {
        c.AbortWithStatusJSON(401, gin.H{"error": err.Error()})
        return
    }
    
    // 4. 创建会话
    c.Set("claims", claims)
    c.Redirect(302, "/dashboard")
}
```

**优势：**
- OAuth2提供授权流程
- JWT提供无状态认证
- 支持第三方登录

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

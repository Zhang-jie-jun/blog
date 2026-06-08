# GORM详解

## GORM是什么？
&emsp;&emsp;GORM是Go语言的ORM（对象关系映射）库，提供了优雅的API来操作数据库。它支持MySQL、PostgreSQL、SQLite等多种数据库。

## GORM的特点

| 特性 | 说明 |
|------|------|
| **全功能ORM** | 支持CRUD、关联、事务等 |
| **自动迁移** | 自动创建/更新表结构 |
| **关联查询** | 支持预加载、嵌套关联 |
| **钩子函数** | 支持Before/After钩子 |
| **事务支持** | 完整的事务管理 |

---

## 一、GORM核心原理

### 1.1 SQL生成机制

**SQL生成流程：**

```
模型定义 → 解析结构体标签 → 构建SQL语句 → 执行并映射结果
```

**核心组件：**

```go
// Schema解析器
type Schema struct {
    TableName  string
    Fields     []*Field
    Relations  []*Relation
    PrimaryKey *Field
}

// SQL生成器
func (db *DB) buildSQL(opts *Query) (string, []interface{}) {
    // 1. 构建SELECT子句
    // 2. 构建FROM子句
    // 3. 构建WHERE条件
    // 4. 构建ORDER BY
    // 5. 构建LIMIT/OFFSET
}
```

### 1.2 连接池管理

**连接池配置：**

```go
// 配置连接池
db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{
    PrepareStmt:            true,  // 预编译语句缓存
    SkipDefaultTransaction: true,  // 禁用默认事务
})

// 获取底层sql.DB配置连接池
sqlDB, _ := db.DB()
sqlDB.SetMaxIdleConns(10)    // 最大空闲连接数
sqlDB.SetMaxOpenConns(100)   // 最大打开连接数
sqlDB.SetConnMaxLifetime(time.Hour)  // 连接最大存活时间
```

**连接池工作流程：**

```
请求 → 获取连接 → 执行SQL → 归还连接
      ↓
   连接池满 → 等待/拒绝
```

### 1.3 事务管理

**事务实现原理：**

```go
func (db *DB) Begin(opts ...*TxOptions) *DB {
    // 1. 开启事务
    tx := db.Session(&Session{NewDB: true})
    tx.Statement.ConnPool = &txPool{}
    
    // 2. 执行BEGIN
    tx.Exec("BEGIN")
    
    return tx
}

func (tx *DB) Commit() error {
    // 执行COMMIT
    return tx.Exec("COMMIT").Error
}

func (tx *DB) Rollback() error {
    // 执行ROLLBACK
    return tx.Exec("ROLLBACK").Error
}
```

**事务传播：**

```go
// 嵌套事务（SavePoint）
tx := db.Begin()
tx.Create(&user1)

// 嵌套事务
tx2 := tx.Begin()
tx2.Create(&user2)
tx2.Commit()  // 创建SavePoint并释放

tx.Commit()
```

### 1.4 模型定义与标签

**标签解析：**

```go
type User struct {
    gorm.Model
    Name     string `gorm:"size:100;not null;column:username"`
    Email    string `gorm:"unique;index:idx_email"`
    Age      int    `gorm:"default:0"`
    Password string `gorm:"-"`  // 忽略此字段
}
```

**标签含义：**

| 标签 | 说明 |
|------|------|
| `column` | 指定数据库列名 |
| `size` | 指定字段长度 |
| `not null` | 非空约束 |
| `unique` | 唯一约束 |
| `index` | 创建索引 |
| `default` | 默认值 |
| `-` | 忽略字段 |

---

## 二、GORM高级特性

### 2.1 关联查询

**一对一关联：**

```go
type User struct {
    gorm.Model
    Profile Profile `gorm:"foreignKey:UserID"`
}

type Profile struct {
    gorm.Model
    UserID uint
    Bio    string
}

// 预加载关联
var user User
db.Preload("Profile").First(&user)
```

**一对多关联：**

```go
type User struct {
    gorm.Model
    Posts []Post `gorm:"foreignKey:UserID"`
}

type Post struct {
    gorm.Model
    UserID uint
    Title  string
}

// 预加载多个关联
db.Preload("Posts", "status = ?", "published").First(&user)
```

**多对多关联：**

```go
type User struct {
    gorm.Model
    Roles []Role `gorm:"many2many:user_roles;"`
}

type Role struct {
    gorm.Model
    Name string
}

// 自定义中间表
type UserRole struct {
    UserID uint
    RoleID uint
    CreatedAt time.Time
}

db.Preload("Roles").First(&user)
```

### 2.2 钩子函数

**钩子执行顺序：**

```
创建：BeforeCreate → 创建记录 → AfterCreate
更新：BeforeUpdate → 更新记录 → AfterUpdate
删除：BeforeDelete → 删除记录 → AfterDelete
查询：AfterFind
```

**钩子实现：**

```go
func (u *User) BeforeCreate(tx *gorm.DB) (err error) {
    // 密码加密
    u.Password = encryptPassword(u.Password)
    // 生成UUID
    u.ID = uuid.New()
    return
}

func (u *User) AfterCreate(tx *gorm.DB) (err error) {
    // 发送欢迎邮件
    go sendWelcomeEmail(u.Email)
    return
}

func (u *User) BeforeUpdate(tx *gorm.DB) (err error) {
    // 记录更新时间
    u.UpdatedAt = time.Now()
    return
}
```

### 2.3 批量操作

**批量插入：**

```go
users := []User{
    {Name: "John", Email: "john@example.com"},
    {Name: "Jane", Email: "jane@example.com"},
}

// 批量插入（单条SQL）
db.Create(&users)

// 分批插入
db.CreateInBatches(users, 100)
```

**批量更新：**

```go
// 批量更新相同值
db.Model(&User{}).Where("status = ?", "inactive").Update("status", "active")

// 批量更新不同值
users := []User{
    {ID: 1, Name: "John Updated"},
    {ID: 2, Name: "Jane Updated"},
}
db.Save(&users)
```

---

## 三、GORM常见问题分析与排查

### 问题1：N+1查询问题

**现象：**
查询列表时，每个记录都会触发额外的关联查询。

**排查步骤：**

```go
// 问题代码（N+1查询）
var users []User
db.Find(&users)  // 1次查询
for _, user := range users {
    db.Model(&user).Association("Posts").Find(&user.Posts)  // N次查询
}

// 使用EXPLAIN分析
db.Debug().Find(&users)  // 查看SQL日志
```

**解决方案：**

```go
// 使用Preload预加载
var users []User
db.Preload("Posts").Find(&users)  // 2次查询

// 嵌套预加载
db.Preload("Posts.Comments").Find(&users)

// 条件预加载
db.Preload("Posts", "status = ?", "published").Find(&users)
```

---

### 问题2：软删除冲突

**现象：**
查询时无法获取已软删除的数据，或软删除标记不生效。

**排查步骤：**

```go
// 检查模型是否包含gorm.Model
type User struct {
    gorm.Model  // 包含DeletedAt字段
    Name string
}

// 检查查询是否包含软删除条件
db.Debug().Find(&users)  // WHERE deleted_at IS NULL
```

**解决方案：**

```go
// 查询包含软删除的数据
db.Unscoped().Find(&users)

// 永久删除（绕过软删除）
db.Unscoped().Delete(&user)

// 恢复软删除数据
db.Unscoped().Model(&user).Update("deleted_at", nil)
```

---

### 问题3：事务回滚失败

**现象：**
事务中的错误没有正确回滚。

**排查步骤：**

```go
tx := db.Begin()
defer func() {
    if r := recover(); r != nil {
        tx.Rollback()
    }
}()

if err := tx.Create(&user).Error; err != nil {
    tx.Rollback()  // 需要显式回滚
    return
}

// 如果不调用Rollback，即使出错事务也不会自动回滚
tx.Commit()
```

**解决方案：**

```go
// 正确的事务处理
tx := db.Begin()
defer func() {
    if r := recover(); r != nil {
        tx.Rollback()
    }
}()

if err := tx.Error; err != nil {
    // 事务开始失败
    return
}

if err := tx.Create(&user).Error; err != nil {
    tx.Rollback()
    return
}

tx.Commit()
```

---

### 问题4：预加载失效

**现象：**
使用Preload后关联数据仍为空。

**排查步骤：**

```go
// 检查外键是否正确
type Post struct {
    gorm.Model
    UserID uint  // 外键必须正确命名
    Title  string
}

// 检查关联定义
type User struct {
    gorm.Model
    Posts []Post `gorm:"foreignKey:UserID"`  // 指定外键
}

// 检查查询条件
db.Preload("Posts").Where("id = ?", 1).First(&user)
```

**解决方案：**

```go
// 确保外键正确
type Post struct {
    gorm.Model
    UserID uint  // 必须与关联字段匹配
    Title  string
}

// 显式指定外键
type User struct {
    gorm.Model
    Posts []Post `gorm:"foreignKey:UserID;references:ID"`
}

// 使用Clauses强制预加载
db.Preload("Posts", db.Where("status = ?", "published")).Find(&users)
```

---

### 问题5：连接池耗尽

**现象：**
应用无法获取数据库连接，报连接超时。

**排查步骤：**

```go
// 检查连接池配置
sqlDB, _ := db.DB()
fmt.Println("Open connections:", sqlDB.Stats().OpenConnections)
fmt.Println("Max open connections:", sqlDB.Stats().MaxOpenConnections)

// 检查是否有连接泄漏
// 查看慢查询日志
```

**解决方案：**

```go
// 正确配置连接池
sqlDB, _ := db.DB()
sqlDB.SetMaxIdleConns(20)
sqlDB.SetMaxOpenConns(100)
sqlDB.SetConnMaxLifetime(time.Minute * 30)
sqlDB.SetConnMaxIdleTime(time.Minute * 10)

// 使用defer确保连接归还
func queryUser(db *gorm.DB, id uint) (*User, error) {
    var user User
    err := db.First(&user, id).Error
    return &user, err
}
```

---

### 问题6：查询性能慢

**现象：**
查询响应时间过长。

**排查步骤：**

```go
// 启用Debug模式查看SQL
db.Debug().Find(&users)

// 使用EXPLAIN分析执行计划
db.Raw("EXPLAIN SELECT * FROM users WHERE name = ?", "John").Scan(&result)

// 检查索引
db.Exec("SHOW INDEX FROM users")
```

**解决方案：**

```go
// 添加索引
type User struct {
    gorm.Model
    Name  string `gorm:"index"`
    Email string `gorm:"uniqueIndex"`
}

// 只查询需要的字段
db.Select("id", "name").Find(&users)

// 使用JOIN代替子查询
db.Joins("JOIN posts ON users.id = posts.user_id").Find(&users)
```

---

### 问题7：自动迁移失败

**现象：**
AutoMigrate无法正确创建或更新表结构。

**排查步骤：**

```go
// 检查模型定义
type User struct {
    gorm.Model
    Name string `gorm:"size:100"`
}

// 查看迁移日志
db.Debug().AutoMigrate(&User{})

// 检查数据库权限
// 确保用户有CREATE/ALTER权限
```

**解决方案：**

```go
// 确保模型字段可访问（首字母大写）
type User struct {
    gorm.Model
    Name string `gorm:"size:100"`  // Name而非name
}

// 使用Migrator手动迁移
db.Migrator().CreateTable(&User{})
db.Migrator().AddColumn(&User{}, "Age")

// 处理复杂迁移
db.Exec("ALTER TABLE users ADD COLUMN age INT DEFAULT 0")
```

---

### 问题8：钩子函数不执行

**现象：**
定义的BeforeCreate/AfterCreate等钩子没有被调用。

**排查步骤：**

```go
// 检查钩子函数签名
func (u *User) BeforeCreate(tx *gorm.DB) (err error) {  // 正确
    return
}

// 错误示例
func (u User) BeforeCreate(tx *gorm.DB) {  // 没有指针接收者
    return
}

// 检查是否使用了正确的方法
db.Create(&user)  // 会触发钩子
db.Exec("INSERT INTO users ...")  // 不会触发钩子
```

**解决方案：**

```go
// 使用指针接收者
func (u *User) BeforeCreate(tx *gorm.DB) (err error) {
    u.CreatedAt = time.Now()
    return
}

// 使用GORM方法而非原生SQL
db.Create(&user)  // 触发钩子
db.Save(&user)    // 触发钩子
```

---

### 问题9：并发安全问题

**现象：**
高并发下数据不一致或panic。

**排查步骤：**

```go
// 检查共享变量
var counter int

func incrementCounter(db *gorm.DB) {
    var count int
    db.Model(&User{}).Count(&count)
    counter = count  // 竞态条件
}

// 检查数据库操作
func updateUser(db *gorm.DB, id uint, name string) {
    db.Model(&User{}).Where("id = ?", id).Update("name", name)
}
```

**解决方案：**

```go
// 使用数据库原子操作
db.Model(&User{}).Where("id = ?", id).UpdateColumn("views", gorm.Expr("views + 1"))

// 使用事务保证一致性
tx := db.Begin()
defer tx.Rollback()

// 先锁定记录
var user User
tx.Clauses(clause.Locking{Strength: "UPDATE"}).First(&user, id)

// 更新
user.Name = "New Name"
tx.Save(&user)

tx.Commit()
```

---

### 问题10：关联保存失败

**现象：**
保存关联数据时，关联表没有被更新。

**排查步骤：**

```go
// 检查关联定义
type User struct {
    gorm.Model
    Posts []Post
}

// 检查保存方式
user := User{Name: "John"}
user.Posts = []Post{{Title: "Post 1"}}

db.Create(&user)  // 只保存User，不保存Posts
```

**解决方案：**

```go
// 使用Association保存
db.Create(&user)
db.Model(&user).Association("Posts").Append(&user.Posts)

// 使用Select指定要保存的关联
db.Select("Posts").Create(&user)

// 使用Preload加载后保存
db.Preload("Posts").First(&user)
user.Posts[0].Title = "Updated"
db.Save(&user)
```

---

## 四、GORM面试题

### 基础问题

**Q1：GORM是什么？有什么特点？**

**A1：**
GORM是Go语言的ORM库，特点：
- 全功能ORM，支持CRUD、关联、事务
- 自动迁移，自动创建/更新表结构
- 支持多种数据库（MySQL、PostgreSQL、SQLite等）
- 钩子函数支持
- 链式API设计

---

**Q2：GORM的模型定义需要注意什么？**

**A2：**
- 结构体字段首字母必须大写（可导出）
- 使用gorm.Model嵌入基础字段
- 使用标签定义约束和映射关系
- 主键默认是ID字段

---

**Q3：GORM如何实现CRUD操作？**

**A3：**
- **Create**：`db.Create(&user)`
- **Read**：`db.First(&user, id)`、`db.Find(&users)`
- **Update**：`db.Model(&user).Update("name", "New")`
- **Delete**：`db.Delete(&user)`

---

**Q4：GORM的事务如何使用？**

**A4：**

```go
tx := db.Begin()
defer func() {
    if r := recover(); r != nil {
        tx.Rollback()
    }
}()

tx.Create(&user)
tx.Create(&post)

tx.Commit()
```

---

**Q5：GORM的钩子函数有哪些？执行顺序是什么？**

**A5：**
- `BeforeCreate` → `AfterCreate`
- `BeforeUpdate` → `AfterUpdate`
- `BeforeDelete` → `AfterDelete`
- `AfterFind`

---

**Q6：GORM如何处理关联查询？**

**A6：**
- 使用`Preload`预加载关联数据
- 支持一对一、一对多、多对多关联
- 使用`Association`方法管理关联

---

**Q7：GORM的自动迁移是什么？如何使用？**

**A7：**
自动迁移会根据模型定义自动创建或更新表结构。

```go
db.AutoMigrate(&User{}, &Post{})
```

---

**Q8：GORM如何执行原生SQL？**

**A8：**

```go
// 执行SQL
db.Exec("UPDATE users SET name = ? WHERE id = ?", "New", 1)

// 查询SQL
db.Raw("SELECT * FROM users WHERE age > ?", 18).Scan(&users)
```

---

**Q9：GORM的软删除如何实现？**

**A9：**
嵌入`gorm.Model`后自动支持软删除，删除时设置`deleted_at`字段。

```go
db.Delete(&user)  // 软删除
db.Unscoped().Delete(&user)  // 物理删除
```

---

**Q10：GORM如何配置连接池？**

**A10：**

```go
sqlDB, _ := db.DB()
sqlDB.SetMaxIdleConns(10)
sqlDB.SetMaxOpenConns(100)
sqlDB.SetConnMaxLifetime(time.Hour)
```

---

### 复杂场景问题

**Q11：如何优化GORM的查询性能？**

**A11：**

**优化策略：**

1. **添加索引：**
```go
type User struct {
    gorm.Model
    Email string `gorm:"uniqueIndex"`
}
```

2. **避免N+1查询：**
```go
db.Preload("Posts").Find(&users)
```

3. **只查询需要的字段：**
```go
db.Select("id", "name").Find(&users)
```

4. **使用JOIN优化查询：**
```go
db.Joins("JOIN posts ON users.id = posts.user_id").Find(&users)
```

5. **批量操作：**
```go
db.CreateInBatches(users, 100)
```

---

**Q12：如何设计GORM的多租户架构？**

**A12：**

**方案一：共享数据库，隔离Schema**

```go
type Tenant struct {
    ID   uint
    Name string
}

func GetTenantDB(tenantID uint) (*gorm.DB, error) {
    dsn := fmt.Sprintf("user:pass@tcp(host:3306)/tenant_%d", tenantID)
    return gorm.Open(mysql.Open(dsn), &gorm.Config{})
}
```

**方案二：共享数据库和表，租户字段隔离**

```go
type User struct {
    gorm.Model
    TenantID uint
    Name     string
}

// 查询时过滤租户
db.Where("tenant_id = ?", tenantID).Find(&users)
```

**方案三：使用表前缀**

```go
db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{
    NamingStrategy: schema.NamingStrategy{
        TablePrefix: "tenant_1_",
    },
})
```

---

**Q13：如何处理GORM的分布式事务？**

**A13：**

**方案一：XA事务**

```go
// 使用两阶段提交
tx, err := db.BeginTx(ctx, &sql.TxOptions{Isolation: sql.LevelSerializable})
if err != nil {
    return
}

// 跨库操作
tx.Exec("INSERT INTO db1.users ...")
tx.Exec("INSERT INTO db2.orders ...")

tx.Commit()
```

**方案二：Saga模式**

```go
type OrderSaga struct {
    Steps []SagaStep
}

type SagaStep struct {
    Action  func() error
    Compensate func() error
}

func (s *OrderSaga) Execute() error {
    for i, step := range s.Steps {
        if err := step.Action(); err != nil {
            // 回滚已执行的步骤
            for j := i - 1; j >= 0; j-- {
                s.Steps[j].Compensate()
            }
            return err
        }
    }
    return nil
}
```

---

**Q14：如何实现GORM的数据审计？**

**A14：**

**使用钩子实现审计日志：**

```go
type AuditLog struct {
    gorm.Model
    TableName  string
    RecordID   uint
    Action     string
    BeforeData string
    AfterData  string
    Operator   string
}

func (u *User) AfterUpdate(tx *gorm.DB) (err error) {
    // 获取更新前的数据
    before := User{}
    tx.Unscoped().First(&before, u.ID)
    
    // 创建审计日志
    log := AuditLog{
        TableName:  "users",
        RecordID:   u.ID,
        Action:     "UPDATE",
        BeforeData: json.Marshal(before),
        AfterData:  json.Marshal(u),
        Operator:   "admin",
    }
    
    tx.Create(&log)
    return
}
```

---

**Q15：如何设计GORM的读写分离？**

**A15：**

**使用读写分离中间件：**

```go
type DB struct {
    master *gorm.DB
    slaves []*gorm.DB
}

func (db *DB) Read() *gorm.DB {
    // 随机选择从库
    return db.slaves[rand.Intn(len(db.slaves))]
}

func (db *DB) Write() *gorm.DB {
    return db.master
}

// 使用
db.Write().Create(&user)
db.Read().Find(&users)
```

**使用gorm.io/plugin/dbresolver：**

```go
import "gorm.io/plugin/dbresolver"

db.Use(dbresolver.Register(dbresolver.Config{
    Sources: []gorm.Dialector{mysql.Open(masterDSN)},
    Replicas: []gorm.Dialector{mysql.Open(slaveDSN1), mysql.Open(slaveDSN2)},
}))
```

---

**Q16：如何处理GORM的大数据量查询？**

**A16：**

**分页查询：**

```go
var users []User
db.Limit(100).Offset(0).Find(&users)  // 第1页
db.Limit(100).Offset(100).Find(&users)  // 第2页
```

**游标分页（适合大数据量）：**

```go
var users []User
lastID := uint(0)

for {
    db.Where("id > ?", lastID).Limit(100).Find(&users)
    if len(users) == 0 {
        break
    }
    lastID = users[len(users)-1].ID
    // 处理数据
}
```

**批量删除：**

```go
// 分批删除
for {
    count := int64(0)
    db.Model(&User{}).Where("created_at < ?", time.Now().Add(-30*24*time.Hour)).
        Limit(1000).Delete(&User{}).Count(&count)
    if count == 0 {
        break
    }
}
```

---

**Q17：如何实现GORM的乐观锁？**

**A17：**

**使用version字段：**

```go
type User struct {
    gorm.Model
    Name    string
    Version int `gorm:"version"`
}

// 更新时检查版本
result := db.Model(&user).Where("version = ?", user.Version).Update("name", "New")
if result.RowsAffected == 0 {
    // 更新失败，版本已变更
    return errors.New("conflict")
}
```

**使用CAS操作：**

```go
db.Model(&User{}).Where("id = ? AND version = ?", id, version).
    Updates(map[string]interface{}{
        "name":    "New",
        "version": gorm.Expr("version + 1"),
    })
```

---

**Q18：如何实现GORM的缓存策略？**

**A18：**

**二级缓存实现：**

```go
type CacheDB struct {
    db    *gorm.DB
    cache *redis.Client
}

func (c *CacheDB) GetUser(id uint) (*User, error) {
    // 先查缓存
    key := fmt.Sprintf("user:%d", id)
    val, err := c.cache.Get(key).Result()
    if err == nil {
        var user User
        json.Unmarshal([]byte(val), &user)
        return &user, nil
    }
    
    // 再查数据库
    var user User
    if err := c.db.First(&user, id).Error; err != nil {
        return nil, err
    }
    
    // 更新缓存
    data, _ := json.Marshal(user)
    c.cache.Set(key, string(data), 5*time.Minute)
    
    return &user, nil
}
```

---

**Q19：如何设计GORM的表结构迁移策略？**

**A19：**

**迁移策略：**

1. **版本化迁移：**
```go
// migration/001_create_users.go
func Migrate_001(db *gorm.DB) error {
    return db.AutoMigrate(&User{})
}

// migration/002_add_email_index.go
func Migrate_002(db *gorm.DB) error {
    return db.Migrator().CreateIndex(&User{}, "idx_users_email")
}
```

2. **蓝色/绿色部署：**
```go
// 先部署新代码（兼容新旧表结构）
// 然后执行迁移
// 最后切换流量
```

3. **回滚策略：**
```go
// 保存迁移前的表结构
// 准备回滚脚本
// 测试回滚流程
```

---

**Q20：如何处理GORM的并发写入冲突？**

**A20：**

**解决方案：**

1. **悲观锁：**
```go
tx := db.Begin()
defer tx.Rollback()

// 锁定记录
var user User
tx.Clauses(clause.Locking{Strength: "UPDATE"}).First(&user, id)

// 更新
user.Balance -= amount
tx.Save(&user)

tx.Commit()
```

2. **乐观锁：**
```go
// 使用version字段
type Account struct {
    gorm.Model
    Balance int
    Version int `gorm:"version"`
}

// CAS更新
result := db.Model(&Account{}).Where("id = ? AND balance >= ? AND version = ?", 
    id, amount, version).
    Updates(map[string]interface{}{
        "balance": gorm.Expr("balance - ?", amount),
        "version": gorm.Expr("version + 1"),
    })

if result.RowsAffected == 0 {
    return errors.New("insufficient balance or conflict")
}
```

3. **队列串行化：**
```go
// 将写入请求放入消息队列
// 单消费者处理，避免并发冲突
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

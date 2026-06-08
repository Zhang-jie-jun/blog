# 单元测试

## 单元测试简介
### 什么是单元测试？
单元测试是对软件中的最小可测试单元进行验证的测试方法。在 Go 语言中，单元测试是开发流程中不可或缺的一部分，Go 提供了原生的 `testing` 包来支持单元测试。

### 测试的重要性
- **保证代码质量**：提前发现潜在的 bug
- **促进代码设计**：迫使开发者编写可测试的代码
- **支持重构**：修改代码后验证功能正确性
- **文档作用**：测试用例本身就是代码使用的文档

## Go 原生测试框架 testing
### 基本测试结构
```go
package calculator

import "testing"

func TestAdd(t *testing.T) {
    result := Add(2, 3)
    expected := 5
    if result != expected {
        t.Errorf("Add(2, 3) = %d, expected %d", result, expected)
    }
}
```

### 测试函数命名规则
- 函数名以 `Test` 开头
- 参数为 `*testing.T`
- 文件名以 `_test.go` 结尾

### 常用断言方法
```go
func TestExample(t *testing.T) {
    t.Log("这是一条日志")
    
    if condition {
        t.Skip("跳过此测试")
    }
    
    if !expected {
        t.Fail()      // 标记失败但继续执行
    }
    
    if !expected {
        t.FailNow()   // 标记失败并立即停止
    }
    
    t.Error("错误信息")    // 输出错误并继续
    t.Fatal("致命错误")   // 输出错误并停止
}
```

### 子测试
```go
func TestAddMultipleCases(t *testing.T) {
    tests := []struct {
        name     string
        a        int
        b        int
        expected int
    }{
        {"positive numbers", 2, 3, 5},
        {"negative numbers", -2, -3, -5},
        {"zero", 0, 0, 0},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result := Add(tt.a, tt.b)
            if result != tt.expected {
                t.Errorf("%s: Add(%d, %d) = %d, expected %d", 
                    tt.name, tt.a, tt.b, result, tt.expected)
            }
        })
    }
}
```

### 基准测试
```go
func BenchmarkAdd(b *testing.B) {
    // 重置计时器（排除初始化时间）
    b.ResetTimer()
    
    for i := 0; i < b.N; i++ {
        Add(2, 3)
    }
}
```

### 运行测试
```bash
go test                # 运行当前包的所有测试
go test -v             # 显示详细输出
go test -run TestAdd   # 运行指定测试
go test -bench=.       # 运行基准测试
go test -cover         # 生成覆盖率报告
```

## Mock 框架 gomock
### gomock 简介
gomock 是 Go 官方提供的 Mock 框架，通过代码生成实现接口的 Mock 对象。

### 安装
```bash
go get github.com/golang/mock/gomock
go install github.com/golang/mock/mockgen@latest
```

### 定义接口
```go
// service/user_service.go
package service

type UserRepository interface {
    FindByID(id int) (*User, error)
    Save(user *User) error
}

type User struct {
    ID   int
    Name string
}

type UserService struct {
    repo UserRepository
}

func NewUserService(repo UserRepository) *UserService {
    return &UserService{repo: repo}
}

func (s *UserService) GetUserName(id int) (string, error) {
    user, err := s.repo.FindByID(id)
    if err != nil {
        return "", err
    }
    return user.Name, nil
}
```

### 生成 Mock 代码
```bash
mockgen -source=service/user_service.go -destination=service/mock_user_repository.go -package=service
```

### 使用 Mock 对象
```go
package service

import (
    "testing"
    
    "github.com/golang/mock/gomock"
)

func TestUserService_GetUserName(t *testing.T) {
    // 创建控制器
    ctrl := gomock.NewController(t)
    defer ctrl.Finish()
    
    // 创建 Mock 对象
    mockRepo := NewMockUserRepository(ctrl)
    
    // 设置预期行为
    expectedUser := &User{ID: 1, Name: "张三"}
    mockRepo.EXPECT().FindByID(1).Return(expectedUser, nil)
    
    // 创建被测试对象
    service := NewUserService(mockRepo)
    
    // 执行测试
    result, err := service.GetUserName(1)
    
    // 验证结果
    if err != nil {
        t.Errorf("Unexpected error: %v", err)
    }
    if result != "张三" {
        t.Errorf("Expected '张三', got '%s'", result)
    }
}
```

### Mock 方法参数匹配
```go
// 精确匹配
mockRepo.EXPECT().FindByID(gomock.Eq(1)).Return(user, nil)

// 任意整数
mockRepo.EXPECT().FindByID(gomock.Any()).Return(user, nil)

// 自定义匹配
mockRepo.EXPECT().FindByID(gomock.Nil()).Return(nil, errors.New("nil id"))
```

### Mock 调用次数
```go
// 期望调用一次
mockRepo.EXPECT().FindByID(1).Return(user, nil).Times(1)

// 期望调用0次
mockRepo.EXPECT().FindByID(2).Return(nil, nil).Times(0)

// 期望调用至少一次
mockRepo.EXPECT().Save(gomock.Any()).Return(nil).MinTimes(1)
```

## 打桩工具 gomonkey
### gomonkey 简介
gomonkey 是一个强大的运行时打桩工具，可以在运行时修改函数、方法、甚至是变量的值。

### 安装
```bash
go get github.com/agiledragon/gomonkey/v2
```

### 打桩函数
```go
package utils

func GetCurrentTime() time.Time {
    return time.Now()
}
```

```go
package utils

import (
    "testing"
    "time"
    
    "github.com/agiledragon/gomonkey/v2"
)

func TestGetCurrentTime(t *testing.T) {
    // 创建桩函数
    patches := gomonkey.ApplyFunc(GetCurrentTime, func() time.Time {
        return time.Date(2023, 1, 1, 0, 0, 0, 0, time.UTC)
    })
    defer patches.Reset()
    
    // 测试
    result := GetCurrentTime()
    expected := time.Date(2023, 1, 1, 0, 0, 0, 0, time.UTC)
    if !result.Equal(expected) {
        t.Errorf("Expected %v, got %v", expected, result)
    }
}
```

### 打桩方法
```go
type Calculator struct{}

func (c *Calculator) Add(a, b int) int {
    return a + b
}
```

```go
func TestCalculator_Add(t *testing.T) {
    calc := &Calculator{}
    
    // 打桩方法
    patches := gomonkey.ApplyMethod(reflect.TypeOf(&Calculator{}), "Add", 
        func(_ *Calculator, a, b int) int {
            return a * b // 修改为乘法
        })
    defer patches.Reset()
    
    // 测试
    result := calc.Add(2, 3)
    if result != 6 { // 预期是乘法结果
        t.Errorf("Expected 6, got %d", result)
    }
}
```

### 打桩变量
```go
var AppVersion = "1.0.0"
```

```go
func TestAppVersion(t *testing.T) {
    // 打桩变量
    patches := gomonkey.ApplyVar(&AppVersion, "2.0.0")
    defer patches.Reset()
    
    if AppVersion != "2.0.0" {
        t.Errorf("Expected '2.0.0', got '%s'", AppVersion)
    }
}
```

## 数据库 Mock 框架 sqlmock
### sqlmock 简介
sqlmock 是一个用于模拟数据库操作的库，可以在不连接真实数据库的情况下测试数据库相关代码。

### 安装
```bash
go get github.com/DATA-DOG/go-sqlmock
```

### 使用示例
```go
package repository

import (
    "database/sql"
    "testing"
    
    "github.com/DATA-DOG/go-sqlmock"
)

func TestUserRepository_FindByID(t *testing.T) {
    // 创建 mock 数据库连接
    db, mock, err := sqlmock.New()
    if err != nil {
        t.Fatalf("Failed to create mock: %v", err)
    }
    defer db.Close()
    
    // 设置预期的 SQL 查询
    rows := sqlmock.NewRows([]string{"id", "name"}).
        AddRow(1, "张三")
    
    mock.ExpectQuery("SELECT id, name FROM users WHERE id = \\?").
        WithArgs(1).
        WillReturnRows(rows)
    
    // 创建 repository
    repo := NewUserRepository(db)
    
    // 执行查询
    user, err := repo.FindByID(1)
    
    // 验证结果
    if err != nil {
        t.Errorf("Unexpected error: %v", err)
    }
    if user.Name != "张三" {
        t.Errorf("Expected '张三', got '%s'", user.Name)
    }
    
    // 验证所有预期都已满足
    if err := mock.ExpectationsWereMet(); err != nil {
        t.Errorf("Unfulfilled expectations: %v", err)
    }
}
```

### Mock 插入操作
```go
func TestUserRepository_Save(t *testing.T) {
    db, mock, err := sqlmock.New()
    if err != nil {
        t.Fatalf("Failed to create mock: %v", err)
    }
    defer db.Close()
    
    // 设置预期的插入操作
    mock.ExpectExec("INSERT INTO users").
        WithArgs("张三", 18).
        WillReturnResult(sqlmock.NewResult(1, 1))
    
    repo := NewUserRepository(db)
    user := &User{Name: "张三", Age: 18}
    
    err = repo.Save(user)
    
    if err != nil {
        t.Errorf("Unexpected error: %v", err)
    }
    
    if err := mock.ExpectationsWereMet(); err != nil {
        t.Errorf("Unfulfilled expectations: %v", err)
    }
}
```

### Mock 错误场景
```go
func TestUserRepository_FindByID_Error(t *testing.T) {
    db, mock, err := sqlmock.New()
    if err != nil {
        t.Fatalf("Failed to create mock: %v", err)
    }
    defer db.Close()
    
    // 设置预期返回错误
    mock.ExpectQuery("SELECT id, name FROM users WHERE id = \\?").
        WithArgs(999).
        WillReturnError(sql.ErrNoRows)
    
    repo := NewUserRepository(db)
    
    _, err = repo.FindByID(999)
    
    if err == nil {
        t.Error("Expected error, got nil")
    }
}
```

## GoConvey 测试框架
### GoConvey 简介
GoConvey 是一个行为驱动开发（BDD）风格的测试框架，提供丰富的断言和漂亮的测试报告。

### 安装
```bash
go get github.com/smartystreets/goconvey/convey
```

### 基本使用
```go
package calculator

import (
    "testing"
    
    . "github.com/smartystreets/goconvey/convey"
)

func TestAdd(t *testing.T) {
    Convey("Given two numbers", t, func() {
        a := 2
        b := 3
        
        Convey("When adding them", func() {
            result := Add(a, b)
            
            Convey("Then the result should be their sum", func() {
                So(result, ShouldEqual, 5)
            })
        })
    })
}
```

### 嵌套测试
```go
func TestUserService(t *testing.T) {
    Convey("User Service", t, func() {
        ctrl := gomock.NewController(t)
        defer ctrl.Finish()
        
        mockRepo := NewMockUserRepository(ctrl)
        service := NewUserService(mockRepo)
        
        Convey("When user exists", func() {
            mockRepo.EXPECT().FindByID(1).Return(&User{ID: 1, Name: "张三"}, nil)
            
            name, err := service.GetUserName(1)
            
            So(err, ShouldBeNil)
            So(name, ShouldEqual, "张三")
        })
        
        Convey("When user does not exist", func() {
            mockRepo.EXPECT().FindByID(999).Return(nil, errors.New("not found"))
            
            _, err := service.GetUserName(999)
            
            So(err, ShouldNotBeNil)
        })
    })
}
```

### 常用断言
```go
So(value, ShouldBeNil)
So(value, ShouldNotBeNil)
So(value, ShouldEqual, expected)
So(value, ShouldNotEqual, expected)
So(value, ShouldBeTrue)
So(value, ShouldBeFalse)
So(value, ShouldContain, element)
So(value, ShouldNotContain, element)
So(value, ShouldHaveLength, length)
So(value, ShouldBeGreaterThan, other)
So(value, ShouldBeLessThan, other)
So(err, ShouldBeError, expected)
```

### 运行测试
```bash
go test -v

# 启动 Web UI（需要安装 goconvey 命令）
go install github.com/smartystreets/goconvey
goconvey
```

## Wire 依赖注入框架
### Wire 简介
Wire 是 Go 官方提供的编译时依赖注入工具，通过代码生成实现依赖注入，提高代码的可测试性和可维护性。

### 安装
```bash
go get github.com/google/wire
go install github.com/google/wire/cmd/wire@latest
```

### 核心概念
- **Provider**：提供依赖的函数
- **Injector**：组装依赖的函数，由 Wire 生成
- **Set**：一组 Provider 的集合

### 使用示例
#### 1. 定义接口和实现
```go
// repository/user_repository.go
package repository

type UserRepository interface {
    FindByID(id int) (*User, error)
    Save(user *User) error
}

type User struct {
    ID   int
    Name string
}

type MySQLUserRepository struct{}

func NewMySQLUserRepository() *MySQLUserRepository {
    return &MySQLUserRepository{}
}

func (r *MySQLUserRepository) FindByID(id int) (*User, error) {
    // 真实数据库查询
    return &User{ID: id, Name: "张三"}, nil
}

func (r *MySQLUserRepository) Save(user *User) error {
    // 真实数据库插入
    return nil
}
```

#### 2. 定义 Service
```go
// service/user_service.go
package service

import "your/package/repository"

type UserService struct {
    repo repository.UserRepository
}

func NewUserService(repo repository.UserRepository) *UserService {
    return &UserService{repo: repo}
}

func (s *UserService) GetUserName(id int) (string, error) {
    user, err := s.repo.FindByID(id)
    if err != nil {
        return "", err
    }
    return user.Name, nil
}
```

#### 3. 创建 Wire Set
```go
// wire/wire.go
//go:build wireinject
// +build wireinject

package wire

import (
    "your/package/repository"
    "your/package/service"
)

func InitializeUserService() *service.UserService {
    wire.Build(
        repository.NewMySQLUserRepository,
        service.NewUserService,
    )
    return nil
}
```

#### 4. 生成 Injector
```bash
wire
```

生成的 `wire_gen.go` 文件：
```go
// Code generated by Wire. DO NOT EDIT.

package wire

import (
    "your/package/repository"
    "your/package/service"
)

func InitializeUserService() *service.UserService {
    mySQLUserRepository := repository.NewMySQLUserRepository()
    userService := service.NewUserService(mySQLUserRepository)
    return userService
}
```

### Wire 提高可测试性
#### 测试时替换依赖
```go
// wire/wire_test.go
//go:build wireinject
// +build wireinject

package wire

import (
    "your/package/repository"
    "your/package/service"
)

func InitializeTestUserService(mockRepo repository.UserRepository) *service.UserService {
    wire.Build(
        service.NewUserService,
    )
    return nil
}
```

#### 测试代码
```go
package service

import (
    "testing"
    
    "github.com/golang/mock/gomock"
    "your/package/repository"
    "your/package/wire"
)

func TestUserService_GetUserName(t *testing.T) {
    // 创建 Mock 对象
    ctrl := gomock.NewController(t)
    defer ctrl.Finish()
    
    mockRepo := repository.NewMockUserRepository(ctrl)
    mockRepo.EXPECT().FindByID(1).Return(&repository.User{ID: 1, Name: "张三"}, nil)
    
    // 使用 Wire 注入 Mock 对象
    service := wire.InitializeTestUserService(mockRepo)
    
    // 执行测试
    result, err := service.GetUserName(1)
    
    if err != nil {
        t.Errorf("Unexpected error: %v", err)
    }
    if result != "张三" {
        t.Errorf("Expected '张三', got '%s'", result)
    }
}
```

### Wire 高级特性
#### 绑定接口到实现
```go
// wire/wire.go
func InitializeUserService() *service.UserService {
    wire.Build(
        // 将 UserRepository 接口绑定到 MySQLUserRepository
        wire.Bind(new(repository.UserRepository), new(*repository.MySQLUserRepository)),
        repository.NewMySQLUserRepository,
        service.NewUserService,
    )
    return nil
}
```

#### 提供值
```go
func provideDSN() string {
    return "root:password@tcp(localhost:3306)/db"
}

func InitializeUserService() *service.UserService {
    wire.Build(
        provideDSN,
        repository.NewMySQLUserRepository,
        service.NewUserService,
    )
    return nil
}
```

#### 清理函数
```go
func NewDB(dsn string) (*sql.DB, func(), error) {
    db, err := sql.Open("mysql", dsn)
    if err != nil {
        return nil, nil, err
    }
    
    cleanup := func() {
        db.Close()
    }
    
    return db, cleanup, nil
}
```

## 测试最佳实践
### 1. 面向接口编程
```go
// 不好的设计
type UserService struct {
    db *sql.DB // 具体类型，难以测试
}

// 好的设计
type UserService struct {
    repo UserRepository // 接口，易于 Mock
}
```

### 2. 避免全局状态
```go
// 不好的设计
var globalDB *sql.DB

func InitDB(dsn string) {
    globalDB, _ = sql.Open("mysql", dsn)
}

// 好的设计
func NewUserRepository(db *sql.DB) *UserRepository {
    return &UserRepository{db: db}
}
```

### 3. 测试隔离
```go
func TestXXX(t *testing.T) {
    // 每个测试用例独立
    ctrl := gomock.NewController(t)
    defer ctrl.Finish()
    
    // 测试逻辑...
}
```

### 4. 使用表格驱动测试
```go
func TestAdd(t *testing.T) {
    tests := []struct {
        name     string
        a, b     int
        expected int
    }{
        {"positive", 2, 3, 5},
        {"negative", -2, -3, -5},
        {"zero", 0, 0, 0},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result := Add(tt.a, tt.b)
            So(result, ShouldEqual, tt.expected)
        })
    }
}
```

### 5. 测试覆盖率
```bash
go test -coverprofile=coverage.out
go tool cover -html=coverage.out -o coverage.html
```

## 常见面试题
### 1. Go 单元测试的命名规则是什么？

**答案：**
- 测试函数名以 `Test` 开头
- 测试文件以 `_test.go` 结尾
- 测试函数参数为 `*testing.T`

### 2. 如何使用 gomock 进行 Mock 测试？

**答案：**
1. 定义接口
2. 使用 `mockgen` 生成 Mock 代码
3. 创建 `gomock.Controller`
4. 创建 Mock 对象并设置预期行为
5. 将 Mock 对象注入到被测试对象中
6. 执行测试并验证

### 3. gomonkey 和 gomock 的区别？

**答案：**
- **gomock**：基于接口的 Mock，需要面向接口编程，更规范
- **gomonkey**：运行时打桩，可以修改任意函数/方法/变量，更灵活但风险更高

### 4. 如何测试数据库操作？

**答案：**
使用 `sqlmock` 模拟数据库连接，设置预期的 SQL 查询和返回结果，验证数据库操作的正确性，无需连接真实数据库。

### 5. GoConvey 的特点是什么？

**答案：**
- BDD 风格的测试语法
- 嵌套的测试结构
- 丰富的断言函数
- 漂亮的 Web UI 测试报告

### 6. 什么是基准测试？如何编写？

**答案：**
基准测试用于评估代码的性能。以 `Benchmark` 开头，参数为 `*testing.B`，在循环中执行被测试代码 `b.N` 次。

### 7. 如何进行测试隔离？

**答案：**
- 每个测试用例独立初始化
- 使用 `defer` 清理资源
- 避免共享状态
- 使用 Mock 对象代替真实依赖

### 8. 什么是表格驱动测试？

**答案：**
将多个测试用例组织成表格形式，通过循环执行所有测试用例，提高测试代码的复用性和可读性。

### 9. 如何生成测试覆盖率报告？

**答案：**
```bash
go test -coverprofile=coverage.out
go tool cover -html=coverage.out
```

### 10. 测试和生产代码应该放在一起吗？

**答案：**
通常放在同一个包中，但测试文件以 `_test.go` 结尾。这样可以访问包内的非导出函数和变量，便于测试。

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

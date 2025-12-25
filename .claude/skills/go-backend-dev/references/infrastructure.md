# 基础设施参考

## 目录

1. [异常定义 (exp)](#异常定义-exp)
2. [常量定义 (constants)](#常量定义-constants)
3. [全局配置 (global)](#全局配置-global)
4. [领域辅助 (domain/helper)](#领域辅助-domainhelper)
5. [基础设施工具 (infrastructure/utils)](#基础设施工具-infrastructureutils)
6. [中间件 (middleware)](#中间件-middleware)
7. [日志](#日志)

---

## 异常定义 (exp)

位置：`backend/src/application/exp/`

### 定义格式

```go
package exp

import (
    "net/http"
    helper "github.com/lshaofan/go-framework/pkg/helper"
)

// 模块异常定义 - 错误码规则: {模块前缀}{序号}
// 模块前缀: 101xx=BackendUser, 102xx=Organization, 103xx=Application...

var (
    ErrBackendUserNotFound       = helper.NewErrorModel(10100, "用户不存在", nil, http.StatusNotFound)
    ErrBackendUserUsernameExists = helper.NewErrorModel(10101, "用户名已存在", nil, http.StatusBadRequest)
    ErrBackendUserEmailExists    = helper.NewErrorModel(10102, "邮箱已存在", nil, http.StatusBadRequest)
    ErrBackendUserPasswordWrong  = helper.NewErrorModel(10103, "密码错误", nil, http.StatusBadRequest)
    ErrBackendUserCreateFailed   = helper.NewErrorModel(10104, "创建用户失败", nil, http.StatusInternalServerError)
    ErrBackendUserUpdateFailed   = helper.NewErrorModel(10105, "更新用户失败", nil, http.StatusInternalServerError)
    ErrBackendUserDeleteFailed   = helper.NewErrorModel(10106, "删除用户失败", nil, http.StatusInternalServerError)
)
```

### 通用异常

```go
// backend/src/application/exp/errors.go
var (
    ErrNotFound       = helper.NewErrorModel(10000, "资源不存在", nil, http.StatusNotFound)
    ErrClientType     = helper.NewErrorModel(10001, "客户端类型错误", nil, http.StatusBadRequest)
    ErrUploadFileType = helper.NewErrorModel(10002, "文件类型不支持", nil, http.StatusBadRequest)
    ErrFileSize       = helper.NewErrorModel(10003, "文件大小超限", nil, http.StatusBadRequest)
)
```

### 认证异常

```go
// backend/src/application/exp/auth.go
var (
    UNAUTHORIZED     = helper.NewErrorModel(40100, "未授权", nil, http.StatusUnauthorized)
    InvalidToken     = helper.NewErrorModel(40101, "无效的Token", nil, http.StatusUnauthorized)
    TokenExpired     = helper.NewErrorModel(40102, "Token已过期", nil, http.StatusUnauthorized)
    RefreshTokenErr  = helper.NewErrorModel(40103, "刷新Token失败", nil, http.StatusUnauthorized)
)
```

### 使用方式

```go
// 在 Service 中
ret.SetError(exp.ErrBackendUserNotFound)

// 在 Action/Middleware 中
a := helper.NewGinActionImpl(c)
a.ThrowError(exp.UNAUTHORIZED)
```

---

## 常量定义 (constants)

位置：`backend/src/application/constants/`

### 状态/枚举常量

```go
// backend/src/application/constants/backend_user.go
package constants

const (
    // 状态
    BackendUserStatusNormal   uint = 1 // 正常
    BackendUserStatusDisabled uint = 2 // 禁用

    // 性别
    BackendUserGenderUnknown uint = 0 // 未知
    BackendUserGenderMale    uint = 1 // 男
    BackendUserGenderFemale  uint = 2 // 女
)
```

### Key 常量

```go
// backend/src/application/constants/auth.go
const (
    AuthorizeLoginCode     = "login"
    TokenCtxKey            = "token"
    BackendUserInfoKey     = "backend_user_info"
    AuthClaimsKey          = "auth_claims"
    HeaderKeyAdminToken    = "Authorization"
    RedisKeyTokenBlacklist = "token:blacklist:"
)
```

### 上下文 Key 类型

```go
// backend/src/application/constants/common.go
type CtxKey string
```

---

## 全局配置 (global)

位置：`backend/src/interfaces/global/`

### 全局配置入口

```go
// backend/src/interfaces/global/index.go
var Config *Conf  // 全局配置实例

type Conf struct {
    viper     *viper.Viper
    env       Env
    app       *AppConfig
    web       *WebConfig
    dbConfig  *DBConfig
}

// 获取配置
global.Config.GetEnv()        // 获取环境
global.Config.GetApp()        // 获取应用配置
global.Config.GetWeb()        // 获取 Web 配置
global.Config.GetDBConfig()   // 获取数据库配置
```

### 数据库配置

```go
// backend/src/interfaces/global/db.go
type DBConfig struct {
    gormDB      *gorm.DB
    redisClient *redis.Client
}

// 获取数据库连接
global.Config.GetDBConfig().GetGormDB()      // GORM 实例
global.Config.GetDBConfig().GetRedisClient() // Redis 客户端
```

### 应用配置

```go
// backend/src/interfaces/global/app.go
type AppConfig struct {
    domain      string
    protocol    string
    publicPath  string
    // ...
}

// 使用示例
global.Config.GetApp().GetDomain()              // 获取域名
global.Config.GetApp().GetDomainWithProtocol()  // 获取完整 URL
global.Config.GetApp().GetPublicPath()          // 获取公共路径
```

### Web 配置

```go
// backend/src/interfaces/global/web.go
type WebConfig struct {
    port  int
    v1Api string
}

global.Config.GetWeb().GetPort()     // 获取端口
global.Config.GetWeb().GetV1Api()    // 获取 API 路径前缀
```

---

## 领域辅助 (domain/helper)

位置：`backend/src/domain/helper/`

### 认证辅助

```go
// backend/src/domain/helper/auth.go

// 创建认证辅助实例
auth := helper.NewAuth(c)

// 设置上下文信息（用于 Service 层）
ctx := helper.SetBackendUserInfoCtx(ctx, ginCtx)

// 从上下文获取 Token
token := helper.GetTokenInCtx(ctx)

// 从上下文获取认证信息
claims := helper.GetAuthClaimsInCtx(ctx)
```

### 通用辅助

```go
// backend/src/domain/helper/common.go
helper.DeleteResource(...)  // 删除资源辅助方法
```

---

## 基础设施工具 (infrastructure/utils)

位置：`backend/src/infrastructure/utils/`

### 密码工具

```go
// backend/src/infrastructure/utils/password.go

// 密码加密
hashedPassword, err := utils.HashPassword(password)

// 密码验证
match := utils.VerifyPassword(password, hashedPassword)

// 检查是否是 bcrypt 哈希
isBcrypt := utils.IsBcryptHash(hash)

// 检查是否需要升级加密
needsUpgrade := utils.NeedsUpgrade(hash)
```

### 路径工具

```go
// backend/src/infrastructure/utils/common.go
fullPath := utils.GetFullPath(relativePath)
```

---

## 中间件 (middleware)

位置：`backend/src/interfaces/middleware/`

### 认证中间件

```go
// backend/src/interfaces/middleware/auth.go

// 管理后台认证
func AdminTokenAuth() gin.HandlerFunc

// 使用
r.Use(middleware.AdminTokenAuth())
```

### CORS 中间件

```go
// backend/src/interfaces/middleware/cors.go
func Cors() gin.HandlerFunc
```

### 中间件工作流程

```go
func AdminTokenAuth() gin.HandlerFunc {
    return func(c *gin.Context) {
        // 1. 获取 Token
        tokenStr := c.GetHeader(constants.HeaderKeyAdminToken)
        
        // 2. 验证 Token
        claims, _, err := tokenManager.ParseToken(c.Request.Context(), tokenStr)
        if err != nil {
            a.ThrowError(exp.InvalidToken)
            return
        }
        
        // 3. 设置上下文
        c.Set(constants.BackendUserInfoKey, claims.User)
        c.Set(constants.AuthClaimsKey, claims)
        c.Set(constants.TokenCtxKey, tokenStr)
        
        c.Next()
    }
}
```

---

## 日志

位置：`backend/src/infrastructure/log/`

### 系统日志

```go
// backend/src/infrastructure/log/system.go
type DefSysLogger struct {
    *logrus.Logger
}

// 返回错误并记录日志
func (l DefSysLogger) ReturnError(err error) error
func (l DefSysLogger) ReturnErrorf(format string, args ...interface{}) error
```

### 使用 logrus

项目使用 logrus 作为日志框架：

```go
import "github.com/sirupsen/logrus"

// 记录日志
logrus.Info("信息日志")
logrus.Warn("警告日志")
logrus.Error("错误日志")
logrus.WithFields(logrus.Fields{
    "user_id": userId,
    "action":  "login",
}).Info("用户登录")
```

---

## 在 Service 中使用全局配置

```go
type BackendUser struct {
    backendUserRepository repository.IBackendUserRepository
    app                   *global.AppConfig  // 注入应用配置
}

func NewBackendUser() *BackendUser {
    return &BackendUser{
        backendUserRepository: dao.NewBackendUser(),
        app:                   global.Config.GetApp(),  // 从全局获取
    }
}
```

## 在 DAO 中使用数据库

```go
func NewBackendUser() *BackendUser {
    return &BackendUser{
        Util: helper.NewUtil[models.BackendUser](global.Config.GetDBConfig().GetGormDB()),
    }
}
```

# API 和 Action 模式参考

## Action 实现模式

### 标准 Action 模板

```go
package actions

import (
    "code_bird_cloud/backend/src/application/dto"
    "code_bird_cloud/backend/src/domain/services"
    "github.com/gin-gonic/gin"
    helper "github.com/lshaofan/go-framework/pkg/helper"
)

// HandleCtxFunc - 上下文处理函数（在 common_actions.go 中定义）
var HandleCtxFunc = func(c *gin.Context, r helper.IBaseRequest) {
    // 设置上下文信息，如当前用户、租户等
}

// 列表 Action
func BackendUserList(c *gin.Context) {
    req := new(dto.BackendUserListRequest)
    helper.HandleRequest(c, req, func(r helper.IBaseRequest) *helper.DefaultResult {
        return services.NewBackendUserService().AdminList(r.(*dto.BackendUserListRequest))
    }, HandleCtxFunc)
}

// 创建 Action
func BackendUserCreate(c *gin.Context) {
    req := new(dto.BackendUserCreateRequest)
    helper.HandleRequest(c, req, func(r helper.IBaseRequest) *helper.DefaultResult {
        return services.NewBackendUserService().AdminCreate(r.(*dto.BackendUserCreateRequest))
    }, HandleCtxFunc)
}

// 详情 Action
func BackendUserShow(c *gin.Context) {
    req := new(dto.BackendUserShowRequest)
    helper.HandleRequest(c, req, func(r helper.IBaseRequest) *helper.DefaultResult {
        return services.NewBackendUserService().AdminShow(r.(*dto.BackendUserShowRequest))
    }, HandleCtxFunc)
}

// 更新 Action
func BackendUserUpdate(c *gin.Context) {
    req := new(dto.BackendUserUpdateRequest)
    helper.HandleRequest(c, req, func(r helper.IBaseRequest) *helper.DefaultResult {
        return services.NewBackendUserService().AdminUpdate(r.(*dto.BackendUserUpdateRequest))
    }, HandleCtxFunc)
}

// 部分更新 Action
func BackendUserPatch(c *gin.Context) {
    req := new(dto.BackendUserPatchRequest)
    helper.HandleRequest(c, req, func(r helper.IBaseRequest) *helper.DefaultResult {
        return services.NewBackendUserService().AdminPatch(r.(*dto.BackendUserPatchRequest))
    }, HandleCtxFunc)
}

// 删除 Action
func BackendUserDelete(c *gin.Context) {
    req := new(dto.BackendUserDeleteRequest)
    helper.HandleRequest(c, req, func(r helper.IBaseRequest) *helper.DefaultResult {
        return services.NewBackendUserService().AdminDelete(r.(*dto.BackendUserDeleteRequest))
    }, HandleCtxFunc)
}
```

## API 路由定义

### 标准路由模板

```go
package api

import (
    "code_bird_cloud/backend/src/interfaces/actions"
    "code_bird_cloud/backend/src/interfaces/middleware"
    "github.com/gin-gonic/gin"
)

func BackendUserRouter(r *gin.RouterGroup) {
    // 创建路由组
    backendUser := r.Group("/backend-user")

    // 应用认证中间件
    backendUser.Use(middleware.AdminTokenAuth())
    {
        // RESTful CRUD 路由
        backendUser.POST("", actions.BackendUserCreate)           // 创建
        backendUser.GET("", actions.BackendUserList)              // 列表
        backendUser.GET("/:id", actions.BackendUserShow)          // 详情
        backendUser.PUT("/:id", actions.BackendUserUpdate)        // 全量更新
        backendUser.PATCH("/:id", actions.BackendUserPatch)       // 部分更新
        backendUser.DELETE("/:id", actions.BackendUserDelete)     // 删除

        // 自定义操作
        backendUser.PUT("/:id/password", actions.BackendUserChangePassword)
        backendUser.POST("/:id/reset-password", actions.BackendUserResetPassword)
    }
}
```

### 路由注册

在 `api/api.go` 中注册路由：

```go
package api

import "github.com/gin-gonic/gin"

func RegisterRoutes(r *gin.RouterGroup) {
    // v1 版本
    v1 := r.Group("/v1")
    {
        BackendUserRouter(v1)
        OrganizationRouter(v1)
        ApplicationRouter(v1)
        // ... 其他模块路由
    }
}
```

## RESTful 路由规范

| 操作 | HTTP 方法 | 路径 | Action |
|------|-----------|------|--------|
| 列表 | GET | `/xxx` | XxxList |
| 创建 | POST | `/xxx` | XxxCreate |
| 详情 | GET | `/xxx/:id` | XxxShow |
| 全量更新 | PUT | `/xxx/:id` | XxxUpdate |
| 部分更新 | PATCH | `/xxx/:id` | XxxPatch |
| 删除 | DELETE | `/xxx/:id` | XxxDelete |

## 中间件使用

```go
// 认证中间件
backendUser.Use(middleware.AdminTokenAuth())

// 多个中间件
backendUser.Use(
    middleware.AdminTokenAuth(),
    middleware.OperationLog(),
)

// 单个路由使用中间件
backendUser.POST("", middleware.AdminTokenAuth(), actions.BackendUserCreate)
```

## 文件命名规范

```
actions/
├── index.go               # 导出和公共函数
├── common_actions.go      # 通用 Action（如上传）
└── {module}_actions.go    # 模块 Action

api/
├── api.go                 # 路由注册入口
├── common.go              # 公共路由（健康检查等）
└── {module}.go            # 模块路由
```

## 响应格式

所有 API 响应格式统一：

```json
// 成功响应
{
    "code": 0,
    "data": { ... },
    "message": "success"
}

// 错误响应
{
    "code": 400,
    "data": "错误详情",
    "message": "参数验证失败"
}
```

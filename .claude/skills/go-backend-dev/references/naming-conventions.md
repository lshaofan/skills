# 文件命名规范

## 目录结构命名

| 目录 | 命名规则 | 示例 |
|------|---------|------|
| `application/dto/` | `{module}_{operation}.go` | `backend_user_list.go` |
| `application/constants/` | `{module}.go` | `backend_user.go` |
| `application/exp/` | `{module}.go` | `backend_user.go` |
| `domain/models/` | `{module}.go` | `backend_user.go` |
| `domain/repository/` | `{module}.go` | `backend_user.go` |
| `domain/services/` | `{module}_{scope}_{operation}.go` | `backend_user_admin_list.go` |
| `domain/helper/` | `{功能}.go` | `auth.go`, `common.go` |
| `infrastructure/dao/` | `{module}.go` | `backend_user.go` |
| `infrastructure/utils/` | `{功能}.go` | `password.go` |
| `interfaces/api/` | `{module}.go` | `backend_user.go` |
| `interfaces/actions/` | `{module}_actions.go` | `backend_user_actions.go` |
| `interfaces/middleware/` | `{功能}.go` | `auth.go`, `cors.go` |

## DTO 文件命名

每个模块的 DTO 按操作类型拆分：

```
application/dto/
├── {module}.go              # 通用数据结构（XxxData）
├── {module}_list.go         # 列表操作
├── {module}_create.go       # 创建操作
├── {module}_show.go         # 详情查询
├── {module}_update.go       # 全量更新
├── {module}_patch.go        # 部分更新
└── {module}_delete.go       # 删除操作
```

**示例 - backend_user 模块**：
```
dto/
├── backend_user.go                    # BackendUserData
├── backend_user_list.go               # BackendUserListRequest/Response
├── backend_user_create.go             # BackendUserCreateRequest/Response
├── backend_user_show.go               # BackendUserShowRequest/Response
├── backend_user_update.go             # BackendUserUpdateRequest/Response
├── backend_user_patch.go              # BackendUserPatchRequest/Response
├── backend_user_delete.go             # BackendUserDeleteRequest/Response
├── backend_user_change_password.go    # 特殊操作
└── backend_user_reset_password.go     # 特殊操作
```

## Service 文件命名

Service 按 `{module}_{scope}_{operation}.go` 命名：

```
domain/services/
├── {module}.go                        # Service 结构和构造函数
├── {module}_admin_list.go             # Admin 列表
├── {module}_admin_create.go           # Admin 创建
├── {module}_admin_show.go             # Admin 详情
├── {module}_admin_update.go           # Admin 更新
├── {module}_admin_delete.go           # Admin 删除
└── {module}_helper.go                 # 辅助方法（可选）
```

**scope 说明**：
- `admin` - 管理后台接口
- `open` - 开放 API
- 无 scope - 通用方法

## 类型命名规范

### DTO 命名

| 类型 | 命名格式 | 示例 |
|------|---------|------|
| 列表请求 | `{Module}ListRequest` | `BackendUserListRequest` |
| 列表响应 | `{Module}ListResponse` | `BackendUserListResponse` |
| 创建请求 | `{Module}CreateRequest` | `BackendUserCreateRequest` |
| 创建响应 | `{Module}CreateResponse` | `BackendUserCreateResponse` |
| 详情请求 | `{Module}ShowRequest` | `BackendUserShowRequest` |
| 更新请求 | `{Module}UpdateRequest` | `BackendUserUpdateRequest` |
| 部分更新 | `{Module}PatchRequest` | `BackendUserPatchRequest` |
| 删除请求 | `{Module}DeleteRequest` | `BackendUserDeleteRequest` |
| 通用数据 | `{Module}Data` | `BackendUserData` |

### Repository 接口命名

```go
type I{Module}Repository interface {
    helper.BaseRepository[models.{Module}]
    // 自定义方法
}
```

示例：`IBackendUserRepository`

### Service 命名

```go
type {Module} struct { ... }

func New{Module}() *{Module}
func New{Module}Service() *{Module}  // 对外暴露的构造函数
```

### Action 函数命名

```go
func {Module}{Operation}(c *gin.Context)
```

示例：`BackendUserList`, `BackendUserCreate`, `BackendUserShow`

### 常量命名

```go
const (
    {Module}{Field}{Value} = value  // 状态/枚举常量
    {Key}Key = "key_name"           // Key 常量
)
```

示例：
```go
const (
    BackendUserStatusNormal   uint = 1
    BackendUserStatusDisabled uint = 2
    BackendUserGenderMale     uint = 1
)

const (
    TokenCtxKey         = "token"
    BackendUserInfoKey  = "backend_user_info"
)
```

### 异常命名

```go
var Err{Module}{Description} = helper.NewErrorModel(code, "消息", nil, httpStatus)
```

示例：
```go
var ErrBackendUserNotFound      = helper.NewErrorModel(10100, "用户不存在", nil, http.StatusNotFound)
var ErrBackendUserUsernameExists = helper.NewErrorModel(10101, "用户名已存在", nil, http.StatusBadRequest)
```

## 路由路径命名

RESTful 风格，使用 kebab-case：

```go
r.Group("/backend-user")      // 模块路由组
r.GET("")                     // 列表
r.POST("")                    // 创建
r.GET("/:id")                 // 详情
r.PUT("/:id")                 // 全量更新
r.PATCH("/:id")               // 部分更新
r.DELETE("/:id")              // 删除
r.PUT("/:id/password")        // 自定义操作
```

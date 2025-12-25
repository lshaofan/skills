# DTO 模式参考

## 基本规则

1. **所有请求 DTO 必须嵌入 `helper.BaseRequest`**
2. **列表请求额外嵌入 `helper.ListRequest`**
3. **使用标签**: `form`, `json`, `query`, `uri`, `binding`

## 列表请求模式

```go
package dto

import helper "github.com/lshaofan/go-framework/pkg/helper"

// 列表请求
type BackendUserListRequest struct {
    helper.BaseRequest
    helper.ListRequest                                                                      // page, page_size, order, field
    Username string `form:"username" json:"username" query:"username" binding:"omitempty"` // 模糊查询
    Status   *uint  `form:"status" json:"status" query:"status" binding:"omitempty,oneof=1 2"` // 精确筛选
}

// 列表响应
type BackendUserListResponse struct {
    Data     []BackendUserData `json:"data"`
    Total    int64             `json:"total"`
    Page     int               `json:"page"`
    PageSize int               `json:"page_size"`
}
```

## 创建请求模式

```go
// 创建请求
type BackendUserCreateRequest struct {
    helper.BaseRequest
    Username string `form:"username" json:"username" binding:"required"`
    Password string `form:"password" json:"password" binding:"required,min=6"`
    Nickname string `form:"nickname" json:"nickname" binding:"omitempty"`
    Email    string `form:"email" json:"email" binding:"omitempty,email"`
    Phone    string `form:"phone" json:"phone" binding:"omitempty"`
    Status   *uint  `form:"status" json:"status" binding:"omitempty,oneof=1 2"`
}

// 创建响应
type BackendUserCreateResponse struct {
    BackendUserData
}
```

## Show/Update/Delete 请求模式

```go
// Show 请求 - 使用 URI 参数
type BackendUserShowRequest struct {
    helper.BaseRequest
    ID uint `uri:"id" binding:"required"`
}

// Update 请求 - URI + Body
type BackendUserUpdateRequest struct {
    helper.BaseRequest
    ID       uint   `uri:"id" binding:"required"`
    Username string `form:"username" json:"username" binding:"required"`
    Nickname string `form:"nickname" json:"nickname" binding:"required"`
    // ... 其他必填字段
}

// Patch 请求 - 部分更新，使用指针类型
type BackendUserPatchRequest struct {
    helper.BaseRequest
    ID       uint    `uri:"id" binding:"required"`
    Nickname *string `form:"nickname" json:"nickname" binding:"omitempty"`
    Status   *uint   `form:"status" json:"status" binding:"omitempty,oneof=1 2"`
}

// Delete 请求
type BackendUserDeleteRequest struct {
    helper.BaseRequest
    ID uint `uri:"id" binding:"required"`
}
```

## 通用数据结构

```go
// 通用数据响应结构
type BackendUserData struct {
    ID        uint   `json:"id"`
    Username  string `json:"username"`
    Nickname  string `json:"nickname"`
    Email     string `json:"email"`
    Phone     string `json:"phone"`
    Status    uint   `json:"status"`
    CreatedAt string `json:"created_at"`
    UpdatedAt string `json:"updated_at"`
}
```

## Binding 验证规则

| 规则 | 说明 | 示例 |
|------|------|------|
| `required` | 必填 | `binding:"required"` |
| `omitempty` | 可选 | `binding:"omitempty"` |
| `min=N` | 最小长度/值 | `binding:"min=6"` |
| `max=N` | 最大长度/值 | `binding:"max=100"` |
| `email` | 邮箱格式 | `binding:"email"` |
| `oneof=a b` | 枚举值 | `binding:"oneof=1 2"` |
| `url` | URL 格式 | `binding:"url"` |

## 文件命名规范

```
dto/
├── {module}.go              # 通用数据结构
├── {module}_list.go         # 列表请求/响应
├── {module}_create.go       # 创建请求/响应
├── {module}_show.go         # 详情请求/响应
├── {module}_update.go       # 全量更新
├── {module}_patch.go        # 部分更新
└── {module}_delete.go       # 删除请求/响应
```

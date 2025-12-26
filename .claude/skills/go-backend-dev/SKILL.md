---
name: go-backend-dev
description: Go 后端开发规范技能，基于 DDD 四层架构和 go-framework SDK。用于：(1) 创建新的 API 接口（CRUD 操作）；(2) 添加新的业务模块（Model、DTO、Service、Action、API 路由）；(3) 修改或扩展现有功能；(4) 遵循项目统一的代码风格和架构规范。触发场景：当用户要求添加后端功能、创建 API、编写 Go 代码时使用此技能。
---

# Go 后端开发规范

## 任务类型识别

根据任务类型选择需要遵循的规范：

| 任务类型 | 涉及文件 | 必读参考文档 |
|---------|---------|-------------|
| **新增业务模块** | Model、Repository、DAO、DTO、Service、Action、API | 全部文档 |
| **新增 API 接口** | DTO、Service 方法、Action、路由 | dto-patterns.md, service-patterns.md, api-patterns.md |
| **修改业务逻辑** | Service | service-patterns.md, infrastructure.md |
| **添加数据模型** | Model、migrate | model-patterns.md, naming-conventions.md |
| **添加数据字段** | Model、DTO | model-patterns.md, dto-patterns.md |
| **添加异常/常量** | exp、constants | infrastructure.md |
| **修改中间件/认证** | middleware、helper | infrastructure.md |

## 架构概述

**DDD 四层架构**：

```
backend/src/
├── application/       # 应用层
│   ├── dto/           # 请求/响应数据结构
│   ├── constants/     # 业务常量
│   ├── exp/           # 异常定义
│   └── template/      # 模板文件
├── domain/            # 领域层
│   ├── models/        # GORM 数据模型
│   ├── repository/    # Repository 接口
│   ├── services/      # 业务服务
│   └── helper/        # 领域辅助函数
├── infrastructure/    # 基础设施层
│   ├── dao/           # DAO 实现
│   ├── config/        # 配置管理
│   ├── utils/         # 工具函数
│   └── log/           # 日志
└── interfaces/        # 接口层
    ├── api/           # 路由定义
    ├── actions/       # 请求处理器
    ├── middleware/    # 中间件
    └── global/        # 全局配置
```

## 核心框架

```go
import helper "github.com/lshaofan/go-framework/pkg/helper"
```

## 新增模块标准流程

1. **定义 Model** → `domain/models/{module}.go`
2. **定义 Repository 接口** → `domain/repository/{module}.go`
3. **实现 DAO** → `infrastructure/dao/{module}.go`
4. **定义 DTOs** → `application/dto/{module}_*.go`
5. **定义常量** → `application/constants/{module}.go`
6. **定义异常** → `application/exp/{module}.go`
7. **实现 Service** → `domain/services/{module}*.go`
8. **实现 Action** → `interfaces/actions/{module}_actions.go`
9. **注册路由** → `interfaces/api/{module}.go`

## 关键模式速查

### Model

```go
type Xxx struct {
    gorm.Model  // 必须嵌入，提供 ID、CreatedAt、UpdatedAt、DeletedAt
    Name   string `gorm:"size:255;not null" json:"name"`
    Status uint   `gorm:"type:tinyint(1);not null;default:1;comment:状态(1=正常、2=禁用)" json:"status"`
}

// TableName 必须实现，显式定义表名
func (Xxx) TableName() string {
    return "xxx"
}
```

### DTO

```go
type XxxListRequest struct {
    helper.BaseRequest
    helper.ListRequest  // 列表请求必须嵌入
    Name string `form:"name" json:"name" query:"name" binding:"omitempty"`
}
```

### Service

```go
func (s *Xxx) AdminList(in *dto.XxxListRequest) *helper.DefaultResult {
    ret := helper.NewDefaultResult()
    data, err := s.xxxRepository.GetListData(pageRequest)
    ret.SetResponse(data, err)
    return ret
}
```

### Action

```go
func XxxList(c *gin.Context) {
    req := new(dto.XxxListRequest)
    helper.HandleRequest(c, req, func(r helper.IBaseRequest) *helper.DefaultResult {
        return services.NewXxxService().AdminList(r.(*dto.XxxListRequest))
    }, HandleCtxFunc)
}
```

### 异常定义

```go
var ErrXxxNotFound = helper.NewErrorModel(10100, "Xxx不存在", nil, http.StatusNotFound)
```

### 常量定义

```go
const (
    XxxStatusNormal   uint = 1 // 正常
    XxxStatusDisabled uint = 2 // 禁用
)
```

## 参考文档

- **架构详解**: [references/architecture.md](references/architecture.md)
- **文件命名规范**: [references/naming-conventions.md](references/naming-conventions.md)
- **Model 模式**: [references/model-patterns.md](references/model-patterns.md)
- **DTO 模式**: [references/dto-patterns.md](references/dto-patterns.md)
- **Service 模式**: [references/service-patterns.md](references/service-patterns.md)
- **API/Action 模式**: [references/api-patterns.md](references/api-patterns.md)
- **基础设施**: [references/infrastructure.md](references/infrastructure.md)
- **go-framework API**: [references/go-framework-api.md](references/go-framework-api.md)

## 代码规范

- 函数长度 ≤ 50 行，嵌套 ≤ 3 层
- 中文注释说明复杂业务逻辑
- 数据库操作必须添加错误处理
- API 响应: `{"code": 0, "data": {}, "message": "success"}`

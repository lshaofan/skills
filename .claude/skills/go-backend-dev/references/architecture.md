# 架构详解

## 目录结构

```
backend/
├── cmd/               # 命令行入口
├── conf/              # 配置文件
│   ├── config.yaml    # 主配置
│   └── config.dev.yaml
├── docs/              # API 文档
└── src/
    ├── application/   # 应用层
    │   ├── dto/       # 数据传输对象
    │   ├── constants/ # 常量定义
    │   ├── exp/       # 异常定义
    │   └── template/  # 模板文件
    ├── domain/        # 领域层
    │   ├── models/    # 领域模型（GORM 实体）
    │   ├── repository/# Repository 接口
    │   ├── services/  # 领域服务
    │   └── helper/    # 领域辅助函数
    ├── infrastructure/# 基础设施层
    │   ├── dao/       # DAO 实现
    │   ├── config/    # 配置管理
    │   ├── utils/     # 工具函数
    │   ├── log/       # 日志
    │   ├── seed/      # 数据初始化
    │   └── authsdk/   # 认证 SDK
    └── interfaces/    # 接口层
        ├── api/       # API 路由定义
        ├── actions/   # 请求处理器
        ├── middleware/# 中间件
        ├── global/    # 全局配置
        ├── start/     # 启动逻辑
        └── build/     # 构建逻辑
```

## 各层职责

### 应用层 (application)

**职责**: 定义数据结构、常量和异常

| 目录 | 职责 | 示例 |
|------|------|------|
| `dto/` | 请求/响应数据结构 | `BackendUserListRequest` |
| `constants/` | 业务常量（状态码、Key 等） | `BackendUserStatusNormal` |
| `exp/` | 自定义异常 | `ErrBackendUserNotFound` |
| `template/` | 模板文件 | 邮件模板等 |

### 领域层 (domain)

**职责**: 核心业务逻辑

| 目录 | 职责 | 示例 |
|------|------|------|
| `models/` | GORM 数据模型 | `BackendUser` 结构体 |
| `repository/` | Repository 接口定义 | `IBackendUserRepository` |
| `services/` | 业务服务实现 | `BackendUser.AdminList()` |
| `helper/` | 领域辅助函数 | 认证上下文处理 |

### 基础设施层 (infrastructure)

**职责**: 技术实现细节

| 目录 | 职责 | 示例 |
|------|------|------|
| `dao/` | Repository 接口实现 | `BackendUser` DAO |
| `config/` | 配置读取管理 | 配置解析 |
| `utils/` | 通用工具函数 | 密码加密、路径处理 |
| `log/` | 日志处理 | 系统日志 |
| `seed/` | 数据初始化 | 初始数据填充 |
| `authsdk/` | 认证 SDK | 证书提供者 |

### 接口层 (interfaces)

**职责**: 处理外部请求

| 目录 | 职责 | 示例 |
|------|------|------|
| `api/` | 路由注册 | `BackendUserRouter()` |
| `actions/` | 请求处理器 | `BackendUserList()` |
| `middleware/` | 中间件 | 认证、CORS、日志 |
| `global/` | 全局配置 | 数据库、应用配置 |
| `start/` | 启动逻辑 | 服务初始化 |

## 依赖关系

```
                    interfaces
                   /    |    \
                  /     |     \
                 ↓      ↓      ↓
    middleware  actions  api
                   \     |
                    \    ↓
                     domain (services, helper)
                        |
                        ↓
                    application (dto, constants, exp)
                        ↑
                        |
                  infrastructure (dao, utils)
                        |
                        ↓
                    domain (repository interface)
```

- `interfaces` 依赖 `domain` 和 `application`
- `infrastructure` 实现 `domain` 定义的接口（依赖倒置）
- `domain` 依赖 `application` 的 DTO
- 各层通过接口解耦

## 模块文件清单

以 `backend_user` 模块为例，完整的文件列表：

```
# 应用层
application/
├── dto/
│   ├── backend_user.go                    # 通用数据结构
│   ├── backend_user_list.go               # 列表请求/响应
│   ├── backend_user_create.go             # 创建请求/响应
│   ├── backend_user_show.go               # 详情请求/响应
│   ├── backend_user_update.go             # 更新请求/响应
│   ├── backend_user_patch.go              # 部分更新
│   ├── backend_user_delete.go             # 删除请求/响应
│   ├── backend_user_change_password.go    # 修改密码
│   └── backend_user_reset_password.go     # 重置密码
├── constants/
│   └── backend_user.go                    # 状态、性别等常量
└── exp/
    └── backend_user.go                    # 用户相关异常

# 领域层
domain/
├── models/
│   └── backend_user.go                    # 数据模型
├── repository/
│   └── backend_user.go                    # Repository 接口
└── services/
    ├── backend_user.go                    # Service 主文件
    ├── backend_user_admin_list.go
    ├── backend_user_admin_create.go
    ├── backend_user_admin_show.go
    ├── backend_user_admin_update.go
    ├── backend_user_admin_patch.go
    ├── backend_user_admin_delete.go
    ├── backend_user_admin_change_password.go
    └── backend_user_admin_reset_password.go

# 基础设施层
infrastructure/
└── dao/
    └── backend_user.go                    # DAO 实现

# 接口层
interfaces/
├── api/
│   └── backend_user.go                    # 路由定义
└── actions/
    └── backend_user_actions.go            # Action 处理器
```

## 新增模块检查清单

创建新模块时，确保完成以下步骤：

- [ ] `domain/models/{module}.go` - 定义数据模型
- [ ] `domain/repository/{module}.go` - 定义 Repository 接口
- [ ] `infrastructure/dao/{module}.go` - 实现 DAO
- [ ] `application/dto/{module}*.go` - 定义所有 DTO
- [ ] `application/constants/{module}.go` - 定义常量
- [ ] `application/exp/{module}.go` - 定义异常
- [ ] `domain/services/{module}*.go` - 实现业务逻辑
- [ ] `interfaces/actions/{module}_actions.go` - 实现 Action
- [ ] `interfaces/api/{module}.go` - 注册路由
- [ ] 在 `interfaces/api/api.go` 中注册模块路由
- [ ] 在 `interfaces/global/migrate.go` 中添加模型迁移

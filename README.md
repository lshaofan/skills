# Dev Skills

项目开发规范技能包集合 - 为 Claude Code 提供专业化的开发辅助技能。

## 项目简介

本仓库用于存放和管理各类 Claude Code 技能包（Skills），每个技能包专注于解决特定的开发场景问题。

## 目录结构

```
skills/
├── .claude/skills/   # 技能包存放目录
│   ├── go-backend-dev/
│   └── skill-creator/
├── templates/        # 技能开发模板
├── docs/             # 项目文档
├── LICENSE
└── README.md
```

## 技能包列表

### go-backend-dev

Go 后端开发规范技能，基于 DDD 四层架构和 go-framework SDK。

**功能**：
- 创建新的 API 接口（CRUD 操作）
- 添加新的业务模块（Model、DTO、Service、Action、API 路由）
- 修改或扩展现有功能
- 遵循项目统一的代码风格和架构规范

**包含文档**：
| 文档 | 说明 |
|------|------|
| `SKILL.md` | 技能主文件，任务分类指引 |
| `references/architecture.md` | DDD 架构详解 |
| `references/naming-conventions.md` | 文件/类型命名规范 |
| `references/dto-patterns.md` | DTO 定义模式 |
| `references/service-patterns.md` | Service 实现模式 |
| `references/api-patterns.md` | API 路由和 Action 模式 |
| `references/infrastructure.md` | 异常、常量、全局配置、中间件等 |
| `references/go-framework-api.md` | go-framework SDK API 参考 |

---

## 安装使用

### 方式一：直接复制（推荐）

将技能包目录复制到你的项目 `.claude/skills/` 目录下：

```bash
# 复制整个技能目录
cp -r .claude/skills/go-backend-dev /your/project/.claude/skills/
```

### 方式二：使用 .skill 文件

1. 获取打包好的 `.skill` 文件（本质是 zip 格式）
2. 解压到项目的 `.claude/skills/` 目录：

```bash
unzip go-backend-dev.skill -d /your/project/.claude/skills/
```

### 方式三：Claude Code 技能管理

通过 Claude Code 的 `/install-skill` 命令安装。

---

## 使用方法

### 自动触发

技能会在以下场景自动触发：
- 当你要求添加后端功能
- 当你要求创建 API 接口
- 当你要求编写 Go 代码

### 手动调用

```
/go-backend-dev
```

### 使用示例

**示例 1：创建新模块**
```
帮我创建一个 article 文章管理模块，包含标题、内容、状态字段，支持 CRUD 操作
```

**示例 2：添加 API 接口**
```
在 backend_user 模块添加一个批量删除接口
```

**示例 3：修改业务逻辑**
```
修改用户列表接口，增加按创建时间范围筛选的功能
```

**示例 4：添加字段**
```
在 Organization 模型中添加 description 描述字段
```

### 任务类型与规范对照

技能会根据任务类型自动参考对应的规范文档：

| 任务类型 | 涉及文件 | 参考规范 |
|---------|---------|---------|
| 新增业务模块 | Model、Repository、DAO、DTO、Service、Action、API | 全部文档 |
| 新增 API 接口 | DTO、Service、Action、路由 | dto, service, api-patterns |
| 修改业务逻辑 | Service | service, infrastructure |
| 添加数据字段 | Model、DTO | naming-conventions, dto |
| 添加异常/常量 | exp、constants | infrastructure |

---

## 技能包开发

### 使用 skill-creator

本仓库包含 `skill-creator` 技能，可用于创建新技能：

```
/skill-creator
```

### 技能包结构

```
skill-name/
├── SKILL.md          # 技能定义文件（必需）
│   ├── YAML 前置元数据（name, description）
│   └── Markdown 指令内容
├── references/       # 参考文档（按需加载）
├── scripts/          # 脚本工具
└── assets/           # 资源文件
```

### 命名规范

- 技能包目录名：kebab-case（如 `go-backend-dev`）
- 文件名：小写

---

## 贡献指南

1. Fork 本仓库
2. 使用 `skill-creator` 创建新技能包
3. 遵循技能包开发规范
4. 提交 Pull Request

## 许可证

MIT License

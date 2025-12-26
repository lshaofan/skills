# Model 定义规范

## 基本规则

1. **必须嵌入 `gorm.Model`** - 提供 ID、CreatedAt、UpdatedAt、DeletedAt 基础字段
2. **必须实现 `TableName()` 方法** - 显式定义表名，避免自动推断
3. **字段使用 gorm 标签定义约束**
4. **字段使用中文注释说明用途**

## 标准 Model 模板

```go
package models

import "gorm.io/gorm"

// {Module} {模块中文名}模型
type {Module} struct {
    gorm.Model
    Name   string `gorm:"size:255;not null" json:"name"`     // 名称
    Status uint   `gorm:"type:tinyint(1);not null;default:1;comment:状态(1=正常、2=禁用)" json:"status"` // 状态
    Sort   int    `gorm:"not null;default:0;comment:排序" json:"sort"` // 排序
    
    // 关联
    Items []Item `gorm:"foreignKey:ModuleID" json:"items,omitempty"` // 关联项
}

// TableName 定义表名
func ({Module}) TableName() string {
    return "{table_name}"
}
```

## 完整示例

```go
package models

import "gorm.io/gorm"

// Media 媒体素材模型
type Media struct {
    gorm.Model
    GroupID      *uint  `gorm:"default:null;comment:分组ID" json:"group_id"`                        // 分组ID
    Name         string `gorm:"size:255;not null;comment:素材名称" json:"name"`                       // 素材名称
    OriginalName string `gorm:"size:255;not null;comment:原始文件名" json:"original_name"`             // 原始文件名
    Path         string `gorm:"size:500;not null;comment:相对路径" json:"path"`                       // 相对路径
    URL          string `gorm:"size:500;not null;comment:访问URL" json:"url"`                       // 访问URL
    FileType     string `gorm:"size:50;not null;comment:文件类型(image/video/file)" json:"file_type"` // 文件类型
    MimeType     string `gorm:"size:100;not null;comment:MIME类型" json:"mime_type"`                // MIME类型
    FileSize     int64  `gorm:"not null;comment:文件大小(字节)" json:"file_size"`                       // 文件大小
    Width        *int   `gorm:"default:null;comment:宽度(图片/视频)" json:"width"`                      // 宽度
    Height       *int   `gorm:"default:null;comment:高度(图片/视频)" json:"height"`                     // 高度
    Extension    string `gorm:"size:50;not null;comment:文件扩展名" json:"extension"`                  // 扩展名
    Description  string `gorm:"size:1000;null;comment:描述" json:"description"`                     // 描述
    Sort         int    `gorm:"not null;default:0;comment:排序" json:"sort"`                        // 排序
    UserID       uint   `gorm:"not null;comment:上传者ID" json:"user_id"`                            // 上传者ID

    // 关联
    Group *MediaGroup  `gorm:"foreignKey:GroupID" json:"group,omitempty"` // 所属分组
    User  *BackendUser `gorm:"foreignKey:UserID" json:"user,omitempty"`   // 上传者
}

// TableName 定义表名
func (Media) TableName() string {
    return "media"
}
```

## gorm.Model 提供的字段

```go
type Model struct {
    ID        uint           `gorm:"primarykey"`
    CreatedAt time.Time
    UpdatedAt time.Time
    DeletedAt gorm.DeletedAt `gorm:"index"`
}
```

- `ID` - 主键，自增
- `CreatedAt` - 创建时间，自动填充
- `UpdatedAt` - 更新时间，自动更新
- `DeletedAt` - 软删除时间

## GORM 标签规范

### 字段类型

| 标签 | 说明 | 示例 |
|------|------|------|
| `size:N` | 字符串长度 | `gorm:"size:255"` |
| `type:xxx` | 指定数据库类型 | `gorm:"type:tinyint(1)"` |
| `type:text` | 长文本 | `gorm:"type:text"` |
| `type:decimal(10,2)` | 精确小数 | `gorm:"type:decimal(10,2)"` |

### 约束

| 标签 | 说明 | 示例 |
|------|------|------|
| `not null` | 非空 | `gorm:"not null"` |
| `null` | 可空 | `gorm:"null"` |
| `unique` | 唯一 | `gorm:"unique"` |
| `default:X` | 默认值 | `gorm:"default:0"` |
| `comment:X` | 字段注释 | `gorm:"comment:状态"` |
| `index` | 普通索引 | `gorm:"index"` |
| `uniqueIndex` | 唯一索引 | `gorm:"uniqueIndex"` |

### 关联

| 标签 | 说明 | 示例 |
|------|------|------|
| `foreignKey:X` | 外键字段 | `gorm:"foreignKey:UserID"` |
| `references:X` | 引用字段 | `gorm:"references:ID"` |
| `many2many:X` | 多对多表名 | `gorm:"many2many:user_roles"` |
| `joinForeignKey:X` | 连接表外键 | `gorm:"joinForeignKey:user_id"` |

## 关联定义

### 一对多（HasMany）

```go
type Organization struct {
    gorm.Model
    Name         string        `gorm:"size:255;not null" json:"name"`
    Applications []Application `gorm:"foreignKey:OrganizationID" json:"applications,omitempty"`
}
```

### 多对一（BelongsTo）

```go
type Application struct {
    gorm.Model
    OrganizationID uint          `gorm:"not null" json:"organization_id"`
    Organization   *Organization `gorm:"foreignKey:OrganizationID" json:"organization,omitempty"`
}
```

### 多对多（Many2Many）

```go
type BackendUser struct {
    gorm.Model
    Username      string         `gorm:"size:255;not null;unique" json:"username"`
    Organizations []Organization `gorm:"many2many:user_organization;joinForeignKey:user_id;joinReferences:organization_id" json:"organizations,omitempty"`
}
```

## 表名命名规范

| 规则 | 示例 |
|------|------|
| 使用蛇形命名（snake_case） | `backend_user` |
| 使用复数形式（可选，保持一致） | `media` 或 `medias` |
| 多对多中间表 | `user_organization` |

## 字段命名规范

| Go 字段 | 数据库列 | 说明 |
|---------|---------|------|
| `UserID` | `user_id` | 外键字段 |
| `CreatedAt` | `created_at` | 时间字段 |
| `IsSuper` | `is_super` | 布尔字段 |
| `FileSize` | `file_size` | 复合词 |

## 可空字段处理

使用指针类型表示可空字段：

```go
type Media struct {
    GroupID  *uint  `gorm:"default:null" json:"group_id"`   // 可空外键
    Width    *int   `gorm:"default:null" json:"width"`      // 可空整数
    Duration *int   `gorm:"default:null" json:"duration"`   // 可空整数
}
```

## 枚举/状态字段

使用 `tinyint(1)` + `comment` 说明：

```go
Status uint `gorm:"type:tinyint(1);not null;default:1;comment:状态(1=正常、2=禁用)" json:"status"`
Gender uint `gorm:"type:tinyint(1);null;default:0;comment:性别(0=未知、1=男、2=女)" json:"gender"`
```

## 在 migrate.go 中注册

新增 Model 后需要在 `interfaces/global/migrate.go` 中注册：

```go
func AutoMigrate(db *gorm.DB) error {
    return db.AutoMigrate(
        &models.BackendUser{},
        &models.Organization{},
        &models.Application{},
        &models.Media{},       // 新增模型
        &models.MediaGroup{},
    )
}
```

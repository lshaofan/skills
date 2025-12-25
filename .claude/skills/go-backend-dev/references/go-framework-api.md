# go-framework SDK API 参考

## 导入

```go
import helper "github.com/lshaofan/go-framework/pkg/helper"
```

## 请求处理

### HandleRequest

处理 HTTP 请求的核心函数：

```go
func HandleRequest(
    c *gin.Context,
    req IBaseRequest,
    serviceCall func(IBaseRequest) *DefaultResult,
    ctxFunc BaseCtxFunc,
)
```

**使用示例：**

```go
func XxxList(c *gin.Context) {
    req := new(dto.XxxListRequest)
    helper.HandleRequest(c, req, func(r helper.IBaseRequest) *helper.DefaultResult {
        return services.NewXxxService().AdminList(r.(*dto.XxxListRequest))
    }, HandleCtxFunc)
}
```

## 请求 DTO

### BaseRequest

所有请求 DTO 必须嵌入：

```go
type BaseRequest struct {
    Ctx context.Context
}

func (r *BaseRequest) SetContext(ctx context.Context)
func (r *BaseRequest) GetContext() context.Context
```

### ListRequest

列表请求通用字段：

```go
type ListRequest struct {
    Page     int    `form:"page" json:"page" query:"page" binding:"required"`
    PageSize int    `form:"page_size" json:"page_size" query:"page_size" binding:"required"`
    Order    string `form:"order" json:"order" query:"order"`   // asc/desc
    Field    string `form:"field" json:"field" query:"field"`   // 排序字段
}
```

## 响应处理

### DefaultResult

Service 方法返回类型：

```go
type DefaultResult struct {
    Err  error
    Data interface{}
}

func NewDefaultResult() *DefaultResult
func (r *DefaultResult) IsError() bool
func (r *DefaultResult) GetError() error
func (r *DefaultResult) SetError(err error)
func (r *DefaultResult) GetData() interface{}
func (r *DefaultResult) SetData(data interface{})
func (r *DefaultResult) SetResponse(data interface{}, err error)
```

**使用示例：**

```go
func (s *Xxx) AdminList(in *dto.XxxListRequest) *helper.DefaultResult {
    ret := helper.NewDefaultResult()
    data, err := s.xxxRepository.GetListData(pageRequest)
    ret.SetResponse(data, err)  // 同时设置 data 和 error
    return ret
}
```

### PageList

分页列表响应：

```go
type PageList[T any] struct {
    Total    int64 `json:"total"`
    Data     []T   `json:"data"`
    Page     int   `json:"page"`
    PageSize int   `json:"page_size"`
}

func NewPageList[T any](data []T, total int64, page, pageSize int) *PageList[T]
```

## ORM 工具

### Util[T]

泛型 ORM 工具类：

```go
type Util[T any] struct {
    DB                *gorm.DB
    Model             T
    PageRequestParams *PageRequest
}

func NewUtil[T any](db *gorm.DB) *Util[T]

// 查询方法
func (u *Util[T]) GetOne(where map[string]interface{}) (*T, error)
func (u *Util[T]) GetList(where map[string]interface{}) ([]T, error)
func (u *Util[T]) GetListWithData(pageRequest *PageRequest) (*PageList[T], error)
func (u *Util[T]) GetAll() ([]T, error)

// 创建方法
func (u *Util[T]) CreateOne(model *T) error
func (u *Util[T]) CreateMany(models []T) error

// 更新方法
func (u *Util[T]) UpdateOne(model *T) error
func (u *Util[T]) UpdateOneColumn(model *T, column string, value interface{}) error
func (u *Util[T]) UpdateMany(models []T) error

// 删除方法
func (u *Util[T]) DeleteOne(model *T) error
func (u *Util[T]) DeleteMany(models []T) error

// 数据库访问
func (u *Util[T]) SetDB(db *gorm.DB)
func (u *Util[T]) GetDB() *gorm.DB
```

### PageRequest

分页查询请求：

```go
type PageRequest struct {
    Page     int
    PageSize int
    Total    int64
    Where    map[string]interface{}
    OrWhere  map[string]interface{}
}

func NewPageReq(page, pageSize int) *PageRequest
func (p *PageRequest) AscSort(field string) *PageRequest
func (p *PageRequest) DescSort(field string) *PageRequest
```

**Where 条件格式：**

```go
pageRequest := &helper.PageRequest{
    Page:     1,
    PageSize: 10,
    Where:    make(map[string]interface{}),
}

// 精确匹配
pageRequest.Where["status"] = 1

// LIKE 查询
pageRequest.Where["username LIKE ?"] = "%" + keyword + "%"

// 排序（特殊键）
pageRequest.Where["_order_by"] = "created_at desc"
```

## Repository 接口

### BaseRepository

基础 Repository 接口：

```go
type BaseRepository[T any] interface {
    GetOneById(id uint) (*T, error)
    FindByField(field string, value interface{}) (*T, error)
    ExistsById(id uint) (bool, error)
    DeleteById(id uint) error
    Create(model *T) error
    Update(model *T) error
    UpdateWithColumns(model *T, columns []string) error
    GetListData(pageRequest *PageRequest) (*PageList[T], error)
}
```

## JWT 工具

### JWTUtil

```go
type JWTUtil struct {
    SecretKey      string
    Issuer         string
    Expired        time.Duration
    RefreshExpired time.Duration
}

func NewJWTUtil(options ...JwtOption) *JWTUtil
func (j *JWTUtil) GenerateToken(claims jwt.MapClaims) (*JWTResponse, error)
func (j *JWTUtil) GetToken(c *gin.Context) string
func (j *JWTUtil) ParseToken(tokenString string) (jwt.MapClaims, error)
func (j *JWTUtil) ValidateToken(tokenString string) (jwt.MapClaims, error)
```

**配置选项：**

```go
helper.WithOAuthJWTSecretKey("secret")
helper.WithOAuthJWTConfigIssuer("issuer")
helper.WithOAuthJWTConfigExpired(24 * time.Hour)
helper.WithOAuthJWTConfigRefreshExpired(7 * 24 * time.Hour)
```

## Action 接口

### Action

```go
type Action interface {
    Success(data interface{})
    Error(err error)
    ThrowError(err error)
    ThrowValidateError(err error)
    BindParam(i interface{}) error
    CreateOK(data interface{})
    UpdateOK(data interface{})
    DeleteOK(data interface{})
    SuccessWithMessage(data interface{}, message string)
}
```

## 中间件

### GinException

异常处理中间件：

```go
func GinException() gin.HandlerFunc
```

**使用：**

```go
r := gin.Default()
r.Use(helper.GinException())
```

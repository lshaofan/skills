# Service 模式参考

## Service 结构定义

```go
package services

import (
    "code_bird_cloud/backend/src/application/dto"
    "code_bird_cloud/backend/src/domain/repository"
    "code_bird_cloud/backend/src/infrastructure/dao"
    "code_bird_cloud/backend/src/interfaces/global"
    helper "github.com/lshaofan/go-framework/pkg/helper"
)

type BackendUser struct {
    backendUserRepository repository.IBackendUserRepository
    app                   *global.AppConfig
}

func NewBackendUser() *BackendUser {
    return &BackendUser{
        backendUserRepository: dao.NewBackendUser(),
        app:                   global.Config,
    }
}

func NewBackendUserService() *BackendUser {
    return NewBackendUser()
}
```

## 方法返回规范

**所有 Service 方法必须返回 `*helper.DefaultResult`**

```go
func (s *BackendUser) AdminList(in *dto.BackendUserListRequest) *helper.DefaultResult {
    ret := helper.NewDefaultResult()
    data, err := s.adminListData(in)
    ret.SetResponse(data, err)
    return ret
}
```

## 列表查询模式

```go
func (s *BackendUser) adminListData(in *dto.BackendUserListRequest) (*dto.BackendUserListResponse, error) {
    // 1. 构建查询请求
    pageRequest := &helper.PageRequest{
        Page:     in.Page,
        PageSize: in.PageSize,
        Where:    make(map[string]interface{}),
    }

    // 2. 添加筛选条件
    if in.Username != "" {
        pageRequest.Where["username LIKE ?"] = "%" + in.Username + "%"
    }
    if in.Status != nil {
        pageRequest.Where["status"] = *in.Status
    }

    // 3. 处理排序
    if in.Field != "" && in.Order != "" {
        pageRequest.Where["_order_by"] = fmt.Sprintf("%s %s", in.Field, in.Order)
    }

    // 4. 调用 Repository
    pageList, err := s.backendUserRepository.GetListData(pageRequest)
    if err != nil {
        return nil, err
    }

    // 5. 转换响应格式
    userDataList := make([]dto.BackendUserData, 0, len(pageList.Data))
    for _, user := range pageList.Data {
        userDataList = append(userDataList, dto.BackendUserData{
            ID:        user.ID,
            Username:  user.Username,
            CreatedAt: user.CreatedAt.Format("2006-01-02 15:04:05"),
        })
    }

    return &dto.BackendUserListResponse{
        Data:     userDataList,
        Total:    pageList.Total,
        Page:     pageList.Page,
        PageSize: pageList.PageSize,
    }, nil
}
```

## 创建操作模式

```go
func (s *BackendUser) AdminCreate(in *dto.BackendUserCreateRequest) *helper.DefaultResult {
    ret := helper.NewDefaultResult()

    // 1. 业务验证
    exists, _ := s.backendUserRepository.FindByField("username", in.Username)
    if exists != nil {
        ret.SetError(errors.New("用户名已存在"))
        return ret
    }

    // 2. 创建实体
    user := &models.BackendUser{
        Username: in.Username,
        Password: utils.HashPassword(in.Password),
        Nickname: in.Nickname,
    }

    // 3. 保存到数据库
    err := s.backendUserRepository.Create(user)
    if err != nil {
        ret.SetError(err)
        return ret
    }

    // 4. 返回结果
    ret.SetData(&dto.BackendUserCreateResponse{
        BackendUserData: dto.BackendUserData{
            ID:       user.ID,
            Username: user.Username,
        },
    })
    return ret
}
```

## 更新操作模式

```go
func (s *BackendUser) AdminUpdate(in *dto.BackendUserUpdateRequest) *helper.DefaultResult {
    ret := helper.NewDefaultResult()

    // 1. 查询现有数据
    user, err := s.backendUserRepository.GetOneById(in.ID)
    if err != nil {
        ret.SetError(errors.New("用户不存在"))
        return ret
    }

    // 2. 更新字段
    user.Nickname = in.Nickname
    user.Email = in.Email

    // 3. 保存更新
    err = s.backendUserRepository.Update(user)
    ret.SetResponse(&dto.BackendUserUpdateResponse{}, err)
    return ret
}
```

## 删除操作模式

```go
func (s *BackendUser) AdminDelete(in *dto.BackendUserDeleteRequest) *helper.DefaultResult {
    ret := helper.NewDefaultResult()

    // 1. 检查是否存在
    _, err := s.backendUserRepository.GetOneById(in.ID)
    if err != nil {
        ret.SetError(errors.New("用户不存在"))
        return ret
    }

    // 2. 执行删除
    err = s.backendUserRepository.DeleteById(in.ID)
    ret.SetResponse(nil, err)
    return ret
}
```

## 文件命名规范

```
services/
├── {module}.go              # Service 结构定义和构造函数
├── {module}_admin_list.go   # 列表方法
├── {module}_admin_create.go # 创建方法
├── {module}_admin_show.go   # 详情方法
├── {module}_admin_update.go # 更新方法
├── {module}_admin_patch.go  # 部分更新
└── {module}_admin_delete.go # 删除方法
```

## 错误处理

```go
// 使用自定义异常
import "code_bird_cloud/backend/src/application/exp"

ret.SetError(exp.ErrBackendUserNotFound)  // 预定义错误
ret.SetError(errors.New("自定义错误信息"))   // 动态错误
```

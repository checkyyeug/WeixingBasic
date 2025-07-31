# API接口文档

## 1. 概述

本文档详细描述了微信小程序用户管理系统的后端API接口，包括接口地址、请求方法、请求参数、响应格式、错误码等信息。

## 2. 公共信息

### 2.1 基础URL
```
https://api.example.com/v1
```

### 2.2 请求头
```
Content-Type: application/json
Authorization: Bearer <token> (部分接口需要)
```

### 2.3 响应格式
```json
{
  "code": 200,
  "message": "success",
  "data": {}
}
```

### 2.4 通用错误码
| 错误码 | 说明 |
|-------|------|
| 200 | 请求成功 |
| 400 | 请求参数错误 |
| 401 | 未授权/Token过期 |
| 403 | 权限不足 |
| 404 | 资源不存在 |
| 500 | 服务器内部错误 |

## 3. 认证相关接口

### 3.1 用户注册

**接口地址**: `POST /auth/register`

**请求参数**:
```json
{
  "username": "string (3-20字符，字母数字下划线)",
  "email": "string (邮箱格式)",
  "password": "string (6-20字符)"
}
```

**响应示例**:
```json
{
  "code": 200,
  "message": "注册成功",
  "data": {
    "userInfo": {
      "id": 1,
      "username": "testuser",
      "email": "test@example.com",
      "nickname": null,
      "phone": null,
      "avatar": null,
      "bio": null,
      "createdAt": "2023-01-01T00:00:00Z"
    },
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

**错误码**:
| 错误码 | 说明 |
|-------|------|
| 1001 | 用户名已存在 |
| 1002 | 邮箱已存在 |
| 1003 | 用户名格式不正确 |
| 1004 | 邮箱格式不正确 |
| 1005 | 密码长度不符合要求 |

### 3.2 用户登录

**接口地址**: `POST /auth/login`

**请求参数**:
```json
{
  "username": "string",
  "password": "string"
}
```

**响应示例**:
```json
{
  "code": 200,
  "message": "登录成功",
  "data": {
    "userInfo": {
      "id": 1,
      "username": "testuser",
      "email": "test@example.com",
      "nickname": "测试用户",
      "phone": null,
      "avatar": "https://example.com/avatar.jpg",
      "bio": "个人简介",
      "roles": [
        {
          "id": 1,
          "name": "user",
          "description": "普通用户"
        }
      ],
      "createdAt": "2023-01-01T00:00:00Z"
    },
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

**错误码**:
| 错误码 | 说明 |
|-------|------|
| 1011 | 用户名或密码错误 |
| 1012 | 用户已被禁用 |

### 3.3 刷新Token

**接口地址**: `POST /auth/refresh`

**请求参数**:
```json
{
  "refreshToken": "string"
}
```

**响应示例**:
```json
{
  "code": 200,
  "message": "刷新成功",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

**错误码**:
| 错误码 | 说明 |
|-------|------|
| 1021 | refreshToken无效或已过期 |

### 3.4 验证Token

**接口地址**: `POST /auth/verify`

**请求参数**:
```json
{
  "token": "string"
}
```

**响应示例**:
```json
{
  "code": 200,
  "message": "Token有效",
  "data": {
    "valid": true
  }
}
```

**错误码**:
| 错误码 | 说明 |
|-------|------|
| 1031 | Token无效或已过期 |

## 4. 用户相关接口

### 4.1 获取用户信息

**接口地址**: `GET /user/profile`

**请求头**: 需要Authorization

**响应示例**:
```json
{
  "code": 200,
  "message": "获取成功",
  "data": {
    "id": 1,
    "username": "testuser",
    "email": "test@example.com",
    "nickname": "测试用户",
    "phone": "13800138000",
    "avatar": "https://example.com/avatar.jpg",
    "bio": "个人简介",
    "roles": [
      {
        "id": 1,
        "name": "user",
        "description": "普通用户",
        "permissions": [
          {
            "id": 1,
            "code": "USER_VIEW",
            "name": "查看用户",
            "description": "查看用户列表权限",
            "category": "用户管理"
          }
        ]
      }
    ],
    "createdAt": "2023-01-01T00:00:00Z"
  }
}
```

**错误码**:
| 错误码 | 说明 |
|-------|------|
| 1041 | 用户不存在 |

### 4.2 更新用户信息

**接口地址**: `PUT /user/profile`

**请求头**: 需要Authorization

**请求参数**:
```json
{
  "nickname": "string (可选)",
  "phone": "string (可选)",
  "bio": "string (可选)"
}
```

**响应示例**:
```json
{
  "code": 200,
  "message": "更新成功",
  "data": {
    "id": 1,
    "username": "testuser",
    "email": "test@example.com",
    "nickname": "新昵称",
    "phone": "13800138000",
    "avatar": "https://example.com/avatar.jpg",
    "bio": "新的个人简介",
    "roles": [
      {
        "id": 1,
        "name": "user",
        "description": "普通用户"
      }
    ],
    "createdAt": "2023-01-01T00:00:00Z",
    "updatedAt": "2023-01-02T00:00:00Z"
  }
}
```

**错误码**:
| 错误码 | 说明 |
|-------|------|
| 1051 | 手机号格式不正确 |

### 4.3 更新用户头像

**接口地址**: `POST /user/avatar`

**请求头**: 需要Authorization

**请求参数**:
```json
{
  "avatar": "string (头像URL)"
}
```

**响应示例**:
```json
{
  "code": 200,
  "message": "更新成功",
  "data": {
    "avatarUrl": "https://example.com/new-avatar.jpg"
  }
}
```

## 5. 角色相关接口

### 5.1 获取角色列表

**接口地址**: `GET /roles`

**请求头**: 需要Authorization，需要ROLE_MANAGE权限

**查询参数**:
```
page: int (可选，默认1)
pageSize: int (可选，默认10)
name: string (可选，角色名称模糊搜索)
```

**响应示例**:
```json
{
  "code": 200,
  "message": "获取成功",
  "data": {
    "list": [
      {
        "id": 1,
        "name": "admin",
        "description": "管理员",
        "permissions": [
          {
            "id": 1,
            "code": "USER_VIEW",
            "name": "查看用户",
            "description": "查看用户列表权限",
            "category": "用户管理"
          }
        ],
        "createdAt": "2023-01-01T00:00:00Z",
        "updatedAt": "2023-01-01T00:00:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "pageSize": 10,
      "total": 3,
      "totalPage": 1
    }
  }
}
```

### 5.2 获取角色详情

**接口地址**: `GET /roles/{id}`

**请求头**: 需要Authorization，需要ROLE_MANAGE权限

**路径参数**:
```
id: int (角色ID)
```

**响应示例**:
```json
{
  "code": 200,
  "message": "获取成功",
  "data": {
    "id": 1,
    "name": "admin",
    "description": "管理员",
    "permissions": [
      {
        "id": 1,
        "code": "USER_VIEW",
        "name": "查看用户",
        "description": "查看用户列表权限",
        "category": "用户管理"
      }
    ],
    "createdAt": "2023-01-01T00:00:00Z",
    "updatedAt": "2023-01-01T00:00:00Z"
  }
}
```

**错误码**:
| 错误码 | 说明 |
|-------|------|
| 1061 | 角色不存在 |

### 5.3 创建角色

**接口地址**: `POST /roles`

**请求头**: 需要Authorization，需要ROLE_MANAGE权限

**请求参数**:
```json
{
  "name": "string (角色名称)",
  "description": "string (角色描述)",
  "permissionIds": ["int"] (权限ID列表，可选)
}
```

**响应示例**:
```json
{
  "code": 200,
  "message": "创建成功",
  "data": {
    "id": 4,
    "name": "new_role",
    "description": "新角色",
    "permissions": [],
    "createdAt": "2023-01-02T00:00:00Z",
    "updatedAt": "2023-01-02T00:00:00Z"
  }
}
```

**错误码**:
| 错误码 | 说明 |
|-------|------|
| 1071 | 角色名称已存在 |
| 1072 | 权限ID不存在 |

### 5.4 更新角色

**接口地址**: `PUT /roles/{id}`

**请求头**: 需要Authorization，需要ROLE_MANAGE权限

**路径参数**:
```
id: int (角色ID)
```

**请求参数**:
```json
{
  "name": "string (角色名称，可选)",
  "description": "string (角色描述，可选)",
  "permissionIds": ["int"] (权限ID列表，可选)
}
```

**响应示例**:
```json
{
  "code": 200,
  "message": "更新成功",
  "data": {
    "id": 1,
    "name": "updated_admin",
    "description": "更新后的管理员",
    "permissions": [
      {
        "id": 1,
        "code": "USER_VIEW",
        "name": "查看用户",
        "description": "查看用户列表权限",
        "category": "用户管理"
      }
    ],
    "createdAt": "2023-01-01T00:00:00Z",
    "updatedAt": "2023-01-02T00:00:00Z"
  }
}
```

**错误码**:
| 错误码 | 说明 |
|-------|------|
| 1081 | 角色不存在 |
| 1082 | 角色名称已存在 |
| 1083 | 权限ID不存在 |

### 5.5 删除角色

**接口地址**: `DELETE /roles/{id}`

**请求头**: 需要Authorization，需要ROLE_MANAGE权限

**路径参数**:
```
id: int (角色ID)
```

**响应示例**:
```json
{
  "code": 200,
  "message": "删除成功"
}
```

**错误码**:
| 错误码 | 说明 |
|-------|------|
| 1091 | 角色不存在 |
| 1092 | 不能删除系统内置角色 |

## 6. 权限相关接口

### 6.1 获取权限列表

**接口地址**: `GET /permissions`

**请求头**: 需要Authorization，需要PERMISSION_MANAGE权限

**查询参数**:
```
page: int (可选，默认1)
pageSize: int (可选，默认10)
category: string (可选，权限分类筛选)
```

**响应示例**:
```json
{
  "code": 200,
  "message": "获取成功",
  "data": {
    "list": [
      {
        "id": 1,
        "code": "USER_VIEW",
        "name": "查看用户",
        "description": "查看用户列表权限",
        "category": "用户管理",
        "createdAt": "2023-01-01T00:00:00Z",
        "updatedAt": "2023-01-01T00:00:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "pageSize": 10,
      "total": 10,
      "totalPage": 1
    }
  }
}
```

### 6.2 获取权限详情

**接口地址**: `GET /permissions/{id}`

**请求头**: 需要Authorization，需要PERMISSION_MANAGE权限

**路径参数**:
```
id: int (权限ID)
```

**响应示例**:
```json
{
  "code": 200,
  "message": "获取成功",
  "data": {
    "id": 1,
    "code": "USER_VIEW",
    "name": "查看用户",
    "description": "查看用户列表权限",
    "category": "用户管理",
    "createdAt": "2023-01-01T00:00:00Z",
    "updatedAt": "2023-01-01T00:00:00Z"
  }
}
```

**错误码**:
| 错误码 | 说明 |
|-------|------|
| 1101 | 权限不存在 |

### 6.3 创建权限

**接口地址**: `POST /permissions`

**请求头**: 需要Authorization，需要PERMISSION_MANAGE权限

**请求参数**:
```json
{
  "code": "string (权限代码，唯一)",
  "name": "string (权限名称)",
  "description": "string (权限描述)",
  "category": "string (权限分类)"
}
```

**响应示例**:
```json
{
  "code": 200,
  "message": "创建成功",
  "data": {
    "id": 11,
    "code": "NEW_PERMISSION",
    "name": "新权限",
    "description": "新权限描述",
    "category": "系统管理",
    "createdAt": "2023-01-02T00:00:00Z",
    "updatedAt": "2023-01-02T00:00:00Z"
  }
}
```

**错误码**:
| 错误码 | 说明 |
|-------|------|
| 1111 | 权限代码已存在 |

### 6.4 更新权限

**接口地址**: `PUT /permissions/{id}`

**请求头**: 需要Authorization，需要PERMISSION_MANAGE权限

**路径参数**:
```
id: int (权限ID)
```

**请求参数**:
```json
{
  "code": "string (权限代码，可选)",
  "name": "string (权限名称，可选)",
  "description": "string (权限描述，可选)",
  "category": "string (权限分类，可选)"
}
```

**响应示例**:
```json
{
  "code": 200,
  "message": "更新成功",
  "data": {
    "id": 1,
    "code": "UPDATED_USER_VIEW",
    "name": "更新后的查看用户",
    "description": "更新后的查看用户列表权限",
    "category": "用户管理",
    "createdAt": "2023-01-01T00:00:00Z",
    "updatedAt": "2023-01-02T00:00:00Z"
  }
}
```

**错误码**:
| 错误码 | 说明 |
|-------|------|
| 1121 | 权限不存在 |
| 1122 | 权限代码已存在 |

### 6.5 删除权限

**接口地址**: `DELETE /permissions/{id}`

**请求头**: 需要Authorization，需要PERMISSION_MANAGE权限

**路径参数**:
```
id: int (权限ID)
```

**响应示例**:
```json
{
  "code": 200,
  "message": "删除成功"
}
```

**错误码**:
| 错误码 | 说明 |
|-------|------|
| 1131 | 权限不存在 |
| 1132 | 不能删除系统内置权限 |

## 7. 用户角色相关接口

### 7.1 获取用户角色

**接口地址**: `GET /users/{userId}/roles`

**请求头**: 需要Authorization，需要ROLE_MANAGE权限

**路径参数**:
```
userId: int (用户ID)
```

**响应示例**:
```json
{
  "code": 200,
  "message": "获取成功",
  "data": {
    "user": {
      "id": 1,
      "username": "testuser",
      "email": "test@example.com"
    },
    "roles": [
      {
        "id": 1,
        "name": "user",
        "description": "普通用户"
      }
    ]
  }
}
```

**错误码**:
| 错误码 | 说明 |
|-------|------|
| 1141 | 用户不存在 |

### 7.2 分配用户角色

**接口地址**: `POST /users/{userId}/roles`

**请求头**: 需要Authorization，需要ROLE_MANAGE权限

**路径参数**:
```
userId: int (用户ID)
```

**请求参数**:
```json
{
  "roleIds": ["int"] (角色ID列表)
}
```

**响应示例**:
```json
{
  "code": 200,
  "message": "分配成功"
}
```

**错误码**:
| 错误码 | 说明 |
|-------|------|
| 1151 | 用户不存在 |
| 1152 | 角色ID不存在 |

## 8. 错误码详细说明

### 8.1 认证相关错误码 (1000-1099)
| 错误码 | 说明 |
|-------|------|
| 1001 | 用户名已存在 |
| 1002 | 邮箱已存在 |
| 1003 | 用户名格式不正确 |
| 1004 | 邮箱格式不正确 |
| 1005 | 密码长度不符合要求 |
| 1011 | 用户名或密码错误 |
| 1012 | 用户已被禁用 |
| 1021 | refreshToken无效或已过期 |
| 1031 | Token无效或已过期 |

### 8.2 用户相关错误码 (1040-1069)
| 错误码 | 说明 |
|-------|------|
| 1041 | 用户不存在 |
| 1051 | 手机号格式不正确 |

### 8.3 角色相关错误码 (1070-1099)
| 错误码 | 说明 |
|-------|------|
| 1061 | 角色不存在 |
| 1071 | 角色名称已存在 |
| 1072 | 权限ID不存在 |
| 1081 | 角色不存在 |
| 1082 | 角色名称已存在 |
| 1083 | 权限ID不存在 |
| 1091 | 角色不存在 |
| 1092 | 不能删除系统内置角色 |

### 8.4 权限相关错误码 (1100-1139)
| 错误码 | 说明 |
|-------|------|
| 1101 | 权限不存在 |
| 1111 | 权限代码已存在 |
| 1121 | 权限不存在 |
| 1122 | 权限代码已存在 |
| 1131 | 权限不存在 |
| 1132 | 不能删除系统内置权限 |

### 8.5 用户角色相关错误码 (1140-1159)
| 错误码 | 说明 |
|-------|------|
| 1141 | 用户不存在 |
| 1151 | 用户不存在 |
| 1152 | 角色ID不存在 |
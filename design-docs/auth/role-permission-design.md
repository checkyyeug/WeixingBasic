# 角色和权限管理模型设计

## 1. 角色定义

### 基础角色
1. **普通用户 (user)**
   - 权限：查看个人信息、修改个人信息、查看公开内容
   
2. **管理员 (admin)**
   - 权限：普通用户的所有权限 + 用户管理、内容管理、系统配置
   
3. **超级管理员 (super_admin)**
   - 权限：管理员的所有权限 + 角色管理、权限分配、系统监控

### 角色属性
```javascript
{
  id: 'string',           // 角色ID
  name: 'string',         // 角色名称
  description: 'string',  // 角色描述
  permissions: [],        // 权限列表
  createdAt: 'date',      // 创建时间
  updatedAt: 'date'       // 更新时间
}
```

## 2. 权限定义

### 权限分类
1. **用户管理权限**
   - USER_VIEW: 查看用户列表
   - USER_EDIT: 编辑用户信息
   - USER_DELETE: 删除用户
   
2. **内容管理权限**
   - CONTENT_VIEW: 查看内容
   - CONTENT_CREATE: 创建内容
   - CONTENT_EDIT: 编辑内容
   - CONTENT_DELETE: 删除内容
   
3. **系统管理权限**
   - SYSTEM_CONFIG: 系统配置
   - ROLE_MANAGE: 角色管理
   - PERMISSION_MANAGE: 权限管理

### 权限属性
```javascript
{
  id: 'string',           // 权限ID
  code: 'string',         // 权限代码
  name: 'string',         // 权限名称
  description: 'string',  // 权限描述
  category: 'string',     // 权限分类
  createdAt: 'date',      // 创建时间
  updatedAt: 'date'       // 更新时间
}
```

## 3. 用户与角色关系

### 用户角色分配
- 一个用户可以拥有多个角色
- 通过用户角色关联表实现多对多关系

### 数据结构
```javascript
// 用户角色关联
{
  userId: 'string',       // 用户ID
  roleId: 'string',       // 角色ID
  assignedAt: 'date',     // 分配时间
  assignedBy: 'string'    // 分配人
}
```

## 4. 角色与权限关系

### 角色权限分配
- 一个角色可以拥有多个权限
- 通过角色权限关联表实现多对多关系

### 数据结构
```javascript
// 角色权限关联
{
  roleId: 'string',       // 角色ID
  permissionId: 'string', // 权限ID
  assignedAt: 'date',     // 分配时间
  assignedBy: 'string'    // 分配人
}
```

## 5. 前端权限控制实现

### 权限检查工具函数
```javascript
// utils/permission.js
const { getState } = require('../store/user.js');

// 检查用户是否具有指定权限
const hasPermission = (permissionCode) => {
  const { userInfo } = getState();
  
  // 获取用户所有角色
  const userRoles = userInfo.roles || [];
  
  // 检查每个角色是否具有该权限
  for (let role of userRoles) {
    const rolePermissions = role.permissions || [];
    if (rolePermissions.includes(permissionCode)) {
      return true;
    }
  }
  
  return false;
};

// 检查用户是否具有指定角色
const hasRole = (roleName) => {
  const { userInfo } = getState();
  
  const userRoles = userInfo.roles || [];
  return userRoles.some(role => role.name === roleName);
};

module.exports = {
  hasPermission,
  hasRole
};
```

### 页面级权限控制
```javascript
// pages/admin/dashboard.js
const { hasRole } = require('../../utils/permission.js');

Page({
  onLoad() {
    // 检查用户是否具有管理员角色
    if (!hasRole('admin')) {
      // 没有权限，跳转到首页或提示无权限
      wx.showToast({
        title: '无权限访问',
        icon: 'none'
      });
      
      setTimeout(() => {
        wx.switchTab({
          url: '/pages/index/index'
        });
      }, 1500);
      
      return;
    }
    
    // 有权限，继续加载页面
    this.loadDashboardData();
  },
  
  loadDashboardData() {
    // 加载管理员面板数据
  }
});
```

### 组件级权限控制
```xml
<!-- 在wxml中使用条件渲染控制组件显示 -->
<view wx:if="{{hasPermission('USER_VIEW')}}">
  <button bindtap="viewUsers">查看用户</button>
</view>

<view wx:if="{{hasPermission('CONTENT_CREATE')}}">
  <button bindtap="createContent">创建内容</button>
</view>
```

```javascript
// 在js中检查权限
const { hasPermission } = require('../../utils/permission.js');

Page({
  data: {
    permissions: {
      canViewUsers: false,
      canCreateContent: false
    }
  },
  
  onLoad() {
    // 检查权限并更新页面数据
    this.setData({
      permissions: {
        canViewUsers: hasPermission('USER_VIEW'),
        canCreateContent: hasPermission('CONTENT_CREATE')
      }
    });
  }
});
```

## 6. 后端API接口设计

### 获取用户角色接口
- URL: `/api/user/roles`
- Method: GET
- Response:
  ```json
  {
    "code": 200,
    "message": "获取成功",
    "data": [
      {
        "id": "role_1",
        "name": "admin",
        "description": "管理员",
        "permissions": ["USER_VIEW", "USER_EDIT", "CONTENT_VIEW"]
      }
    ]
  }
  ```

### 分配用户角色接口
- URL: `/api/user/roles`
- Method: POST
- Request Body:
  ```json
  {
    "userId": "string",
    "roleIds": ["string"]
  }
  ```
- Response:
  ```json
  {
    "code": 200,
    "message": "分配成功"
  }
  ```

### 获取角色权限接口
- URL: `/api/roles/permissions`
- Method: GET
- Response:
  ```json
  {
    "code": 200,
    "message": "获取成功",
    "data": [
      {
        "id": "perm_1",
        "code": "USER_VIEW",
        "name": "查看用户",
        "description": "查看用户列表权限",
        "category": "用户管理"
      }
    ]
  }
  ```

### 分配角色权限接口
- URL: `/api/roles/permissions`
- Method: POST
- Request Body:
  ```json
  {
    "roleId": "string",
    "permissionIds": ["string"]
  }
  ```
- Response:
  ```json
  {
    "code": 200,
    "message": "分配成功"
  }
  ```

## 7. 数据存储方案

### 本地存储
- 用户角色信息存储在本地，减少网络请求
- 角色权限信息缓存，提高权限检查效率

### 服务端存储
- 用户角色关联表
- 角色权限关联表
- 角色和权限的元数据表

## 8. 安全考虑

1. 权限验证：前后端都需要进行权限验证
2. 数据隔离：确保用户只能访问自己有权限的数据
3. 操作日志：记录角色和权限变更操作
4. 定期审查：定期审查用户角色和权限分配
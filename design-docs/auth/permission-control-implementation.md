# 权限控制逻辑实现方案

## 1. 路由守卫实现

### 页面级权限控制
在用户访问需要权限的页面时进行检查，如果用户没有相应权限则阻止访问。

```javascript
// utils/route-guard.js
const { hasPermission, hasRole } = require('./permission.js');

// 页面权限检查
const checkPagePermission = (pagePath, requiredPermissions) => {
  // 如果没有设置权限要求，则允许访问
  if (!requiredPermissions || requiredPermissions.length === 0) {
    return true;
  }
  
  // 检查用户是否具有所需权限
  for (let permission of requiredPermissions) {
    if (hasPermission(permission)) {
      return true;
    }
  }
  
  return false;
};

// 角色权限检查
const checkPageRole = (pagePath, requiredRoles) => {
  // 如果没有设置角色要求，则允许访问
  if (!requiredRoles || requiredRoles.length === 0) {
    return true;
  }
  
  // 检查用户是否具有所需角色
  for (let role of requiredRoles) {
    if (hasRole(role)) {
      return true;
    }
  }
  
  return false;
};

module.exports = {
  checkPagePermission,
  checkPageRole
};
```

### 应用启动时的权限检查
在应用启动时检查用户登录状态和权限。

```javascript
// app.js
const { getUserInfo } = require('./utils/auth.js');
const { setState } = require('./store/user.js');

App({
  globalData: {
    userInfo: null,
    token: null
  },
  
  onLaunch() {
    // 检查用户登录状态
    this.checkLoginStatus();
  },
  
  checkLoginStatus() {
    const { userInfo, token } = getUserInfo();
    
    if (token && userInfo) {
      // 设置全局用户状态
      setState({
        isLogin: true,
        userInfo: userInfo,
        token: token
      });
      
      // 验证token有效性
      this.verifyToken(token);
    } else {
      // 未登录，跳转到登录页面
      wx.redirectTo({
        url: '/pages/auth/login/login'
      });
    }
  },
  
  verifyToken(token) {
    // 调用后端接口验证token
    // 如果token无效，则清除用户信息并跳转到登录页面
  }
});
```

## 2. 组件级权限控制

### 条件渲染控制组件显示
根据用户权限控制组件的显示与隐藏。

```xml
<!-- 在wxml中使用条件渲染 -->
<view wx:if="{{hasPermission('USER_MANAGE')}}">
  <button bindtap="manageUsers">用户管理</button>
</view>

<view wx:if="{{hasRole('admin')}}">
  <button bindtap="systemSettings">系统设置</button>
</view>
```

### 权限控制工具函数
```javascript
// utils/permission.js
const { getState } = require('../store/user.js');

// 检查用户是否具有指定权限
const hasPermission = (permissionCode) => {
  const { userInfo } = getState();
  
  // 如果没有用户信息，认为没有权限
  if (!userInfo || !userInfo.roles) {
    return false;
  }
  
  // 检查每个角色是否具有该权限
  for (let role of userInfo.roles) {
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
  
  // 如果没有用户信息，认为没有角色
  if (!userInfo || !userInfo.roles) {
    return false;
  }
  
  return userInfo.roles.some(role => role.name === roleName);
};

// 检查用户是否具有任意一个权限
const hasAnyPermission = (permissionCodes) => {
  for (let code of permissionCodes) {
    if (hasPermission(code)) {
      return true;
    }
  }
  
  return false;
};

// 检查用户是否具有所有权限
const hasAllPermissions = (permissionCodes) => {
  for (let code of permissionCodes) {
    if (!hasPermission(code)) {
      return false;
    }
  }
  
  return true;
};

module.exports = {
  hasPermission,
  hasRole,
  hasAnyPermission,
  hasAllPermissions
};
```

### 在页面中使用权限控制
```javascript
// pages/some-page/some-page.js
const { hasPermission, hasRole } = require('../../utils/permission.js');

Page({
  data: {
    permissions: {
      canManageUsers: false,
      canEditContent: false,
      isAdmin: false
    }
  },
  
  onLoad() {
    // 检查权限并更新页面数据
    this.setData({
      permissions: {
        canManageUsers: hasPermission('USER_MANAGE'),
        canEditContent: hasPermission('CONTENT_EDIT'),
        isAdmin: hasRole('admin')
      }
    });
  },
  
  onManageUsers() {
    // 再次检查权限（防止页面状态过期）
    if (!hasPermission('USER_MANAGE')) {
      wx.showToast({
        title: '无权限操作',
        icon: 'none'
      });
      return;
    }
    
    // 执行管理用户操作
  }
});
```

## 3. 接口权限控制

### 请求拦截器中验证权限
在发起网络请求前检查权限。

```javascript
// utils/request.js
const { hasPermission } = require('./permission.js');

const request = (options) => {
  // 检查接口权限（如果需要）
  if (options.permission && !hasPermission(options.permission)) {
    return Promise.reject({
      code: 403,
      message: '无接口访问权限'
    });
  }
  
  // 继续执行请求
  return new Promise((resolve, reject) => {
    const token = wx.getStorageSync('token');
    
    wx.request({
      url: BASE_URL + options.url,
      method: options.method || 'GET',
      data: options.data || {},
      header: {
        'Content-Type': 'application/json',
        'Authorization': token ? `Bearer ${token}` : ''
      },
      success(res) {
        if (res.statusCode === 200) {
          resolve(res.data);
        } else if (res.statusCode === 401) {
          // Token过期，需要重新登录
          wx.redirectTo({
            url: '/pages/auth/login/login'
          });
        } else if (res.statusCode === 403) {
          // 无权限访问
          wx.showToast({
            title: '无权限访问',
            icon: 'none'
          });
          reject(res);
        } else {
          reject(res);
        }
      },
      fail(err) {
        reject(err);
      }
    });
  });
};

module.exports = {
  request
};
```

### 带权限检查的API调用
```javascript
// api/user.js
const { request } = require('../utils/request.js');

// 获取用户列表（需要USER_VIEW权限）
const getUserList = () => {
  return request({
    url: '/users',
    method: 'GET',
    permission: 'USER_VIEW' // 指定需要的权限
  });
};

// 删除用户（需要USER_DELETE权限）
const deleteUser = (userId) => {
  return request({
    url: `/users/${userId}`,
    method: 'DELETE',
    permission: 'USER_DELETE' // 指定需要的权限
  });
};
```

## 4. 权限更新机制

### 权限变更通知
当用户权限发生变更时，及时更新本地权限信息。

```javascript
// utils/permission-update.js
const { setState } = require('../store/user.js');
const { getUserProfile } = require('../api/user.js');

// 刷新用户权限信息
const refreshUserPermissions = () => {
  return getUserProfile().then(response => {
    if (response.code === 200) {
      // 更新本地存储的用户信息
      const userInfo = response.data;
      wx.setStorageSync('userInfo', userInfo);
      
      // 更新全局状态
      setState({
        userInfo: userInfo
      });
      
      return userInfo;
    } else {
      return Promise.reject(response);
    }
  });
};

module.exports = {
  refreshUserPermissions
};
```

### 定时刷新权限
定期刷新用户权限信息，确保权限的实时性。

```javascript
// app.js
App({
  // ... 其他代码
  
  onLaunch() {
    // 检查用户登录状态
    this.checkLoginStatus();
    
    // 启动定时刷新权限
    this.startPermissionRefresh();
  },
  
  startPermissionRefresh() {
    // 每30分钟刷新一次权限
    setInterval(() => {
      const { isLogin } = getState();
      if (isLogin) {
        refreshUserPermissions().catch(error => {
          console.error('刷新权限失败:', error);
        });
      }
    }, 30 * 60 * 1000); // 30分钟
  }
});
```

## 5. 权限控制最佳实践

### 1. 前后端双重验证
- 前端进行权限检查以提供良好的用户体验
- 后端必须进行权限验证以确保安全性

### 2. 权限缓存策略
- 合理缓存用户权限信息，减少网络请求
- 在关键操作前重新验证权限

### 3. 无权限处理
- 提供友好的无权限提示
- 引导用户到合适的页面

### 4. 权限变更通知
- 当用户权限发生变更时，及时更新界面
- 提供权限变更的提示信息

## 6. 权限控制测试

### 权限控制测试用例
```javascript
// test/permission.test.js
const { hasPermission, hasRole } = require('../utils/permission.js');
const { setState } = require('../store/user.js');

// 模拟用户状态
const mockUserState = {
  isLogin: true,
  userInfo: {
    id: 1,
    username: 'testuser',
    roles: [
      {
        id: 'role_1',
        name: 'admin',
        permissions: ['USER_VIEW', 'USER_EDIT', 'CONTENT_VIEW']
      }
    ]
  },
  token: 'mock_token'
};

describe('权限控制测试', () => {
  beforeEach(() => {
    // 设置模拟用户状态
    setState(mockUserState);
  });
  
  test('检查用户权限', () => {
    expect(hasPermission('USER_VIEW')).toBe(true);
    expect(hasPermission('USER_DELETE')).toBe(false);
  });
  
  test('检查用户角色', () => {
    expect(hasRole('admin')).toBe(true);
    expect(hasRole('user')).toBe(false);
  });
});
```

## 7. 安全考虑

### 1. 权限验证
- 所有权限检查必须在后端进行验证
- 前端权限检查仅用于改善用户体验

### 2. 数据隔离
- 确保用户只能访问自己有权限的数据
- 在数据库查询中加入权限过滤条件

### 3. 操作日志
- 记录所有涉及权限的操作
- 便于安全审计和问题追踪

### 4. 权限变更审批
- 重要权限的变更需要审批流程
- 记录权限变更的原因和审批人
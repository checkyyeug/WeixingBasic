# 角色管理功能实现方案

## 1. 角色管理页面设计

### 角色列表页面 (roles.wxml)
```xml
<!-- pages/auth/roles/roles.wxml -->
<view class="container">
  <view class="header">
    <text class="title">角色管理</text>
    <button class="add-btn" bindtap="onAddRole">添加角色</button>
  </view>
  
  <view class="role-list">
    <block wx:for="{{roles}}" wx:key="id">
      <view class="role-item">
        <view class="role-info">
          <text class="role-name">{{item.name}}</text>
          <text class="role-desc">{{item.description}}</text>
        </view>
        <view class="role-actions">
          <button class="action-btn" bindtap="onEditRole" data-role="{{item}}">编辑</button>
          <button class="action-btn delete" bindtap="onDeleteRole" data-role="{{item}}">删除</button>
        </view>
      </view>
    </block>
  </view>
</view>
```

### 角色列表页面样式 (roles.wxss)
```css
/* pages/auth/roles/roles.wxss */
.container {
  padding: 20rpx;
  background-color: #f5f5f5;
  min-height: 100vh;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30rpx;
}

.title {
  font-size: 36rpx;
  font-weight: bold;
  color: #333;
}

.add-btn {
  background-color: #007aff;
  color: #fff;
  border-radius: 8rpx;
  font-size: 28rpx;
  padding: 10rpx 20rpx;
}

.role-list {
  background-color: #fff;
  border-radius: 10rpx;
  overflow: hidden;
}

.role-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 30rpx;
  border-bottom: 1rpx solid #eee;
}

.role-item:last-child {
  border-bottom: none;
}

.role-info {
  flex: 1;
}

.role-name {
  font-size: 32rpx;
  font-weight: bold;
  color: #333;
  display: block;
  margin-bottom: 10rpx;
}

.role-desc {
  font-size: 28rpx;
  color: #666;
}

.role-actions {
  display: flex;
}

.action-btn {
  background-color: #f0f0f0;
  color: #333;
  border-radius: 8rpx;
  font-size: 24rpx;
  padding: 10rpx 20rpx;
  margin-left: 20rpx;
}

.action-btn.delete {
  background-color: #ff3b30;
  color: #fff;
}
```

### 角色列表页面逻辑 (roles.js)
```javascript
// pages/auth/roles/roles.js
const { hasPermission } = require('../../utils/permission.js');
const { getRoles, deleteRole } = require('../../api/role.js');

Page({
  data: {
    roles: []
  },

  onLoad() {
    // 检查权限
    if (!hasPermission('ROLE_MANAGE')) {
      wx.showToast({
        title: '无权限访问',
        icon: 'none'
      });
      
      setTimeout(() => {
        wx.navigateBack();
      }, 1500);
      
      return;
    }
    
    // 加载角色列表
    this.loadRoles();
  },

  loadRoles() {
    wx.showLoading({
      title: '加载中...'
    });
    
    getRoles().then(response => {
      wx.hideLoading();
      
      if (response.code === 200) {
        this.setData({
          roles: response.data
        });
      } else {
        wx.showToast({
          title: '加载失败',
          icon: 'none'
        });
      }
    }).catch(error => {
      wx.hideLoading();
      wx.showToast({
        title: '网络错误',
        icon: 'none'
      });
    });
  },

  onAddRole() {
    wx.navigateTo({
      url: '/pages/auth/roles/edit/edit'
    });
  },

  onEditRole(e) {
    const role = e.currentTarget.dataset.role;
    
    wx.navigateTo({
      url: `/pages/auth/roles/edit/edit?id=${role.id}`
    });
  },

  onDeleteRole(e) {
    const role = e.currentTarget.dataset.role;
    
    wx.showModal({
      title: '确认删除',
      content: `确定要删除角色"${role.name}"吗？`,
      success: (res) => {
        if (res.confirm) {
          this.deleteRole(role.id);
        }
      }
    });
  },

  deleteRole(roleId) {
    wx.showLoading({
      title: '删除中...'
    });
    
    deleteRole(roleId).then(response => {
      wx.hideLoading();
      
      if (response.code === 200) {
        wx.showToast({
          title: '删除成功',
          icon: 'success'
        });
        
        // 重新加载角色列表
        this.loadRoles();
      } else {
        wx.showToast({
          title: '删除失败',
          icon: 'none'
        });
      }
    }).catch(error => {
      wx.hideLoading();
      wx.showToast({
        title: '网络错误',
        icon: 'none'
      });
    });
  }
});
```

## 2. 角色编辑页面设计

### 角色编辑页面 (edit.wxml)
```xml
<!-- pages/auth/roles/edit/edit.wxml -->
<view class="container">
  <view class="header">
    <text class="title">{{isEdit ? '编辑角色' : '添加角色'}}</text>
  </view>
  
  <view class="form">
    <view class="input-group">
      <input class="input" placeholder="角色名称" value="{{roleForm.name}}" bindinput="onNameInput" />
    </view>
    
    <view class="input-group">
      <textarea class="textarea" placeholder="角色描述" value="{{roleForm.description}}" bindinput="onDescriptionInput" />
    </view>
    
    <view class="section">
      <view class="section-title">权限分配</view>
      <view class="permission-list">
        <block wx:for="{{permissions}}" wx:key="id">
          <view class="permission-item">
            <checkbox 
              checked="{{roleForm.permissions.includes(item.code)}}" 
              bindchange="onPermissionChange" 
              data-code="{{item.code}}"
            />
            <text class="permission-name">{{item.name}}</text>
            <text class="permission-desc">{{item.description}}</text>
          </view>
        </block>
      </view>
    </view>
    
    <button class="save-btn" bindtap="onSave">保存</button>
  </view>
</view>
```

### 角色编辑页面样式 (edit.wxss)
```css
/* pages/auth/roles/edit/edit.wxss */
.container {
  padding: 20rpx;
  background-color: #f5f5f5;
  min-height: 100vh;
}

.header {
  text-align: center;
  margin-bottom: 30rpx;
}

.title {
  font-size: 36rpx;
  font-weight: bold;
  color: #333;
}

.form {
  background-color: #fff;
  padding: 30rpx;
  border-radius: 10rpx;
}

.input-group {
  margin-bottom: 30rpx;
  border-bottom: 1rpx solid #eee;
}

.input {
  height: 80rpx;
  font-size: 32rpx;
  color: #333;
}

.textarea {
  width: 100%;
  height: 150rpx;
  font-size: 32rpx;
  color: #333;
  padding: 20rpx 0;
}

.section {
  margin-bottom: 40rpx;
}

.section-title {
  font-size: 32rpx;
  font-weight: bold;
  color: #333;
  margin-bottom: 20rpx;
}

.permission-list {
  max-height: 400rpx;
  overflow-y: auto;
}

.permission-item {
  display: flex;
  align-items: center;
  padding: 20rpx 0;
  border-bottom: 1rpx solid #f0f0f0;
}

.permission-item:last-child {
  border-bottom: none;
}

.permission-name {
  font-size: 28rpx;
  color: #333;
  margin: 0 20rpx;
  flex: 1;
}

.permission-desc {
  font-size: 24rpx;
  color: #999;
}

.save-btn {
  background-color: #007aff;
  color: #fff;
  border-radius: 10rpx;
  font-size: 36rpx;
  height: 80rpx;
  line-height: 80rpx;
}
```

### 角色编辑页面逻辑 (edit.js)
```javascript
// pages/auth/roles/edit/edit.js
const { hasPermission } = require('../../../utils/permission.js');
const { getRole, createRole, updateRole } = require('../../../api/role.js');
const { getPermissions } = require('../../../api/permission.js');

Page({
  data: {
    isEdit: false,
    roleId: '',
    roleForm: {
      name: '',
      description: '',
      permissions: []
    },
    permissions: []
  },

  onLoad(options) {
    // 检查权限
    if (!hasPermission('ROLE_MANAGE')) {
      wx.showToast({
        title: '无权限访问',
        icon: 'none'
      });
      
      setTimeout(() => {
        wx.navigateBack();
      }, 1500);
      
      return;
    }
    
    // 加载权限列表
    this.loadPermissions();
    
    // 如果是编辑模式，加载角色信息
    if (options.id) {
      this.setData({
        isEdit: true,
        roleId: options.id
      });
      
      this.loadRole(options.id);
    }
  },

  loadPermissions() {
    getPermissions().then(response => {
      if (response.code === 200) {
        this.setData({
          permissions: response.data
        });
      }
    });
  },

  loadRole(roleId) {
    wx.showLoading({
      title: '加载中...'
    });
    
    getRole(roleId).then(response => {
      wx.hideLoading();
      
      if (response.code === 200) {
        this.setData({
          roleForm: {
            name: response.data.name,
            description: response.data.description,
            permissions: response.data.permissions.map(p => p.code)
          }
        });
      } else {
        wx.showToast({
          title: '加载失败',
          icon: 'none'
        });
      }
    }).catch(error => {
      wx.hideLoading();
      wx.showToast({
        title: '网络错误',
        icon: 'none'
      });
    });
  },

  onNameInput(e) {
    this.setData({
      'roleForm.name': e.detail.value
    });
  },

  onDescriptionInput(e) {
    this.setData({
      'roleForm.description': e.detail.value
    });
  },

  onPermissionChange(e) {
    const code = e.currentTarget.dataset.code;
    const checked = e.detail.value;
    
    let permissions = [...this.data.roleForm.permissions];
    
    if (checked) {
      // 添加权限
      if (!permissions.includes(code)) {
        permissions.push(code);
      }
    } else {
      // 移除权限
      permissions = permissions.filter(p => p !== code);
    }
    
    this.setData({
      'roleForm.permissions': permissions
    });
  },

  onSave() {
    const { isEdit, roleId, roleForm } = this.data;
    
    // 表单验证
    if (!roleForm.name) {
      wx.showToast({
        title: '请输入角色名称',
        icon: 'none'
      });
      return;
    }
    
    wx.showLoading({
      title: '保存中...'
    });
    
    if (isEdit) {
      // 更新角色
      updateRole(roleId, roleForm).then(response => {
        wx.hideLoading();
        
        if (response.code === 200) {
          wx.showToast({
            title: '更新成功',
            icon: 'success'
          });
          
          setTimeout(() => {
            wx.navigateBack();
          }, 1500);
        } else {
          wx.showToast({
            title: '更新失败',
            icon: 'none'
          });
        }
      }).catch(error => {
        wx.hideLoading();
        wx.showToast({
          title: '网络错误',
          icon: 'none'
        });
      });
    } else {
      // 创建角色
      createRole(roleForm).then(response => {
        wx.hideLoading();
        
        if (response.code === 200) {
          wx.showToast({
            title: '创建成功',
            icon: 'success'
          });
          
          setTimeout(() => {
            wx.navigateBack();
          }, 1500);
        } else {
          wx.showToast({
            title: '创建失败',
            icon: 'none'
          });
        }
      }).catch(error => {
        wx.hideLoading();
        wx.showToast({
          title: '网络错误',
          icon: 'none'
        });
      });
    }
  }
});
```

## 3. 角色管理API接口

### 角色相关API (api/role.js)
```javascript
// api/role.js
const { request } = require('../utils/request.js');

// 获取角色列表
const getRoles = () => {
  return request({
    url: '/roles',
    method: 'GET'
  });
};

// 获取角色详情
const getRole = (roleId) => {
  return request({
    url: `/roles/${roleId}`,
    method: 'GET'
  });
};

// 创建角色
const createRole = (roleData) => {
  return request({
    url: '/roles',
    method: 'POST',
    data: roleData
  });
};

// 更新角色
const updateRole = (roleId, roleData) => {
  return request({
    url: `/roles/${roleId}`,
    method: 'PUT',
    data: roleData
  });
};

// 删除角色
const deleteRole = (roleId) => {
  return request({
    url: `/roles/${roleId}`,
    method: 'DELETE'
  });
};

module.exports = {
  getRoles,
  getRole,
  createRole,
  updateRole,
  deleteRole
};
```

## 4. 权限管理API接口

### 权限相关API (api/permission.js)
```javascript
// api/permission.js
const { request } = require('../utils/request.js');

// 获取权限列表
const getPermissions = () => {
  return request({
    url: '/permissions',
    method: 'GET'
  });
};

// 获取权限详情
const getPermission = (permissionId) => {
  return request({
    url: `/permissions/${permissionId}`,
    method: 'GET'
  });
};

// 创建权限
const createPermission = (permissionData) => {
  return request({
    url: '/permissions',
    method: 'POST',
    data: permissionData
  });
};

// 更新权限
const updatePermission = (permissionId, permissionData) => {
  return request({
    url: `/permissions/${permissionId}`,
    method: 'PUT',
    data: permissionData
  });
};

// 删除权限
const deletePermission = (permissionId) => {
  return request({
    url: `/permissions/${permissionId}`,
    method: 'DELETE'
  });
};

module.exports = {
  getPermissions,
  getPermission,
  createPermission,
  updatePermission,
  deletePermission
};
```

## 5. 用户角色分配功能

### 用户角色分配页面设计

#### 用户角色分配页面 (user-roles.wxml)
```xml
<!-- pages/auth/roles/user-roles.wxml -->
<view class="container">
  <view class="header">
    <text class="title">用户角色分配</text>
  </view>
  
  <view class="user-info">
    <text class="username">{{user.username}}</text>
    <text class="email">{{user.email}}</text>
  </view>
  
  <view class="section">
    <view class="section-title">角色分配</view>
    <view class="role-list">
      <block wx:for="{{roles}}" wx:key="id">
        <view class="role-item">
          <checkbox 
            checked="{{userRoles.includes(item.id)}}" 
            bindchange="onRoleChange" 
            data-role="{{item}}"
          />
          <view class="role-info">
            <text class="role-name">{{item.name}}</text>
            <text class="role-desc">{{item.description}}</text>
          </view>
        </view>
      </block>
    </view>
  </view>
  
  <button class="save-btn" bindtap="onSave">保存</button>
</view>
```

### 用户角色分配页面逻辑 (user-roles.js)
```javascript
// pages/auth/roles/user-roles.js
const { hasPermission } = require('../../utils/permission.js');
const { getUserRoles, assignUserRoles } = require('../../api/user-role.js');
const { getRoles } = require('../../api/role.js');

Page({
  data: {
    userId: '',
    user: {},
    roles: [],
    userRoles: [] // 用户当前拥有的角色ID列表
  },

  onLoad(options) {
    // 检查权限
    if (!hasPermission('ROLE_MANAGE')) {
      wx.showToast({
        title: '无权限访问',
        icon: 'none'
      });
      
      setTimeout(() => {
        wx.navigateBack();
      }, 1500);
      
      return;
    }
    
    if (options.userId) {
      this.setData({
        userId: options.userId
      });
      
      // 加载用户信息和角色信息
      this.loadUserData(options.userId);
    }
  },

  loadUserData(userId) {
    wx.showLoading({
      title: '加载中...'
    });
    
    // 并行加载用户角色和所有角色
    Promise.all([
      getUserRoles(userId),
      getRoles()
    ]).then(([userRolesRes, rolesRes]) => {
      wx.hideLoading();
      
      if (userRolesRes.code === 200 && rolesRes.code === 200) {
        // 提取用户角色ID列表
        const userRoleIds = userRolesRes.data.map(role => role.id);
        
        this.setData({
          user: userRolesRes.data.user,
          roles: rolesRes.data,
          userRoles: userRoleIds
        });
      } else {
        wx.showToast({
          title: '加载失败',
          icon: 'none'
        });
      }
    }).catch(error => {
      wx.hideLoading();
      wx.showToast({
        title: '网络错误',
        icon: 'none'
      });
    });
  },

  onRoleChange(e) {
    const role = e.currentTarget.dataset.role;
    const checked = e.detail.value;
    
    let userRoles = [...this.data.userRoles];
    
    if (checked) {
      // 添加角色
      if (!userRoles.includes(role.id)) {
        userRoles.push(role.id);
      }
    } else {
      // 移除角色
      userRoles = userRoles.filter(roleId => roleId !== role.id);
    }
    
    this.setData({
      userRoles: userRoles
    });
  },

  onSave() {
    const { userId, userRoles } = this.data;
    
    wx.showLoading({
      title: '保存中...'
    });
    
    assignUserRoles(userId, userRoles).then(response => {
      wx.hideLoading();
      
      if (response.code === 200) {
        wx.showToast({
          title: '保存成功',
          icon: 'success'
        });
        
        setTimeout(() => {
          wx.navigateBack();
        }, 1500);
      } else {
        wx.showToast({
          title: '保存失败',
          icon: 'none'
        });
      }
    }).catch(error => {
      wx.hideLoading();
      wx.showToast({
        title: '网络错误',
        icon: 'none'
      });
    });
  }
});
```

## 6. 用户角色管理API接口

### 用户角色相关API (api/user-role.js)
```javascript
// api/user-role.js
const { request } = require('../utils/request.js');

// 获取用户角色
const getUserRoles = (userId) => {
  return request({
    url: `/users/${userId}/roles`,
    method: 'GET'
  });
};

// 分配用户角色
const assignUserRoles = (userId, roleIds) => {
  return request({
    url: `/users/${userId}/roles`,
    method: 'POST',
    data: {
      roleIds: roleIds
    }
  });
};

module.exports = {
  getUserRoles,
  assignUserRoles
};
```

## 7. 后端API接口设计

### 角色管理接口
1. 获取角色列表：GET /api/roles
2. 获取角色详情：GET /api/roles/:id
3. 创建角色：POST /api/roles
4. 更新角色：PUT /api/roles/:id
5. 删除角色：DELETE /api/roles/:id

### 权限管理接口
1. 获取权限列表：GET /api/permissions
2. 获取权限详情：GET /api/permissions/:id
3. 创建权限：POST /api/permissions
4. 更新权限：PUT /api/permissions/:id
5. 删除权限：DELETE /api/permissions/:id

### 用户角色管理接口
1. 获取用户角色：GET /api/users/:userId/roles
2. 分配用户角色：POST /api/users/:userId/roles

## 8. 安全考虑

1. 权限验证：确保只有具有ROLE_MANAGE权限的用户才能访问角色管理功能
2. 数据验证：对所有输入数据进行验证，防止恶意数据
3. 操作日志：记录所有角色和权限的变更操作
4. 防止越权：确保用户只能管理自己有权限的角色
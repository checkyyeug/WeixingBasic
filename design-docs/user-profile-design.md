# 用户信息管理功能设计方案

## 1. 用户个人信息页面设计

### 个人信息页面结构 (profile.wxml)
```xml
<!-- pages/user/profile/profile.wxml -->
<view class="container">
  <view class="header">
    <text class="title">个人信息</text>
  </view>
  
  <view class="profile-card">
    <view class="avatar-section" bindtap="onAvatarTap">
      <image class="avatar" src="{{userInfo.avatar || '/images/default-avatar.png'}}" mode="aspectFill"></image>
      <text class="change-avatar">点击更换头像</text>
    </view>
    
    <view class="info-item">
      <text class="label">用户名</text>
      <text class="value">{{userInfo.username}}</text>
    </view>
    
    <view class="info-item">
      <text class="label">邮箱</text>
      <text class="value">{{userInfo.email}}</text>
    </view>
    
    <view class="info-item">
      <text class="label">注册时间</text>
      <text class="value">{{userInfo.createdAt}}</text>
    </view>
    
    <view class="info-item">
      <text class="label">角色</text>
      <view class="roles">
        <block wx:for="{{userInfo.roles}}" wx:key="id">
          <text class="role-tag">{{item.name}}</text>
        </block>
      </view>
    </view>
  </view>
  
  <button class="edit-btn" bindtap="onEditProfile">编辑资料</button>
  <button class="logout-btn" bindtap="onLogout">退出登录</button>
</view>
```

### 个人信息页面样式 (profile.wxss)
```css
/* pages/user/profile/profile.wxss */
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

.profile-card {
  background-color: #fff;
  border-radius: 10rpx;
  padding: 40rpx;
  margin-bottom: 30rpx;
}

.avatar-section {
  text-align: center;
  margin-bottom: 40rpx;
}

.avatar {
  width: 120rpx;
  height: 120rpx;
  border-radius: 50%;
  border: 2rpx solid #eee;
}

.change-avatar {
  display: block;
  font-size: 24rpx;
  color: #007aff;
  margin-top: 20rpx;
}

.info-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20rpx 0;
  border-bottom: 1rpx solid #f0f0f0;
}

.info-item:last-child {
  border-bottom: none;
}

.label {
  font-size: 28rpx;
  color: #666;
}

.value {
  font-size: 28rpx;
  color: #333;
}

.roles {
  display: flex;
}

.role-tag {
  background-color: #007aff;
  color: #fff;
  font-size: 20rpx;
  padding: 4rpx 12rpx;
  border-radius: 6rpx;
  margin-right: 10rpx;
}

.edit-btn, .logout-btn {
  width: 100%;
  margin-bottom: 20rpx;
  border-radius: 10rpx;
  font-size: 32rpx;
  height: 80rpx;
  line-height: 80rpx;
}

.edit-btn {
  background-color: #007aff;
  color: #fff;
}

.logout-btn {
  background-color: #ff3b30;
  color: #fff;
}
```

### 个人信息页面逻辑 (profile.js)
```javascript
// pages/user/profile/profile.js
const { getState } = require('../../store/user.js');
const { clearUserInfo } = require('../../utils/auth.js');

Page({
  data: {
    userInfo: {}
  },

  onLoad() {
    // 获取用户信息
    const { userInfo } = getState();
    this.setData({
      userInfo: userInfo
    });
  },

  onAvatarTap() {
    // 选择图片并上传头像
    wx.chooseImage({
      count: 1,
      sizeType: ['compressed'],
      sourceType: ['album', 'camera'],
      success: (res) => {
        const tempFilePath = res.tempFilePaths[0];
        this.uploadAvatar(tempFilePath);
      }
    });
  },

  uploadAvatar(filePath) {
    wx.showLoading({
      title: '上传中...'
    });
    
    // 上传头像到服务器
    wx.uploadFile({
      url: 'https://your-api-domain.com/api/user/avatar',
      filePath: filePath,
      name: 'avatar',
      header: {
        'Authorization': `Bearer ${getState().token}`
      },
      success: (res) => {
        wx.hideLoading();
        
        if (res.statusCode === 200) {
          const data = JSON.parse(res.data);
          if (data.code === 200) {
            // 更新用户信息
            const userInfo = this.data.userInfo;
            userInfo.avatar = data.data.avatarUrl;
            
            this.setData({
              userInfo: userInfo
            });
            
            wx.showToast({
              title: '上传成功',
              icon: 'success'
            });
          } else {
            wx.showToast({
              title: '上传失败',
              icon: 'none'
            });
          }
        } else {
          wx.showToast({
            title: '上传失败',
            icon: 'none'
          });
        }
      },
      fail: () => {
        wx.hideLoading();
        wx.showToast({
          title: '网络错误',
          icon: 'none'
        });
      }
    });
  },

  onEditProfile() {
    wx.navigateTo({
      url: '/pages/user/profile/edit/edit'
    });
  },

  onLogout() {
    wx.showModal({
      title: '确认退出',
      content: '确定要退出登录吗？',
      success: (res) => {
        if (res.confirm) {
          // 清除用户信息
          clearUserInfo();
          
          // 跳转到登录页面
          wx.redirectTo({
            url: '/pages/auth/login/login'
          });
        }
      }
    });
  }
});
```

## 2. 编辑个人信息页面设计

### 编辑个人信息页面结构 (edit.wxml)
```xml
<!-- pages/user/profile/edit/edit.wxml -->
<view class="container">
  <view class="header">
    <text class="title">编辑资料</text>
  </view>
  
  <view class="form">
    <view class="input-group">
      <text class="label">用户名</text>
      <input class="input" value="{{profileForm.username}}" bindinput="onUsernameInput" />
    </view>
    
    <view class="input-group">
      <text class="label">邮箱</text>
      <input class="input" value="{{profileForm.email}}" bindinput="onEmailInput" />
    </view>
    
    <view class="input-group">
      <text class="label">昵称</text>
      <input class="input" value="{{profileForm.nickname}}" bindinput="onNicknameInput" />
    </view>
    
    <view class="input-group">
      <text class="label">手机号</text>
      <input class="input" value="{{profileForm.phone}}" bindinput="onPhoneInput" />
    </view>
    
    <view class="input-group">
      <text class="label">个人简介</text>
      <textarea class="textarea" value="{{profileForm.bio}}" bindinput="onBioInput" />
    </view>
  </view>
  
  <button class="save-btn" bindtap="onSave">保存</button>
</view>
```

### 编辑个人信息页面样式 (edit.wxss)
```css
/* pages/user/profile/edit/edit.wxss */
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
  border-radius: 10rpx;
  padding: 0 30rpx;
}

.input-group {
  padding: 20rpx 0;
  border-bottom: 1rpx solid #f0f0f0;
}

.input-group:last-child {
  border-bottom: none;
}

.label {
  display: block;
  font-size: 28rpx;
  color: #666;
  margin-bottom: 10rpx;
}

.input {
  height: 60rpx;
  font-size: 32rpx;
  color: #333;
}

.textarea {
  width: 100%;
  height: 120rpx;
  font-size: 32rpx;
  color: #333;
  padding: 20rpx 0;
}

.save-btn {
  margin-top: 50rpx;
  background-color: #007aff;
  color: #fff;
  border-radius: 10rpx;
  font-size: 36rpx;
  height: 80rpx;
  line-height: 80rpx;
}
```

### 编辑个人信息页面逻辑 (edit.js)
```javascript
// pages/user/profile/edit/edit.js
const { getState, setState } = require('../../../store/user.js');
const { updateProfile } = require('../../../api/user.js');

Page({
  data: {
    profileForm: {
      username: '',
      email: '',
      nickname: '',
      phone: '',
      bio: ''
    }
  },

  onLoad() {
    // 获取当前用户信息
    const { userInfo } = getState();
    
    this.setData({
      profileForm: {
        username: userInfo.username || '',
        email: userInfo.email || '',
        nickname: userInfo.nickname || '',
        phone: userInfo.phone || '',
        bio: userInfo.bio || ''
      }
    });
  },

  onUsernameInput(e) {
    this.setData({
      'profileForm.username': e.detail.value
    });
  },

  onEmailInput(e) {
    this.setData({
      'profileForm.email': e.detail.value
    });
  },

  onNicknameInput(e) {
    this.setData({
      'profileForm.nickname': e.detail.value
    });
  },

  onPhoneInput(e) {
    this.setData({
      'profileForm.phone': e.detail.value
    });
  },

  onBioInput(e) {
    this.setData({
      'profileForm.bio': e.detail.value
    });
  },

  onSave() {
    const { profileForm } = this.data;
    
    // 表单验证
    if (!profileForm.username) {
      wx.showToast({
        title: '请输入用户名',
        icon: 'none'
      });
      return;
    }
    
    if (!profileForm.email) {
      wx.showToast({
        title: '请输入邮箱',
        icon: 'none'
      });
      return;
    }
    
    wx.showLoading({
      title: '保存中...'
    });
    
    // 更新用户信息
    updateProfile(profileForm).then(response => {
      wx.hideLoading();
      
      if (response.code === 200) {
        // 更新本地存储的用户信息
        const { userInfo } = getState();
        const updatedUserInfo = {
          ...userInfo,
          ...profileForm
        };
        
        setState({
          userInfo: updatedUserInfo
        });
        
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

## 3. 用户信息管理API接口

### 用户相关API (api/user.js)
```javascript
// api/user.js
const { request } = require('../utils/request.js');

// 获取用户信息
const getUserProfile = () => {
  return request({
    url: '/user/profile',
    method: 'GET'
  });
};

// 更新用户信息
const updateProfile = (profileData) => {
  return request({
    url: '/user/profile',
    method: 'PUT',
    data: profileData
  });
};

// 更新用户头像
const updateAvatar = (avatarFile) => {
  return request({
    url: '/user/avatar',
    method: 'POST',
    data: {
      avatar: avatarFile
    }
  });
};

module.exports = {
  getUserProfile,
  updateProfile,
  updateAvatar
};
```

## 4. 用户信息数据结构

### 用户信息对象
```javascript
{
  id: 'string',           // 用户ID
  username: 'string',     // 用户名
  email: 'string',        // 邮箱
  nickname: 'string',     // 昵称
  phone: 'string',        // 手机号
  avatar: 'string',       // 头像URL
  bio: 'string',          // 个人简介
  roles: [],              // 用户角色列表
  createdAt: 'date',      // 注册时间
  updatedAt: 'date'       // 更新时间
}
```

## 5. 后端API接口设计

### 用户信息管理接口
1. 获取用户信息：GET /api/user/profile
2. 更新用户信息：PUT /api/user/profile
3. 更新用户头像：POST /api/user/avatar

## 6. 安全考虑

1. 数据验证：对用户输入的所有信息进行验证
2. 权限控制：确保用户只能修改自己的信息
3. 文件上传安全：对上传的头像文件进行安全检查
4. 敏感信息保护：不返回用户的敏感信息（如密码）
5. 输入过滤：防止XSS攻击，对用户输入进行过滤
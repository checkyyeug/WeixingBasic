# 用户登录页面设计

## 页面结构 (login.wxml)
```xml
<!--login.wxml-->
<view class="container">
  <view class="header">
    <text class="title">用户登录</text>
  </view>
  
  <view class="form">
    <view class="input-group">
      <input class="input" placeholder="用户名/邮箱" bindinput="onUsernameInput" />
    </view>
    
    <view class="input-group">
      <input class="input" password placeholder="密码" bindinput="onPasswordInput" />
    </view>
    
    <button class="login-btn" bindtap="onLogin">登录</button>
    
    <view class="register-link">
      <text>还没有账户？</text>
      <navigator url="../register/register" class="link">立即注册</navigator>
    </view>
    
    <view class="forgot-password">
      <navigator url="../forgot/forgot" class="link">忘记密码？</navigator>
    </view>
  </view>
</view>
```

## 页面样式 (login.wxss)
```css
/* login.wxss */
.container {
  padding: 40rpx;
  background-color: #f5f5f5;
  min-height: 100vh;
}

.header {
  text-align: center;
  margin-bottom: 60rpx;
}

.title {
  font-size: 48rpx;
  font-weight: bold;
  color: #333;
}

.form {
  background-color: #fff;
  padding: 40rpx;
  border-radius: 10rpx;
  box-shadow: 0 2rpx 10rpx rgba(0, 0, 0, 0.1);
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

.login-btn {
  margin-top: 50rpx;
  background-color: #007aff;
  color: #fff;
  border-radius: 10rpx;
  font-size: 36rpx;
  height: 80rpx;
  line-height: 80rpx;
}

.login-btn:active {
  background-color: #0062cc;
}

.register-link, .forgot-password {
  text-align: center;
  margin-top: 30rpx;
  font-size: 28rpx;
  color: #666;
}

.forgot-password {
  margin-top: 20rpx;
}

.link {
  color: #007aff;
  margin-left: 10rpx;
}
```

## 页面配置 (login.json)
```json
{
  "usingComponents": {}
}
```

## 页面逻辑 (login.js)
```javascript
// login.js
const { login } = require('../../utils/auth.js');
const { saveUserInfo } = require('../../utils/auth.js');

Page({
  data: {
    username: '',
    password: ''
  },

  onUsernameInput(e) {
    this.setData({
      username: e.detail.value
    });
  },

  onPasswordInput(e) {
    this.setData({
      password: e.detail.value
    });
  },

  onLogin() {
    const { username, password } = this.data;
    
    // 表单验证
    if (!username) {
      wx.showToast({
        title: '请输入用户名或邮箱',
        icon: 'none'
      });
      return;
    }
    
    if (!password) {
      wx.showToast({
        title: '请输入密码',
        icon: 'none'
      });
      return;
    }
    
    // 调用登录接口
    this.loginUser({
      username,
      password
    });
  },

  loginUser(credentials) {
    // 这里应该调用实际的登录接口
    // 暂时用模拟代码演示
    wx.showLoading({
      title: '登录中...'
    });
    
    // 模拟网络请求
    setTimeout(() => {
      wx.hideLoading();
      
      // 模拟登录成功
      const userInfo = {
        id: 1,
        username: credentials.username,
        email: 'user@example.com',
        role: 'user'
      };
      
      const token = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...'; // 模拟JWT token
      
      // 保存用户信息到本地存储
      saveUserInfo(userInfo, token);
      
      wx.showToast({
        title: '登录成功',
        icon: 'success'
      });
      
      // 登录成功后跳转到首页
      setTimeout(() => {
        wx.switchTab({
          url: '../../index/index'
        });
      }, 1500);
    }, 1500);
  }
});
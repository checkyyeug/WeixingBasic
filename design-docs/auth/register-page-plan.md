# 用户注册页面设计

## 页面结构 (register.wxml)
```xml
<!--register.wxml-->
<view class="container">
  <view class="header">
    <text class="title">用户注册</text>
  </view>
  
  <view class="form">
    <view class="input-group">
      <input class="input" placeholder="用户名" bindinput="onUsernameInput" />
    </view>
    
    <view class="input-group">
      <input class="input" placeholder="邮箱" bindinput="onEmailInput" />
    </view>
    
    <view class="input-group">
      <input class="input" password placeholder="密码" bindinput="onPasswordInput" />
    </view>
    
    <view class="input-group">
      <input class="input" password placeholder="确认密码" bindinput="onConfirmPasswordInput" />
    </view>
    
    <button class="register-btn" bindtap="onRegister">注册</button>
    
    <view class="login-link">
      <text>已有账户？</text>
      <navigator url="../login/login" class="link">立即登录</navigator>
    </view>
  </view>
</view>
```

## 页面样式 (register.wxss)
```css
/* register.wxss */
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

.register-btn {
  margin-top: 50rpx;
  background-color: #007aff;
  color: #fff;
  border-radius: 10rpx;
  font-size: 36rpx;
  height: 80rpx;
  line-height: 80rpx;
}

.register-btn:active {
  background-color: #0062cc;
}

.login-link {
  text-align: center;
  margin-top: 30rpx;
  font-size: 28rpx;
  color: #666;
}

.link {
  color: #007aff;
  margin-left: 10rpx;
}
```

## 页面配置 (register.json)
```json
{
  "usingComponents": {}
}
```

## 页面逻辑 (register.js)
```javascript
// register.js
Page({
  data: {
    username: '',
    email: '',
    password: '',
    confirmPassword: ''
  },

  onUsernameInput(e) {
    this.setData({
      username: e.detail.value
    });
  },

  onEmailInput(e) {
    this.setData({
      email: e.detail.value
    });
  },

  onPasswordInput(e) {
    this.setData({
      password: e.detail.value
    });
  },

  onConfirmPasswordInput(e) {
    this.setData({
      confirmPassword: e.detail.value
    });
  },

  onRegister() {
    const { username, email, password, confirmPassword } = this.data;
    
    // 表单验证
    if (!username) {
      wx.showToast({
        title: '请输入用户名',
        icon: 'none'
      });
      return;
    }
    
    if (!email) {
      wx.showToast({
        title: '请输入邮箱',
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
    
    if (password !== confirmPassword) {
      wx.showToast({
        title: '两次输入的密码不一致',
        icon: 'none'
      });
      return;
    }
    
    // 调用注册接口
    this.registerUser({
      username,
      email,
      password
    });
  },

  registerUser(userInfo) {
    // 这里应该调用实际的注册接口
    // 暂时用模拟代码演示
    wx.showLoading({
      title: '注册中...'
    });
    
    // 模拟网络请求
    setTimeout(() => {
      wx.hideLoading();
      
      // 模拟注册成功
      wx.showToast({
        title: '注册成功',
        icon: 'success'
      });
      
      // 注册成功后跳转到登录页面
      setTimeout(() => {
        wx.redirectTo({
          url: '../login/login'
        });
      }, 1500);
    }, 1500);
  }
});
# 用户登录功能实现方案

## 1. 登录页面逻辑实现说明

在登录页面(login.js)中，我们需要：

1. 收集用户输入的登录信息（用户名/邮箱、密码）
2. 进行表单验证
3. 调用登录接口
4. 处理登录结果
5. 保存用户信息和token到本地存储
6. 跳转到适当页面

具体的实现代码已经在login-page-plan.md中描述。

## 2. 登录功能扩展

### 自动登录检查
在应用启动时检查用户是否已登录：

```javascript
// app.js
App({
  onLaunch() {
    // 检查用户是否已登录
    this.checkLoginStatus();
  },
  
  checkLoginStatus() {
    const { token } = require('./utils/auth.js').getUserInfo();
    
    if (token) {
      // 验证token是否有效
      this.verifyToken(token);
    } else {
      // 未登录，跳转到登录页面
      wx.redirectTo({
        url: '/pages/auth/login/login'
      });
    }
  },
  
  verifyToken(token) {
    // 调用验证token的接口
    // 如果token有效，则继续正常使用
    // 如果token无效，则清除用户信息并跳转到登录页面
  }
});
```

### Token刷新机制
实现token自动刷新机制：

```javascript
// utils/auth.js
const refreshToken = () => {
  const refreshToken = wx.getStorageSync('refreshToken');
  
  if (!refreshToken) {
    // 没有refreshToken，需要重新登录
    return Promise.reject('No refresh token');
  }
  
  return request({
    url: '/auth/refresh',
    method: 'POST',
    data: {
      refreshToken
    }
  }).then(response => {
    // 保存新的token
    wx.setStorageSync('token', response.data.token);
    wx.setStorageSync('refreshToken', response.data.refreshToken);
    
    return response.data.token;
  });
};

// 在请求拦截器中处理token过期
const requestWithAuth = (options) => {
  return request(options).catch(error => {
    if (error.statusCode === 401) {
      // token过期，尝试刷新
      return refreshToken().then(newToken => {
        // 使用新token重新发起请求
        options.header.Authorization = `Bearer ${newToken}`;
        return request(options);
      }).catch(() => {
        // 刷新失败，需要重新登录
        clearUserInfo();
        wx.redirectTo({
          url: '/pages/auth/login/login'
        });
        
        return Promise.reject('Token refresh failed');
      });
    }
    
    return Promise.reject(error);
  });
};
```

## 3. 登出功能实现

```javascript
// utils/auth.js
const logout = () => {
  // 清除本地存储的用户信息
  clearUserInfo();
  
  // 跳转到登录页面
  wx.redirectTo({
    url: '/pages/auth/login/login'
  });
};

// 在页面中使用登出功能
// page.js
Page({
  onLogout() {
    // 确认登出
    wx.showModal({
      title: '确认登出',
      content: '确定要退出登录吗？',
      success(res) {
        if (res.confirm) {
          // 调用登出函数
          logout();
        }
      }
    });
  }
});
```

## 4. 后端API接口设计

### 登录接口
- URL: `/api/auth/login`
- Method: POST
- Request Body:
  ```json
  {
    "username": "string",
    "password": "string"
  }
  ```
- Response:
  ```json
  {
    "code": 200,
    "message": "登录成功",
    "data": {
      "userInfo": {},
      "token": "string",
      "refreshToken": "string"
    }
  }
  ```

### Token验证接口
- URL: `/api/auth/verify`
- Method: POST
- Request Body:
  ```json
  {
    "token": "string"
  }
  ```
- Response:
  ```json
  {
    "code": 200,
    "message": "Token有效",
    "data": {
      "valid": true
    }
  }
  ```

### Token刷新接口
- URL: `/api/auth/refresh`
- Method: POST
- Request Body:
  ```json
  {
    "refreshToken": "string"
  }
  ```
- Response:
  ```json
  {
    "code": 200,
    "message": "Token刷新成功",
    "data": {
      "token": "string",
      "refreshToken": "string"
    }
  }
  ```

## 5. 安全考虑

1. Token过期处理：设置合理的token过期时间
2. Refresh Token机制：使用refresh token来刷新访问token
3. HTTPS通信：确保数据传输安全
4. 密码加密：后端需要使用bcrypt等加密算法对密码进行加密存储
5. 输入验证：前后端都需要进行输入验证
6. 防止暴力破解：限制登录尝试次数
7. 日志记录：记录登录日志，便于安全审计
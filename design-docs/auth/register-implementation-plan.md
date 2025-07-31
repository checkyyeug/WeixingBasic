# 用户注册功能实现方案

## 1. 网络请求封装 (utils/request.js)

```javascript
// utils/request.js
const BASE_URL = 'https://your-api-domain.com/api';

const request = (options) => {
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

## 2. 用户认证工具函数 (utils/auth.js)

```javascript
// utils/auth.js
const { request } = require('./request');

// 用户注册
const register = (userInfo) => {
  return request({
    url: '/auth/register',
    method: 'POST',
    data: userInfo
  });
};

// 用户登录
const login = (credentials) => {
  return request({
    url: '/auth/login',
    method: 'POST',
    data: credentials
  });
};

// 保存用户信息到本地存储
const saveUserInfo = (userInfo, token) => {
  wx.setStorageSync('userInfo', userInfo);
  wx.setStorageSync('token', token);
};

// 获取本地存储的用户信息
const getUserInfo = () => {
  return {
    userInfo: wx.getStorageSync('userInfo'),
    token: wx.getStorageSync('token')
  };
};

// 清除用户信息
const clearUserInfo = () => {
  wx.removeStorageSync('userInfo');
  wx.removeStorageSync('token');
};

module.exports = {
  register,
  login,
  saveUserInfo,
  getUserInfo,
  clearUserInfo
};
```

## 3. 用户注册API接口 (api/auth.js)

```javascript
// api/auth.js
const { request } = require('../utils/request');

// 用户注册接口
const registerUser = (userInfo) => {
  return request({
    url: '/auth/register',
    method: 'POST',
    data: userInfo
  });
};

// 用户登录接口
const loginUser = (credentials) => {
  return request({
    url: '/auth/login',
    method: 'POST',
    data: credentials
  });
};

// 获取用户信息接口
const getUserProfile = () => {
  return request({
    url: '/user/profile',
    method: 'GET'
  });
};

module.exports = {
  registerUser,
  loginUser,
  getUserProfile
};
```

## 4. 用户状态管理 (store/user.js)

```javascript
// store/user.js
const { getUserInfo } = require('../utils/auth');

// 初始化用户状态
const initState = () => {
  const { userInfo, token } = getUserInfo();
  return {
    isLogin: !!token,
    userInfo: userInfo || {},
    token: token || ''
  };
};

// 用户状态
let userState = initState();

// 获取用户状态
const getState = () => {
  return userState;
};

// 更新用户状态
const setState = (newState) => {
  userState = {
    ...userState,
    ...newState
  };
};

// 重置用户状态
const resetState = () => {
  userState = initState();
};

module.exports = {
  getState,
  setState,
  resetState
};
```

## 5. 注册页面逻辑实现说明

在注册页面(register.js)中，我们需要：

1. 收集用户输入的注册信息（用户名、邮箱、密码）
2. 进行表单验证
3. 调用注册接口
4. 处理注册结果

具体的实现代码已经在register-page-plan.md中描述。

## 6. 后端API接口设计

### 注册接口
- URL: `/api/auth/register`
- Method: POST
- Request Body:
  ```json
  {
    "username": "string",
    "email": "string",
    "password": "string"
  }
  ```
- Response:
  ```json
  {
    "code": 200,
    "message": "注册成功",
    "data": {
      "userInfo": {},
      "token": "string"
    }
  }
  ```

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
      "token": "string"
    }
  }
  ```

## 7. 安全考虑

1. 密码加密存储：后端需要使用bcrypt等加密算法对密码进行加密存储
2. Token机制：使用JWT生成访问令牌
3. HTTPS通信：确保数据传输安全
4. 输入验证：前后端都需要进行输入验证
5. 防止重复注册：检查用户名和邮箱是否已存在
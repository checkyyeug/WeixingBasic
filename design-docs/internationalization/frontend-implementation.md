# 跨平台用户管理系统前端国际化实现方案

## 1. 前端国际化实现概述

### 1.1 实现目标

基于已设计的国际化架构和语言切换机制，本文档详细描述前端国际化的具体实现方案，包括：

1. **语言资源管理**：设计和实现语言资源的组织、加载和缓存机制
2. **国际化组件**：实现语言切换组件和文本国际化组件
3. **页面国际化**：实现页面级别的国际化支持
4. **API请求国际化**：处理API请求和响应的国际化
5. **错误处理和回退机制**：确保国际化功能的健壮性

### 1.2 技术选型

- **uni-app+Vue框架**：使用uni-app框架实现跨平台开发，兼容微信小程序、iOS和Android
- **Vue 3 Composition API**：使用Vue 3的Composition API提高开发效率和代码复用性
- **模块化设计**：采用模块化设计，便于维护和扩展
- **Vue响应式状态管理**：使用Vue的响应式系统处理语言变更通知

## 2. 语言资源管理实现

### 2.1 语言资源文件结构

```
src/
├── i18n/                      # 国际化目录
│   ├── locales/               # 语言资源目录
│   │   ├── en.json            # 英文语言资源
│   │   ├── zh-CN.json         # 简体中文语言资源
│   │   └── zh-TW.json         # 繁体中文语言资源
│   ├── index.js               # 国际化入口文件
│   ├── manager.js             # 国际化管理器
│   ├── storage.js             # 国际化存储管理
│   ├── utils.js               # 国际化工具函数
│   └── plugins/               # Vue插件目录
│       └── i18n.js            # Vue国际化插件
├── components/                # 组件目录
│   ├── LanguageSwitcher.vue  # 语言切换组件
│   └── FormattedText.vue      # 格式化文本组件
└── composables/               # 组合式函数目录
    └── useI18n.js            # 国际化组合式函数
```

### 2.2 语言资源文件内容示例

#### 2.2.1 英文语言资源 (en.json)

```json
{
  "common": {
    "ok": "OK",
    "cancel": "Cancel",
    "save": "Save",
    "delete": "Delete",
    "edit": "Edit",
    "loading": "Loading...",
    "success": "Success",
    "error": "Error",
    "confirm": "Confirm",
    "languageChanged": "Language changed successfully",
    "changingLanguage": "Changing language...",
    "languageChangeFailed": "Failed to change language",
    "languageChangeError": "Error changing language",
    "languageSynced": "Language settings synced"
  },
  "user": {
    "username": "Username",
    "email": "Email",
    "password": "Password",
    "nickname": "Nickname",
    "phone": "Phone",
    "bio": "Bio",
    "profile": {
      "title": "User Profile",
      "edit": "Edit Profile",
      "saveSuccess": "Profile saved successfully",
      "saveFailed": "Failed to save profile"
    },
    "login": {
      "title": "Login",
      "submit": "Login",
      "success": "Login successful",
      "failed": "Login failed",
      "invalidCredentials": "Invalid username or password",
      "accountNotFound": "Account not found"
    },
    "register": {
      "title": "Register",
      "submit": "Register",
      "success": "Registration successful",
      "failed": "Registration failed",
      "usernameExists": "Username already exists",
      "emailExists": "Email already exists"
    }
  },
  "role": {
    "user": "User",
    "admin": "Administrator",
    "super_admin": "Super Administrator",
    "management": {
      "title": "Role Management",
      "add": "Add Role",
      "edit": "Edit Role",
      "delete": "Delete Role",
      "assign": "Assign Role"
    }
  },
  "permission": {
    "USER_VIEW": "View Users",
    "USER_EDIT": "Edit Users",
    "USER_DELETE": "Delete Users",
    "CONTENT_VIEW": "View Content",
    "CONTENT_CREATE": "Create Content",
    "CONTENT_EDIT": "Edit Content",
    "CONTENT_DELETE": "Delete Content",
    "SYSTEM_CONFIG": "System Configuration",
    "ROLE_MANAGE": "Role Management",
    "PERMISSION_MANAGE": "Permission Management"
  },
  "error": {
    "network": "Network error, please try again",
    "server": "Server error, please try again later",
    "unknown": "Unknown error occurred",
    "permissionDenied": "Permission denied",
    "sessionExpired": "Session expired, please login again"
  }
}
```

#### 2.2.2 简体中文语言资源 (zh-CN.json)

```json
{
  "common": {
    "ok": "确定",
    "cancel": "取消",
    "save": "保存",
    "delete": "删除",
    "edit": "编辑",
    "loading": "加载中...",
    "success": "成功",
    "error": "错误",
    "confirm": "确认",
    "languageChanged": "语言切换成功",
    "changingLanguage": "正在切换语言...",
    "languageChangeFailed": "语言切换失败",
    "languageChangeError": "切换语言出错",
    "languageSynced": "语言设置已同步"
  },
  "user": {
    "username": "用户名",
    "email": "邮箱",
    "password": "密码",
    "nickname": "昵称",
    "phone": "手机号",
    "bio": "个人简介",
    "profile": {
      "title": "用户资料",
      "edit": "编辑资料",
      "saveSuccess": "资料保存成功",
      "saveFailed": "资料保存失败"
    },
    "login": {
      "title": "登录",
      "submit": "登录",
      "success": "登录成功",
      "failed": "登录失败",
      "invalidCredentials": "用户名或密码错误",
      "accountNotFound": "账号不存在"
    },
    "register": {
      "title": "注册",
      "submit": "注册",
      "success": "注册成功",
      "failed": "注册失败",
      "usernameExists": "用户名已存在",
      "emailExists": "邮箱已存在"
    }
  },
  "role": {
    "user": "普通用户",
    "admin": "管理员",
    "super_admin": "超级管理员",
    "management": {
      "title": "角色管理",
      "add": "添加角色",
      "edit": "编辑角色",
      "delete": "删除角色",
      "assign": "分配角色"
    }
  },
  "permission": {
    "USER_VIEW": "查看用户",
    "USER_EDIT": "编辑用户",
    "USER_DELETE": "删除用户",
    "CONTENT_VIEW": "查看内容",
    "CONTENT_CREATE": "创建内容",
    "CONTENT_EDIT": "编辑内容",
    "CONTENT_DELETE": "删除内容",
    "SYSTEM_CONFIG": "系统配置",
    "ROLE_MANAGE": "角色管理",
    "PERMISSION_MANAGE": "权限管理"
  },
  "error": {
    "network": "网络错误，请重试",
    "server": "服务器错误，请稍后重试",
    "unknown": "发生未知错误",
    "permissionDenied": "权限不足",
    "sessionExpired": "会话已过期，请重新登录"
  }
}
```

#### 2.2.3 繁体中文语言资源 (zh-TW.json)

```json
{
  "common": {
    "ok": "確定",
    "cancel": "取消",
    "save": "保存",
    "delete": "刪除",
    "edit": "編輯",
    "loading": "加載中...",
    "success": "成功",
    "error": "錯誤",
    "confirm": "確認",
    "languageChanged": "語言切換成功",
    "changingLanguage": "正在切換語言...",
    "languageChangeFailed": "語言切換失敗",
    "languageChangeError": "切換語言出錯",
    "languageSynced": "語言設置已同步"
  },
  "user": {
    "username": "用戶名",
    "email": "郵箱",
    "password": "密碼",
    "nickname": "暱稱",
    "phone": "手機號",
    "bio": "個人簡介",
    "profile": {
      "title": "用戶資料",
      "edit": "編輯資料",
      "saveSuccess": "資料保存成功",
      "saveFailed": "資料保存失敗"
    },
    "login": {
      "title": "登錄",
      "submit": "登錄",
      "success": "登錄成功",
      "failed": "登錄失敗",
      "invalidCredentials": "用戶名或密碼錯誤",
      "accountNotFound": "賬號不存在"
    },
    "register": {
      "title": "註冊",
      "submit": "註冊",
      "success": "註冊成功",
      "failed": "註冊失敗",
      "usernameExists": "用戶名已存在",
      "emailExists": "郵箱已存在"
    }
  },
  "role": {
    "user": "普通用戶",
    "admin": "管理員",
    "super_admin": "超級管理員",
    "management": {
      "title": "角色管理",
      "add": "添加角色",
      "edit": "編輯角色",
      "delete": "刪除角色",
      "assign": "分配角色"
    }
  },
  "permission": {
    "USER_VIEW": "查看用戶",
    "USER_EDIT": "編輯用戶",
    "USER_DELETE": "刪除用戶",
    "CONTENT_VIEW": "查看內容",
    "CONTENT_CREATE": "創建內容",
    "CONTENT_EDIT": "編輯內容",
    "CONTENT_DELETE": "刪除內容",
    "SYSTEM_CONFIG": "系統配置",
    "ROLE_MANAGE": "角色管理",
    "PERMISSION_MANAGE": "權限管理"
  },
  "error": {
    "network": "網絡錯誤，請重試",
    "server": "服務器錯誤，請稍後重試",
    "unknown": "發生未知錯誤",
    "permissionDenied": "權限不足",
    "sessionExpired": "會話已過期，請重新登錄"
  }
}
```

### 2.3 国际化管理器实现

```javascript
// src/i18n/manager.js
import { ref, reactive, watchEffect } from 'vue';
import I18nStorage from './storage.js';

// 创建响应式状态
const currentLocale = ref('zh-CN'); // 默认语言
const locales = reactive({}); // 语言资源缓存
const fallbackLocale = ref('zh-CN'); // 回退语言
const isInitialized = ref(false);
const changeCallbacks = []; // 语言变更回调函数
const storage = new I18nStorage();

// 国际化管理器
export const useI18nManager = () => {
  // 初始化国际化管理器
  const init = async () => {
    if (isInitialized.value) {
      return;
    }
    
    try {
      // 从本地存储获取用户语言设置
      const savedLocale = storage.getUserLocale();
      
      if (savedLocale && isValidLocale(savedLocale)) {
        currentLocale.value = savedLocale;
      } else {
        // 从后端获取用户语言设置
        const userLocale = await fetchUserLocaleFromBackend();
        if (userLocale && isValidLocale(userLocale)) {
          currentLocale.value = userLocale;
          // 保存到本地存储
          storage.saveUserLocale(userLocale);
        }
      }
      
      // 检查是否需要同步语言资源
      if (storage.needsSync()) {
        await syncLocales();
      } else {
        // 加载语言资源
        await loadLocales();
      }
      
      isInitialized.value = true;
      
      // 通知初始化完成
      notifyInitComplete();
    } catch (error) {
      console.error('Failed to initialize I18nManager:', error);
      
      // 即使初始化失败，也要设置为已初始化，避免阻塞
      isInitialized.value = true;
      notifyInitComplete();
    }
  };

  // 通知初始化完成
  const notifyInitComplete = () => {
    changeCallbacks.forEach(callback => {
      try {
        callback(currentLocale.value);
      } catch (error) {
        console.error('Error in locale change callback:', error);
      }
    });
  };

  // 添加语言变更回调
  const addChangeCallback = (callback) => {
    if (typeof callback === 'function') {
      changeCallbacks.push(callback);
    }
  };

  // 移除语言变更回调
  const removeChangeCallback = (callback) => {
    const index = changeCallbacks.indexOf(callback);
    if (index !== -1) {
      changeCallbacks.splice(index, 1);
    }
  };

  // 验证语言代码是否有效
  const isValidLocale = (locale) => {
    return ['en', 'zh-CN', 'zh-TW'].includes(locale);
  };

  // 从后端获取用户语言设置
  const fetchUserLocaleFromBackend = async () => {
    try {
      const token = uni.getStorageSync('token');
      if (!token) {
        return null;
      }
      
      const [error, response] = await uni.request({
        url: `${API_BASE_URL}/user/locale`,
        method: 'GET',
        header: {
          'Authorization': `Bearer ${token}`
        }
      });
      
      if (!error && response.statusCode === 200 && response.data.code === 200) {
        return response.data.data.locale;
      }
    } catch (error) {
      console.error('Failed to fetch user locale from backend:', error);
    }
    
    return null;
  };

  // 同步语言资源
  const syncLocales = async () => {
    try {
      const localeList = ['en', 'zh-CN', 'zh-TW'];
      
      // 获取语言资源版本
      const [versionError, versionResponse] = await uni.request({
        url: `${API_BASE_URL}/i18n/version`,
        method: 'GET'
      });
      
      if (!versionError && versionResponse.statusCode === 200 && versionResponse.data.code === 200) {
        const serverVersion = versionResponse.data.data.version;
        const localVersion = storage.getLocaleVersion();
        
        // 如果版本不同，清除缓存
        if (serverVersion !== localVersion) {
          storage.clearLocaleCache();
          storage.saveLocaleVersion(serverVersion);
        }
      }
      
      // 加载语言资源
      for (const locale of localeList) {
        await updateLocaleFromBackend(locale);
      }
      
      // 更新最后同步时间
      storage.saveLastSyncTime();
    } catch (error) {
      console.error('Failed to sync locales:', error);
      
      // 如果同步失败，尝试从缓存加载
      await loadLocales();
    }
  };

  // 加载语言资源
  const loadLocales = async () => {
    const localeList = ['en', 'zh-CN', 'zh-TW'];
    
    for (const locale of localeList) {
      try {
        // 优先从缓存加载
        const cachedLocale = storage.getLocaleCache(locale);
        if (cachedLocale) {
          locales[locale] = cachedLocale;
        } else {
          // 如果缓存中没有，从本地文件加载
          locales[locale] = await loadLocalLocale(locale);
        }
      } catch (error) {
        console.error(`Failed to load locale ${locale}:`, error);
        locales[locale] = {};
      }
    }
  };

  // 从本地加载语言资源
  const loadLocalLocale = async (locale) => {
    try {
      // 在uni-app中，可以使用require直接导入JSON文件
      const localeData = require(`./locales/${locale}.json`);
      return localeData;
    } catch (error) {
      console.error(`Failed to load local locale ${locale}:`, error);
      return {};
    }
  };

  // 从后端更新语言资源
  const updateLocaleFromBackend = async (locale) => {
    try {
      const [error, response] = await uni.request({
        url: `${API_BASE_URL}/i18n/locales/${locale}`,
        method: 'GET'
      });
      
      if (!error && response.statusCode === 200 && response.data.code === 200) {
        const localeData = response.data.data;
        
        // 更新内存中的语言资源
        locales[locale] = localeData;
        
        // 更新缓存
        storage.saveLocaleCache(locale, localeData);
      }
    } catch (error) {
      console.error(`Failed to update locale ${locale} from backend:`, error);
    }
  };

  // 切换语言
  const setLocale = async (locale) => {
    if (!isValidLocale(locale)) {
      console.error(`Invalid locale: ${locale}`);
      return false;
    }
    
    if (locale === currentLocale.value) {
      return true;
    }
    
    currentLocale.value = locale;
    
    // 保存到本地存储
    storage.saveUserLocale(locale);
    
    // 保存到后端
    try {
      await saveUserLocaleToBackend(locale);
    } catch (error) {
      console.error('Failed to save user locale to backend:', error);
      // 即使保存到后端失败，也继续执行
    }
    
    // 通知所有页面语言已更改
    notifyLocaleChange();
    
    return true;
  };

  // 保存用户语言设置到后端
  const saveUserLocaleToBackend = async (locale) => {
    const token = uni.getStorageSync('token');
    if (!token) {
      return;
    }
    
    const [error, response] = await uni.request({
      url: `${API_BASE_URL}/user/locale`,
      method: 'PUT',
      data: { locale },
      header: {
        'Authorization': `Bearer ${token}`
      }
    });
    
    if (error || response.statusCode !== 200 || response.data.code !== 200) {
      throw new Error('Failed to save user locale to backend');
    }
  };

  // 获取当前语言
  const getCurrentLocale = () => {
    return currentLocale.value;
  };

  // 获取翻译文本
  const t = (key, params = {}) => {
    if (!isInitialized.value) {
      console.warn('I18nManager not initialized');
      return key;
    }
    
    const locale = locales[currentLocale.value];
    if (!locale) {
      console.warn(`Locale ${currentLocale.value} not loaded`);
      return key;
    }
    
    // 支持嵌套键，如 'user.profile.title'
    const keys = key.split('.');
    let value = locale;
    
    for (const k of keys) {
      if (value && typeof value === 'object' && k in value) {
        value = value[k];
      } else {
        // 如果当前语言没有找到翻译，尝试回退语言
        if (currentLocale.value !== fallbackLocale.value) {
          const fallbackValue = getFallbackTranslation(key);
          if (fallbackValue !== key) {
            return interpolate(fallbackValue, params);
          }
        }
        return key;
      }
    }
    
    if (typeof value !== 'string') {
      return key;
    }
    
    return interpolate(value, params);
  };

  // 获取回退翻译
  const getFallbackTranslation = (key) => {
    const fallback = locales[fallbackLocale.value];
    if (!fallback) {
      return key;
    }
    
    const keys = key.split('.');
    let value = fallback;
    
    for (const k of keys) {
      if (value && typeof value === 'object' && k in value) {
        value = value[k];
      } else {
        return key;
      }
    }
    
    return typeof value === 'string' ? value : key;
  };

  // 参数插值
  const interpolate = (text, params) => {
    return text.replace(/\{(\w+)\}/g, (match, key) => {
      return key in params ? params[key] : match;
    });
  };

  // 通知语言变更
  const notifyLocaleChange = () => {
    // 通知所有回调函数
    changeCallbacks.forEach(callback => {
      try {
        callback(currentLocale.value);
      } catch (error) {
        console.error('Error in locale change callback:', error);
      }
    });
    
    // 通过全局事件通知所有页面
    const app = getApp();
    if (app && app.globalData) {
      app.globalData.currentLocale = currentLocale.value;
      app.globalData.localeChanged = Date.now();
    }
    
    // 触发自定义事件
    uni.$emit('localeChange', currentLocale.value);
  };

  // 格式化日期
  const formatDate = (date, format) => {
    // 根据当前语言格式化日期
    if (currentLocale.value === 'en') {
      // 英文日期格式
      // 实现逻辑...
    } else if (currentLocale.value === 'zh-TW') {
      // 繁体中文日期格式
      // 实现逻辑...
    } else {
      // 简体中文日期格式
      // 实现逻辑...
    }
  };

  // 格式化数字
  const formatNumber = (number, options = {}) => {
    // 根据当前语言格式化数字
    if (currentLocale.value === 'en') {
      // 英文数字格式
      return new Intl.NumberFormat('en-US', options).format(number);
    } else if (currentLocale.value === 'zh-TW') {
      // 繁体中文数字格式
      return new Intl.NumberFormat('zh-TW', options).format(number);
    } else {
      // 简体中文数字格式
      return new Intl.NumberFormat('zh-CN', options).format(number);
    }
  };

  // 格式化货币
  const formatCurrency = (amount, currency = 'CNY') => {
    // 根据当前语言格式化货币
    if (currentLocale.value === 'en') {
      return new Intl.NumberFormat('en-US', { style: 'currency', currency }).format(amount);
    } else if (currentLocale.value === 'zh-TW') {
      return new Intl.NumberFormat('zh-TW', { style: 'currency', currency }).format(amount);
    } else {
      return new Intl.NumberFormat('zh-CN', { style: 'currency', currency }).format(amount);
    }
  };

  // 返回国际化管理器的方法和状态
  return {
    init,
    setLocale,
    getCurrentLocale,
    t,
    formatDate,
    formatNumber,
    formatCurrency,
    addChangeCallback,
    removeChangeCallback,
    isInitialized: computed(() => isInitialized.value),
    currentLocale: computed(() => currentLocale.value)
  };
};

// 创建默认的国际化管理器实例
const i18nManager = useI18nManager();

export default i18nManager;

### 2.4 国际化存储管理实现

```javascript
// src/i18n/storage.js

class I18nStorage {
  constructor() {
    this.STORAGE_KEYS = {
      USER_LOCALE: 'user_locale',
      LOCALE_CACHE: 'locale_cache',
      LOCALE_VERSION: 'locale_version',
      LAST_SYNC_TIME: 'last_sync_time'
    };
  }

  // 获取用户语言设置
  getUserLocale() {
    try {
      return uni.getStorageSync(this.STORAGE_KEYS.USER_LOCALE);
    } catch (error) {
      console.error('Failed to get user locale from storage:', error);
      return null;
    }
  }

  // 保存用户语言设置
  saveUserLocale(locale) {
    try {
      uni.setStorageSync(this.STORAGE_KEYS.USER_LOCALE, locale);
      return true;
    } catch (error) {
      console.error('Failed to save user locale to storage:', error);
      return false;
    }
  }

  // 获取语言资源缓存
  getLocaleCache(locale) {
    try {
      const cacheKey = `${this.STORAGE_KEYS.LOCALE_CACHE}_${locale}`;
      const cachedData = uni.getStorageSync(cacheKey);
      
      if (cachedData) {
        return JSON.parse(cachedData);
      }
      
      return null;
    } catch (error) {
      console.error(`Failed to get locale cache for ${locale}:`, error);
      return null;
    }
  }

  // 保存语言资源缓存
  saveLocaleCache(locale, data) {
    try {
      const cacheKey = `${this.STORAGE_KEYS.LOCALE_CACHE}_${locale}`;
      uni.setStorageSync(cacheKey, JSON.stringify(data));
      return true;
    } catch (error) {
      console.error(`Failed to save locale cache for ${locale}:`, error);
      return false;
    }
  }

  // 清除语言资源缓存
  clearLocaleCache() {
    try {
      const keys = uni.getStorageInfoSync().keys;
      
      for (const key of keys) {
        if (key.startsWith(this.STORAGE_KEYS.LOCALE_CACHE)) {
          uni.removeStorageSync(key);
        }
      }
      
      return true;
    } catch (error) {
      console.error('Failed to clear locale cache:', error);
      return false;
    }
  }

  // 获取语言资源版本
  getLocaleVersion() {
    try {
      return uni.getStorageSync(this.STORAGE_KEYS.LOCALE_VERSION);
    } catch (error) {
      console.error('Failed to get locale version from storage:', error);
      return null;
    }
  }

  // 保存语言资源版本
  saveLocaleVersion(version) {
    try {
      uni.setStorageSync(this.STORAGE_KEYS.LOCALE_VERSION, version);
      return true;
    } catch (error) {
      console.error('Failed to save locale version to storage:', error);
      return false;
    }
  }

  // 获取最后同步时间
  getLastSyncTime() {
    try {
      return uni.getStorageSync(this.STORAGE_KEYS.LAST_SYNC_TIME);
    } catch (error) {
      console.error('Failed to get last sync time from storage:', error);
      return null;
    }
  }

  // 保存最后同步时间
  saveLastSyncTime() {
    try {
      uni.setStorageSync(this.STORAGE_KEYS.LAST_SYNC_TIME, Date.now());
      return true;
    } catch (error) {
      console.error('Failed to save last sync time to storage:', error);
      return false;
    }
  }

  // 检查是否需要同步
  needsSync() {
    try {
      const lastSyncTime = this.getLastSyncTime();
      
      // 如果没有同步过，或者距离上次同步超过24小时，则需要同步
      if (!lastSyncTime) {
        return true;
      }
      
      const oneDay = 24 * 60 * 60 * 1000;
      return Date.now() - lastSyncTime > oneDay;
    } catch (error) {
      console.error('Failed to check if sync is needed:', error);
      return false;
    }
  }
}

export default I18nStorage;
```

### 2.5 Vue国际化插件实现

```javascript
// src/i18n/plugins/i18n.js
import { ref, computed, provide, inject } from 'vue';
import i18nManager from '../manager.js';

// 创建插件
const I18nPlugin = {
  install(app, options = {}) {
    // 提供全局的国际化管理器
    app.config.globalProperties.$i18n = i18nManager;
    
    // 提供全局的翻译方法
    app.config.globalProperties.$t = i18nManager.t;
    
    // 提供全局的语言切换方法
    app.config.globalProperties.$setLocale = i18nManager.setLocale;
    
    // 提供全局的当前语言
    app.config.globalProperties.$currentLocale = computed(() => i18nManager.getCurrentLocale());
    
    // 提供全局的格式化方法
    app.config.globalProperties.$formatDate = i18nManager.formatDate;
    app.config.globalProperties.$formatNumber = i18nManager.formatNumber;
    app.config.globalProperties.$formatCurrency = i18nManager.formatCurrency;
    
    // 提供依赖注入
    app.provide('i18n', i18nManager);
    app.provide('t', i18nManager.t);
    app.provide('setLocale', i18nManager.setLocale);
    app.provide('currentLocale', computed(() => i18nManager.getCurrentLocale()));
    app.provide('formatDate', i18nManager.formatDate);
    app.provide('formatNumber', i18nManager.formatNumber);
    app.provide('formatCurrency', i18nManager.formatCurrency);
    
    // 初始化国际化管理器
    i18nManager.init().catch(error => {
      console.error('Failed to initialize i18n:', error);
    });
  }
};

// 创建组合式函数
export function useI18n() {
  const i18n = inject('i18n');
  const t = inject('t');
  const setLocale = inject('setLocale');
  const currentLocale = inject('currentLocale');
  const formatDate = inject('formatDate');
  const formatNumber = inject('formatNumber');
  const formatCurrency = inject('formatCurrency');
  
  if (!i18n || !t || !setLocale || !currentLocale) {
    throw new Error('I18n plugin not installed. Make sure to install it with app.use(I18nPlugin).');
  }
  
  return {
    i18n,
    t,
    setLocale,
    currentLocale,
    formatDate,
    formatNumber,
    formatCurrency
  };
}

export default I18nPlugin;
```

### 2.6 国际化组合式函数实现

```javascript
// src/composables/useI18n.js
import { ref, computed, onMounted, onUnmounted } from 'vue';
import { useI18n } from '../i18n/plugins/i18n.js';

export function useLocale() {
  const { i18n, t, setLocale, currentLocale, formatDate, formatNumber, formatCurrency } = useI18n();
  const isChanging = ref(false);
  const errorMessage = ref('');
  
  // 切换语言
  const changeLocale = async (locale) => {
    if (isChanging.value) {
      return;
    }
    
    isChanging.value = true;
    errorMessage.value = '';
    
    try {
      const success = await setLocale(locale);
      
      if (!success) {
        errorMessage.value = t('error.languageChangeFailed');
      }
      
      return success;
    } catch (error) {
      console.error('Failed to change locale:', error);
      errorMessage.value = t('error.languageChangeError');
      return false;
    } finally {
      isChanging.value = false;
    }
  };
  
  // 格式化带参数的文本
  const formatText = (key, params = {}) => {
    return t(key, params);
  };
  
  // 格式化相对时间
  const formatRelativeTime = (date) => {
    const now = new Date();
    const diff = now - date;
    const seconds = Math.floor(diff / 1000);
    const minutes = Math.floor(seconds / 60);
    const hours = Math.floor(minutes / 60);
    const days = Math.floor(hours / 24);
    
    if (days > 0) {
      return formatText('time.daysAgo', { days });
    } else if (hours > 0) {
      return formatText('time.hoursAgo', { hours });
    } else if (minutes > 0) {
      return formatText('time.minutesAgo', { minutes });
    } else {
      return formatText('time.justNow');
    }
  };
  
  // 监听语言变化
  const onLocaleChange = (callback) => {
    i18n.addChangeCallback(callback);
    
    // 组件卸载时移除回调
    onUnmounted(() => {
      i18n.removeChangeCallback(callback);
    });
  };
  
  return {
    currentLocale,
    isChanging,
    errorMessage,
    changeLocale,
    formatText,
    formatDate,
    formatNumber,
    formatCurrency,
    formatRelativeTime,
    onLocaleChange
  };
}

export default useLocale;
```

### 2.7 语言切换组件实现

```vue
<!-- src/components/LanguageSwitcher.vue -->
<template>
  <view class="language-switcher">
    <view 
      v-for="locale in supportedLocales" 
      :key="locale.code"
      :class="['language-option', { active: currentLocale === locale.code }]"
      @click="handleChangeLocale(locale.code)"
    >
      <text class="language-name">{{ locale.name }}</text>
      <text v-if="currentLocale === locale.code" class="check-icon">✓</text>
    </view>
    
    <view v-if="isChanging" class="loading-indicator">
      <text>{{ $t('common.changingLanguage') }}</text>
    </view>
    
    <view v-if="errorMessage" class="error-message">
      <text>{{ errorMessage }}</text>
    </view>
  </view>
</template>

<script>
import { defineComponent } from 'vue';
import { useLocale } from '../composables/useI18n.js';

export default defineComponent({
  name: 'LanguageSwitcher',
  setup() {
    const { 
      currentLocale, 
      isChanging, 
      errorMessage, 
      changeLocale 
    } = useLocale();
    
    const supportedLocales = [
      { code: 'en', name: 'English' },
      { code: 'zh-CN', name: '简体中文' },
      { code: 'zh-TW', name: '繁體中文' }
    ];
    
    const handleChangeLocale = async (locale) => {
      if (locale === currentLocale.value) {
        return;
      }
      
      const success = await changeLocale(locale);
      
      if (success) {
        uni.showToast({
          title: 'Language changed',
          icon: 'success'
        });
      }
    };
    
    return {
      currentLocale,
      isChanging,
      errorMessage,
      supportedLocales,
      handleChangeLocale
    };
  }
});
</script>

<style>
.language-switcher {
  display: flex;
  flex-direction: column;
  gap: 10px;
  padding: 15px;
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.language-option {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 16px;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.language-option:hover {
  background-color: #f5f5f5;
}

.language-option.active {
  background-color: #e6f7ff;
  color: #1890ff;
}

.language-name {
  font-size: 16px;
}

.check-icon {
  font-size: 18px;
  font-weight: bold;
  color: #1890ff;
}

.loading-indicator {
  text-align: center;
  padding: 10px;
  color: #666;
}

.error-message {
  text-align: center;
  padding: 10px;
  color: #ff4d4f;
}
</style>
```

### 2.8 格式化文本组件实现

```vue
<!-- src/components/FormattedText.vue -->
<template>
  <text 
    :class="className"
    @click="handleClick"
  >
    {{ formattedText }}
  </text>
</template>

<script>
import { defineComponent, computed, watch } from 'vue';
import { useLocale } from '../composables/useI18n.js';

export default defineComponent({
  name: 'FormattedText',
  props: {
    // 翻译键
    key: {
      type: String,
      required: true
    },
    // 翻译参数
    params: {
      type: Object,
      default: () => ({})
    },
    // 默认文本（当翻译不存在时使用）
    default: {
      type: String,
      default: ''
    },
    // 自定义类名
    className: {
      type: String,
      default: ''
    },
    // 是否格式化为HTML
    isHtml: {
      type: Boolean,
      default: false
    }
  },
  emits: ['click'],
  setup(props, { emit }) {
    const { formatText, currentLocale } = useLocale();
    
    // 格式化文本
    const formattedText = computed(() => {
      try {
        const text = formatText(props.key, props.params);
        
        // 如果翻译结果与键相同，说明没有找到翻译，使用默认文本
        if (text === props.key && props.default) {
          return props.default;
        }
        
        return text;
      } catch (error) {
        console.error(`Failed to format text for key "${props.key}":`, error);
        return props.default || props.key;
      }
    });
    
    // 点击处理
    const handleClick = (event) => {
      emit('click', event);
    };
    
    return {
      formattedText,
      handleClick
    };
  }
});
</script>

<style>
/* 可以根据需要添加样式 */
</style>
```

## 3. 页面国际化实现

### 3.1 页面级别国际化

```vue
<!-- src/pages/index/index.vue -->
<template>
  <view class="container">
    <view class="header">
      <text class="title">{{ $t('home.title') }}</text>
      <LanguageSwitcher />
    </view>
    
    <view class="content">
      <text class="welcome">{{ $t('home.welcome', { name: userName }) }}</text>
      
      <view class="stats">
        <view class="stat-item">
          <text class="stat-number">{{ formatNumber(userCount) }}</text>
          <text class="stat-label">{{ $t('home.userCount') }}</text>
        </view>
        <view class="stat-item">
          <text class="stat-number">{{ formatCurrency(revenue) }}</text>
          <text class="stat-label">{{ $t('home.revenue') }}</text>
        </view>
      </view>
      
      <view class="actions">
        <button @click="handleRefresh">{{ $t('common.refresh') }}</button>
        <button @click="handleSettings">{{ $t('common.settings') }}</button>
      </view>
    </view>
    
    <view class="footer">
      <text class="last-updated">
        {{ $t('home.lastUpdated', { time: formatRelativeTime(lastUpdated) }) }}
      </text>
    </view>
  </view>
</template>

<script>
import { defineComponent, ref, onMounted } from 'vue';
import { useLocale } from '../../composables/useI18n.js';
import LanguageSwitcher from '../../components/LanguageSwitcher.vue';

export default defineComponent({
  name: 'HomePage',
  components: {
    LanguageSwitcher
  },
  setup() {
    const { 
      currentLocale, 
      formatText, 
      formatDate, 
      formatNumber, 
      formatCurrency, 
      formatRelativeTime,
      onLocaleChange 
    } = useLocale();
    
    const userName = ref('John Doe');
    const userCount = ref(12345);
    const revenue = ref(98765.43);
    const lastUpdated = ref(new Date());
    
    // 刷新数据
    const handleRefresh = () => {
      // 模拟数据刷新
      userCount.value = Math.floor(Math.random() * 100000);
      revenue.value = Math.random() * 100000;
      lastUpdated.value = new Date();
      
      uni.showToast({
        title: formatText('common.refreshed'),
        icon: 'success'
      });
    };
    
    // 打开设置
    const handleSettings = () => {
      uni.navigateTo({
        url: '/pages/settings/index'
      });
    };
    
    // 监听语言变化，重新格式化数据
    onLocaleChange(() => {
      // 语言变化时，可以在这里执行一些操作
      console.log('Language changed to:', currentLocale.value);
    });
    
    onMounted(() => {
      // 初始化页面数据
      handleRefresh();
    });
    
    return {
      userName,
      userCount,
      revenue,
      lastUpdated,
      handleRefresh,
      handleSettings,
      formatNumber,
      formatCurrency,
      formatRelativeTime
    };
  }
});
</script>

<style>
.container {
  padding: 20px;
  background-color: #f5f5f5;
  min-height: 100vh;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.title {
  font-size: 24px;
  font-weight: bold;
  color: #333;
}

.content {
  background-color: #fff;
  border-radius: 8px;
  padding: 20px;
  margin-bottom: 20px;
}

.welcome {
  font-size: 18px;
  margin-bottom: 20px;
  color: #666;
}

.stats {
  display: flex;
  justify-content: space-around;
  margin-bottom: 20px;
}

.stat-item {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.stat-number {
  font-size: 24px;
  font-weight: bold;
  color: #1890ff;
}

.stat-label {
  font-size: 14px;
  color: #666;
  margin-top: 5px;
}

.actions {
  display: flex;
  justify-content: space-around;
}

.actions button {
  padding: 10px 20px;
  border-radius: 4px;
  background-color: #1890ff;
  color: #fff;
}

.footer {
  text-align: center;
  color: #999;
  font-size: 12px;
}

.last-updated {
  color: #999;
  font-size: 12px;
}
</style>
```

### 3.2 API请求国际化

```javascript
// src/utils/request.js
import { useLocale } from '../composables/useI18n.js';

// 创建请求拦截器
const requestInterceptor = (config) => {
  const { currentLocale } = useLocale();
  
  // 添加语言头
  config.header = config.header || {};
  config.header['Accept-Language'] = currentLocale.value;
  
  // 添加认证令牌
  const token = uni.getStorageSync('token');
  if (token) {
    config.header['Authorization'] = `Bearer ${token}`;
  }
  
  return config;
};

// 创建响应拦截器
const responseInterceptor = (response) => {
  const { t } = useLocale();
  
  // 处理国际化错误消息
  if (response.statusCode >= 400) {
    let errorMessage = '';
    
    if (response.data && response.data.message) {
      // 如果后端返回了国际化错误消息，直接使用
      errorMessage = response.data.message;
    } else {
      // 根据状态码使用前端定义的错误消息
      switch (response.statusCode) {
        case 400:
          errorMessage = t('error.badRequest');
          break;
        case 401:
          errorMessage = t('error.unauthorized');
          break;
        case 403:
          errorMessage = t('error.forbidden');
          break;
        case 404:
          errorMessage = t('error.notFound');
          break;
        case 500:
          errorMessage = t('error.server');
          break;
        default:
          errorMessage = t('error.unknown');
      }
    }
    
    // 显示错误消息
    uni.showToast({
      title: errorMessage,
      icon: 'none',
      duration: 3000
    });
    
    // 抛出错误，阻止后续处理
    return Promise.reject(new Error(errorMessage));
  }
  
  return response;
};

// 封装uni.request
export function request(options) {
  return new Promise((resolve, reject) => {
    // 应用请求拦截器
    const config = requestInterceptor(options);
    
    uni.request({
      ...config,
      success: (response) => {
        try {
          // 应用响应拦截器
          const interceptedResponse = responseInterceptor(response);
          resolve(interceptedResponse);
        } catch (error) {
          reject(error);
        }
      },
      fail: (error) => {
        const { t } = useLocale();
        const errorMessage = t('error.network');
        
        uni.showToast({
          title: errorMessage,
          icon: 'none',
          duration: 3000
        });
        
        reject(new Error(errorMessage));
      }
    });
  });
}

// 导出常用的请求方法
export function get(url, data = {}, options = {}) {
  return request({
    url,
    method: 'GET',
    data,
    ...options
  });
}

export function post(url, data = {}, options = {}) {
  return request({
    url,
    method: 'POST',
    data,
    ...options
  });
}

export function put(url, data = {}, options = {}) {
  return request({
    url,
    method: 'PUT',
    data,
    ...options
  });
}

export function del(url, data = {}, options = {}) {
  return request({
    url,
    method: 'DELETE',
    data,
    ...options
  });
}

export default {
  request,
  get,
  post,
  put,
  delete: del
};
```

## 4. 错误处理和回退机制

### 4.1 错误处理策略

1. **初始化失败处理**
   - 如果国际化管理器初始化失败，系统仍然可以运行，但使用默认语言
   - 在控制台记录错误信息，便于调试
   - 提供重试机制，允许用户手动重新初始化

2. **语言资源加载失败处理**
   - 优先使用缓存的语言资源
   - 如果缓存中没有，使用本地默认语言资源
   - 如果本地资源也不可用，使用硬编码的回退文本

3. **语言切换失败处理**
   - 如果切换语言失败，保持当前语言不变
   - 向用户显示错误消息
   - 记录错误日志，便于后续分析

4. **翻译缺失处理**
   - 如果某个键在当前语言中没有翻译，尝试使用回退语言
   - 如果回退语言中也没有，使用键名或默认文本
   - 记录缺失的翻译，便于后续补充

### 4.2 回退机制实现

```javascript
// src/i18n/fallback.js
import { ref } from 'vue';
import i18nManager from './manager.js';

// 回退策略配置
const fallbackStrategy = {
  // 是否启用回退机制
  enabled: true,
  // 回退语言顺序
  locales: ['zh-CN', 'en'],
  // 是否记录缺失翻译
  logMissing: true,
  // 是否使用键名作为回退文本
  useKeyAsFallback: true
};

// 缺失翻译记录
const missingTranslations = ref([]);

// 添加缺失翻译记录
const addMissingTranslation = (key, locale) => {
  if (!fallbackStrategy.logMissing) {
    return;
  }
  
  // 检查是否已经记录过
  const exists = missingTranslations.value.some(
    item => item.key === key && item.locale === locale
  );
  
  if (!exists) {
    missingTranslations.value.push({
      key,
      locale,
      timestamp: new Date().toISOString()
    });
    
    console.warn(`Missing translation for key "${key}" in locale "${locale}"`);
  }
};

// 获取回退翻译
const getFallbackTranslation = (key, currentLocale) => {
  if (!fallbackStrategy.enabled) {
    return null;
  }
  
  // 尝试按顺序使用回退语言
  for (const locale of fallbackStrategy.locales) {
    // 跳过当前语言
    if (locale === currentLocale) {
      continue;
    }
    
    try {
      const translation = i18nManager.getRawTranslation(key, locale);
      if (translation && translation !== key) {
        return translation;
      }
    } catch (error) {
      console.error(`Failed to get fallback translation for key "${key}" in locale "${locale}":`, error);
    }
  }
  
  // 记录缺失翻译
  addMissingTranslation(key, currentLocale);
  
  // 如果配置了使用键名作为回退文本
  if (fallbackStrategy.useKeyAsFallback) {
    return key;
  }
  
  return null;
};

// 导出回退机制
export const useFallback = () => {
  return {
    fallbackStrategy,
    missingTranslations,
    getFallbackTranslation,
    addMissingTranslation
  };
};

export default useFallback;
```

## 5. 总结

本文档详细描述了跨平台用户管理系统前端国际化的具体实现方案，包括：

1. **语言资源管理**：设计了语言资源的组织结构、加载机制和缓存策略
2. **国际化管理器**：实现了基于Vue 3 Composition API的国际化管理器
3. **存储管理**：实现了基于uni-app存储API的国际化存储管理
4. **Vue插件**：提供了Vue插件和组合式函数，便于在组件中使用国际化功能
5. **组件实现**：实现了语言切换组件和格式化文本组件
6. **页面国际化**：展示了如何在页面中使用国际化功能
7. **API请求国际化**：实现了API请求和响应的国际化处理
8. **错误处理和回退机制**：设计了健壮的错误处理和回退机制

这个实现方案支持英文、简体中文和繁体中文三种语言，具有良好的性能、用户体验和可维护性，能够满足跨平台应用对国际化功能的需求。

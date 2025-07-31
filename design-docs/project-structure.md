# 项目结构说明

## 1. 概述

本文档详细描述了微信小程序用户管理系统的完整项目结构，包括前端小程序代码结构和后端服务代码结构，确保开发团队能够按照统一的结构进行开发。

## 2. 前端项目结构 (微信小程序)

```
frontend/
├── app.js                 # 小程序入口文件
├── app.json               # 全局配置
├── app.wxss               # 全局样式
├── project.config.json    # 项目配置文件
├── sitemap.json           # 小程序搜索引擎收录配置
├── components/            # 组件目录
│   ├── navigation-bar/    # 导航栏组件
│   │   ├── navigation-bar.js
│   │   ├── navigation-bar.json
│   │   ├── navigation-bar.wxml
│   │   └── navigation-bar.wxss
│   └── user/              # 用户相关组件
│       ├── avatar/        # 头像组件
│       │   ├── avatar.js
│       │   ├── avatar.json
│       │   ├── avatar.wxml
│       │   └── avatar.wxss
│       └── role-tag/      # 角色标签组件
│           ├── role-tag.js
│           ├── role-tag.json
│           ├── role-tag.wxml
│           └── role-tag.wxss
├── pages/                 # 页面目录
│   ├── index/             # 首页
│   │   ├── index.js
│   │   ├── index.json
│   │   ├── index.wxml
│   │   └── index.wxss
│   ├── auth/              # 认证相关页面
│   │   ├── register/      # 注册页面
│   │   │   ├── register.js
│   │   │   ├── register.json
│   │   │   ├── register.wxml
│   │   │   └── register.wxss
│   │   ├── login/         # 登录页面
│   │   │   ├── login.js
│   │   │   ├── login.json
│   │   │   ├── login.wxml
│   │   │   └── login.wxss
│   │   └── forgot/        # 忘记密码页面
│   │       ├── forgot.js
│   │       ├── forgot.json
│   │       ├── forgot.wxml
│   │       └── forgot.wxss
│   ├── user/              # 用户中心页面
│   │   ├── profile/       # 个人信息页面
│   │   │   ├── profile.js
│   │   │   ├── profile.json
│   │   │   ├── profile.wxml
│   │   │   └── profile.wxss
│   │   ├── profile/       # 编辑个人信息页面
│   │   │   ├── edit/
│   │   │   │   ├── edit.js
│   │   │   │   ├── edit.json
│   │   │   │   ├── edit.wxml
│   │   │   │   └── edit.wxss
│   │   └── roles/         # 角色管理页面
│   │       ├── roles.js
│   │       ├── roles.json
│   │       ├── roles.wxml
│   │       └── roles.wxss
│   │       ├── edit/      # 编辑角色页面
│   │       │   ├── edit.js
│   │       │   ├── edit.json
│   │       │   ├── edit.wxml
│   │       │   └── edit.wxss
│   │       └── user-roles/ # 用户角色分配页面
│   │           ├── user-roles.js
│   │           ├── user-roles.json
│   │           ├── user-roles.wxml
│   │           └── user-roles.wxss
├── utils/                 # 工具函数目录
│   ├── auth.js            # 认证相关工具函数
│   ├── request.js         # 网络请求封装
│   ├── permission.js      # 权限控制工具函数
│   ├── util.js            # 通用工具函数
│   └── validator.js       # 数据验证工具函数
├── api/                   # API接口目录
│   ├── user.js            # 用户相关API
│   ├── auth.js            # 认证相关API
│   ├── role.js            # 角色相关API
│   └── permission.js      # 权限相关API
└── store/                 # 状态管理
    └── user.js            # 用户状态管理
```

## 3. 后端项目结构

```
backend/
├── app/main.py            # FastAPI应用入口文件
├── app/__init__.py        # Python包初始化文件
├── app/api/               # API路由目录
│   ├── __init__.py        # Python包初始化文件
│   ├── v1/                # API v1版本
│   │   ├── __init__.py    # Python包初始化文件
│   │   ├── auth.py        # 认证相关API
│   │   ├── users.py       # 用户相关API
│   │   ├── roles.py       # 角色相关API
│   │   └── permissions.py # 权限相关API
├── app/core/              # 核心配置和安全相关
│   ├── __init__.py        # Python包初始化文件
│   ├── config.py          # 配置管理
│   ├── security.py        # 安全相关功能（JWT、密码哈希等）
│   └── database.py        # 数据库连接和会话管理
├── app/models/            # 数据模型目录
│   ├── __init__.py        # Python包初始化文件
│   ├── user.py            # 用户模型
│   ├── role.py            # 角色模型
│   ├── permission.py      # 权限模型
│   ├── user_role.py       # 用户角色关联模型
│   └── role_permission.py # 角色权限关联模型
├── app/schemas/           # Pydantic数据模型（请求/响应）
│   ├── __init__.py        # Python包初始化文件
│   ├── user.py            # 用户数据模型
│   ├── role.py            # 角色数据模型
│   ├── permission.py      # 权限数据模型
│   └── token.py           # Token数据模型
├── app/crud/              # 数据库操作（增删改查）
│   ├── __init__.py        # Python包初始化文件
│   ├── user.py            # 用户相关数据库操作
│   ├── role.py            # 角色相关数据库操作
│   ├── permission.py      # 权限相关数据库操作
│   └── base.py            # 基础数据库操作类
├── app/utils/             # 工具函数目录
│   ├── __init__.py        # Python包初始化文件
│   ├── password.py        # 密码处理工具
│   └── logger.py          # 日志工具
├── tests/                 # 测试目录
│   ├── __init__.py        # Python包初始化文件
│   ├── test_auth.py       # 认证相关测试
│   ├── test_users.py      # 用户相关测试
│   ├── test_roles.py      # 角色相关测试
│   └── test_permissions.py # 权限相关测试
├── requirements.txt       # Python依赖包列表
├── alembic/               # 数据库迁移工具目录
│   ├── env.py             # Alembic配置
│   ├── script.py.mako     # 迁移脚本模板
│   └── versions/          # 数据库迁移脚本
├── alembic.ini            # Alembic配置文件
├── .env                   # 环境变量配置文件
├── .env.development       # 开发环境配置
├── .env.production        # 生产环境配置
├── .gitignore             # Git忽略文件配置
├── README.md              # 项目说明文档
└── Dockerfile             # Docker镜像构建文件
```

## 4. 文件命名规范

### 4.1 Python文件
- 使用小写字母和下划线分隔：`user_profile.py`
- 测试文件以`test_`开头：`test_user_profile.py`

### 4.2 配置文件
- 环境配置文件：`.env`, `.env.development`, `.env.production`
- 其他配置文件使用小写字母和连字符分隔：`alembic.ini`

### 4.3 Markdown文档
- 使用小写字母和连字符分隔：`design-document.md`

### 4.4 前端文件
- JavaScript文件：使用小写字母和连字符分隔：`user-profile.js`
- JSON配置文件：使用小写字母和连字符分隔：`app-config.json`
- 样式文件：使用小写字母和连字符分隔：`user-profile.wxss`

## 5. 代码组织原则

### 5.1 模块化
- 每个功能模块应有独立的目录
- 模块内部文件按功能分类组织

### 5.2 分层架构
- API层：处理HTTP请求和响应
- 业务逻辑层：实现业务逻辑（CRUD操作）
- 数据模型层：处理数据访问和ORM映射
- 核心层：处理配置、安全等横切关注点

### 5.3 依赖管理
- 明确模块间的依赖关系
- 避免循环依赖
- 使用依赖注入降低耦合度

## 6. 配置文件管理

### 6.1 环境配置
- 不同环境使用不同的配置文件
- 敏感信息通过环境变量传递
- 配置文件应包含默认值

### 6.2 配置优先级
1. 环境变量
2. 环境配置文件
3. 默认配置

## 7. 版本控制

### 7.1 Git分支策略
- `main`：生产环境代码
- `develop`：开发环境代码
- `feature/*`：功能开发分支
- `release/*`：发布准备分支
- `hotfix/*`：紧急修复分支

### 7.2 提交信息规范
- 使用Angular提交规范
- 提交信息应包含类型、作用域和描述
- 示例：`feat(auth): add user registration`

## 8. 文档管理

### 8.1 技术文档
- 所有接口应有详细文档
- 数据库设计应有ER图和说明
- 部署和运维应有详细指南

### 8.2 代码注释
- 复杂业务逻辑应有注释说明
- 公共函数应有文档字符串
- 类和方法应有功能说明

## 9. 测试策略

### 9.1 测试分类
- 单元测试：测试独立函数和模块
- 集成测试：测试模块间交互
- 端到端测试：测试完整业务流程

### 9.2 测试覆盖率
- 核心业务逻辑覆盖率应达到90%以上
- API接口测试覆盖率应达到100%
- 数据库操作测试覆盖率应达到95%以上

## 10. 部署结构

### 10.1 生产环境
- 前端代码部署在CDN或静态资源服务器
- 后端服务部署在应用服务器
- 数据库部署在专用数据库服务器
- 使用负载均衡器分发请求

### 10.2 开发环境
- 前后端代码可在同一台服务器运行
- 使用Docker容器化部署
- 数据库可使用本地或开发专用实例

## 11. 安全考虑

### 11.1 代码安全
- 不在代码中硬编码敏感信息
- 对用户输入进行严格验证
- 使用安全的密码存储机制

### 11.2 网络安全
- 使用HTTPS加密传输
- 配置适当的CORS策略
- 防止常见的Web攻击（XSS、CSRF等）

### 11.3 数据安全
- 定期备份重要数据
- 对敏感数据进行加密存储
- 实施严格的访问控制策略

## 12. 性能优化

### 12.1 前端优化
- 压缩和合并静态资源
- 使用缓存策略
- 实施懒加载和预加载

### 12.2 后端优化
- 数据库查询优化
- 使用缓存减少数据库访问
- 实施API限流防止滥用

### 12.3 数据库优化
- 合理设计索引
- 定期分析和优化查询
- 实施读写分离

## 13. 监控和日志

### 13.1 日志记录
- 记录关键业务操作
- 记录错误和异常信息
- 日志应包含时间戳、级别和上下文信息

### 13.2 性能监控
- 监控API响应时间
- 监控数据库性能
- 监控服务器资源使用情况

### 13.3 错误追踪
- 实施统一的错误处理机制
- 记录错误堆栈信息
- 设置错误告警机制

## 14. 团队协作

### 14.1 代码规范
- 使用flake8等工具统一代码风格
- 制定并遵守团队编码规范
- 定期进行代码审查

### 14.2 开发流程
- 使用Git Flow或GitHub Flow进行版本控制
- 实施持续集成/持续部署(CI/CD)
- 定期进行技术分享和知识传递

## 15. 维护和升级

### 15.1 版本管理
- 遵循语义化版本控制规范
- 详细记录版本变更日志
- 制定向后兼容策略

### 15.2 技术债务
- 定期评估和处理技术债务
- 制定重构计划
- 平衡新功能开发和系统维护
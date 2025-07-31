# 数据库设计文档

## 1. 概述

本系统使用PostgreSQL数据库存储用户、角色、权限等相关信息。数据库设计遵循第三范式，确保数据的一致性和完整性。

## 2. 数据库表结构

### 2.1 用户表 (users)
存储用户基本信息

```sql
CREATE TABLE users (
  id BIGSERIAL PRIMARY KEY,
  username VARCHAR(50) NOT NULL UNIQUE,
  email VARCHAR(100) NOT NULL UNIQUE,
  password_hash VARCHAR(255) NOT NULL,
  nickname VARCHAR(50),
  phone VARCHAR(20),
  avatar VARCHAR(255),
  bio TEXT,
  status SMALLINT DEFAULT 1,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

COMMENT ON TABLE users IS '用户表';
COMMENT ON COLUMN users.id IS '用户ID';
COMMENT ON COLUMN users.username IS '用户名';
COMMENT ON COLUMN users.email IS '邮箱';
COMMENT ON COLUMN users.password_hash IS '密码哈希值';
COMMENT ON COLUMN users.nickname IS '昵称';
COMMENT ON COLUMN users.phone IS '手机号';
COMMENT ON COLUMN users.avatar IS '头像URL';
COMMENT ON COLUMN users.bio IS '个人简介';
COMMENT ON COLUMN users.status IS '状态：1-正常，0-禁用';
COMMENT ON COLUMN users.created_at IS '创建时间';
COMMENT ON COLUMN users.updated_at IS '更新时间';

-- 创建索引
CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_status ON users(status);

-- 创建更新时间触发器
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
   NEW.updated_at = CURRENT_TIMESTAMP;
   RETURN NEW;
END;
$$ language 'plpgsql';

CREATE TRIGGER update_users_updated_at BEFORE UPDATE ON users FOR EACH ROW EXECUTE PROCEDURE update_updated_at_column();
```

### 2.2 角色表 (roles)
存储角色信息

```sql
CREATE TABLE roles (
  id BIGSERIAL PRIMARY KEY,
  name VARCHAR(50) NOT NULL UNIQUE,
  description VARCHAR(255),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

COMMENT ON TABLE roles IS '角色表';
COMMENT ON COLUMN roles.id IS '角色ID';
COMMENT ON COLUMN roles.name IS '角色名称';
COMMENT ON COLUMN roles.description IS '角色描述';
COMMENT ON COLUMN roles.created_at IS '创建时间';
COMMENT ON COLUMN roles.updated_at IS '更新时间';

-- 创建索引
CREATE INDEX idx_roles_name ON roles(name);

-- 创建更新时间触发器
CREATE TRIGGER update_roles_updated_at BEFORE UPDATE ON roles FOR EACH ROW EXECUTE PROCEDURE update_updated_at_column();
```

### 2.3 权限表 (permissions)
存储权限信息

```sql
CREATE TABLE permissions (
  id BIGSERIAL PRIMARY KEY,
  code VARCHAR(50) NOT NULL UNIQUE,
  name VARCHAR(100) NOT NULL,
  description VARCHAR(255),
  category VARCHAR(50),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

COMMENT ON TABLE permissions IS '权限表';
COMMENT ON COLUMN permissions.id IS '权限ID';
COMMENT ON COLUMN permissions.code IS '权限代码';
COMMENT ON COLUMN permissions.name IS '权限名称';
COMMENT ON COLUMN permissions.description IS '权限描述';
COMMENT ON COLUMN permissions.category IS '权限分类';
COMMENT ON COLUMN permissions.created_at IS '创建时间';
COMMENT ON COLUMN permissions.updated_at IS '更新时间';

-- 创建索引
CREATE INDEX idx_permissions_code ON permissions(code);
CREATE INDEX idx_permissions_category ON permissions(category);

-- 创建更新时间触发器
CREATE TRIGGER update_permissions_updated_at BEFORE UPDATE ON permissions FOR EACH ROW EXECUTE PROCEDURE update_updated_at_column();
```

### 2.4 用户角色关联表 (user_roles)
存储用户与角色的多对多关系

```sql
CREATE TABLE user_roles (
  id BIGSERIAL PRIMARY KEY,
  user_id BIGINT NOT NULL,
  role_id BIGINT NOT NULL,
  assigned_by BIGINT,
  assigned_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(user_id, role_id),
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
  FOREIGN KEY (role_id) REFERENCES roles(id) ON DELETE CASCADE
);

COMMENT ON TABLE user_roles IS '用户角色关联表';
COMMENT ON COLUMN user_roles.id IS '关联ID';
COMMENT ON COLUMN user_roles.user_id IS '用户ID';
COMMENT ON COLUMN user_roles.role_id IS '角色ID';
COMMENT ON COLUMN user_roles.assigned_by IS '分配人ID';
COMMENT ON COLUMN user_roles.assigned_at IS '分配时间';

-- 创建索引
CREATE INDEX idx_user_roles_user_id ON user_roles(user_id);
CREATE INDEX idx_user_roles_role_id ON user_roles(role_id);
```

### 2.5 角色权限关联表 (role_permissions)
存储角色与权限的多对多关系

```sql
CREATE TABLE role_permissions (
  id BIGSERIAL PRIMARY KEY,
  role_id BIGINT NOT NULL,
  permission_id BIGINT NOT NULL,
  assigned_by BIGINT,
  assigned_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
  
  UNIQUE(role_id, permission_id),
  FOREIGN KEY (role_id) REFERENCES roles(id) ON DELETE CASCADE,
  FOREIGN KEY (permission_id) REFERENCES permissions(id) ON DELETE CASCADE
);

COMMENT ON TABLE role_permissions IS '角色权限关联表';
COMMENT ON COLUMN role_permissions.id IS '关联ID';
COMMENT ON COLUMN role_permissions.role_id IS '角色ID';
COMMENT ON COLUMN role_permissions.permission_id IS '权限ID';
COMMENT ON COLUMN role_permissions.assigned_by IS '分配人ID';
COMMENT ON COLUMN role_permissions.assigned_at IS '分配时间';

-- 创建索引
CREATE INDEX idx_role_permissions_role_id ON role_permissions(role_id);
CREATE INDEX idx_role_permissions_permission_id ON role_permissions(permission_id);
```

### 2.6 用户会话表 (user_sessions)
存储用户会话信息

```sql
CREATE TABLE user_sessions (
  id BIGSERIAL PRIMARY KEY,
  user_id BIGINT NOT NULL,
  token VARCHAR(255) NOT NULL UNIQUE,
  refresh_token VARCHAR(255) UNIQUE,
  expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
  ip_address VARCHAR(45),
  user_agent TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
  
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

COMMENT ON TABLE user_sessions IS '用户会话表';
COMMENT ON COLUMN user_sessions.id IS '会话ID';
COMMENT ON COLUMN user_sessions.user_id IS '用户ID';
COMMENT ON COLUMN user_sessions.token IS '访问令牌';
COMMENT ON COLUMN user_sessions.refresh_token IS '刷新令牌';
COMMENT ON COLUMN user_sessions.expires_at IS '过期时间';
COMMENT ON COLUMN user_sessions.ip_address IS 'IP地址';
COMMENT ON COLUMN user_sessions.user_agent IS '用户代理';
COMMENT ON COLUMN user_sessions.created_at IS '创建时间';

-- 创建索引
CREATE INDEX idx_user_sessions_token ON user_sessions(token);
CREATE INDEX idx_user_sessions_refresh_token ON user_sessions(refresh_token);
CREATE INDEX idx_user_sessions_user_id ON user_sessions(user_id);
CREATE INDEX idx_user_sessions_expires_at ON user_sessions(expires_at);
```

## 3. 初始数据

### 3.1 初始角色
```sql
INSERT INTO roles (name, description) VALUES 
('user', '普通用户'),
('admin', '管理员'),
('super_admin', '超级管理员');
```

### 3.2 初始权限
```sql
INSERT INTO permissions (code, name, description, category) VALUES
-- 用户管理权限
('USER_VIEW', '查看用户', '查看用户列表权限', '用户管理'),
('USER_EDIT', '编辑用户', '编辑用户信息权限', '用户管理'),
('USER_DELETE', '删除用户', '删除用户权限', '用户管理'),

-- 内容管理权限
('CONTENT_VIEW', '查看内容', '查看内容权限', '内容管理'),
('CONTENT_CREATE', '创建内容', '创建内容权限', '内容管理'),
('CONTENT_EDIT', '编辑内容', '编辑内容权限', '内容管理'),
('CONTENT_DELETE', '删除内容', '删除内容权限', '内容管理'),

-- 系统管理权限
('SYSTEM_CONFIG', '系统配置', '系统配置权限', '系统管理'),
('ROLE_MANAGE', '角色管理', '角色管理权限', '系统管理'),
('PERMISSION_MANAGE', '权限管理', '权限管理权限', '系统管理');
```

### 3.3 角色权限分配
```sql
-- 普通用户权限
INSERT INTO role_permissions (role_id, permission_id) 
SELECT r.id, p.id 
FROM roles r, permissions p 
WHERE r.name = 'user' 
AND p.code IN ('USER_VIEW', 'CONTENT_VIEW');

-- 管理员权限
INSERT INTO role_permissions (role_id, permission_id) 
SELECT r.id, p.id 
FROM roles r, permissions p 
WHERE r.name = 'admin' 
AND p.code IN ('USER_VIEW', 'USER_EDIT', 'CONTENT_VIEW', 'CONTENT_CREATE', 'CONTENT_EDIT');

-- 超级管理员权限
INSERT INTO role_permissions (role_id, permission_id) 
SELECT r.id, p.id 
FROM roles r, permissions p 
WHERE r.name = 'super_admin';
```

## 4. 数据库索引优化

### 4.1 查询优化
1. 在用户表的username和email字段上创建索引，提高登录查询效率
2. 在角色表的name字段上创建索引，提高角色查询效率
3. 在权限表的code字段上创建索引，提高权限查询效率
4. 在用户角色关联表和角色权限关联表上创建复合索引，提高关联查询效率

### 4.2 分页查询优化
对于需要分页查询的表（如用户列表），使用覆盖索引优化分页查询性能。

## 5. 数据库安全

### 5.1 数据加密
1. 用户密码使用bcrypt进行哈希存储
2. 敏感信息（如手机号）在应用层进行加密存储

### 5.2 SQL注入防护
1. 使用参数化查询
2. 对用户输入进行严格验证和过滤

### 5.3 访问控制
1. 数据库用户权限最小化原则
2. 应用程序使用专用数据库用户

## 6. 数据库备份与恢复

### 6.1 备份策略
1. 每日全量备份
2. 每小时增量备份
3. 备份数据异地存储

### 6.2 恢复方案
1. 提供备份数据恢复脚本
2. 定期测试备份数据完整性

## 7. 数据库迁移

### 7.1 迁移工具
使用Alembic作为数据库迁移工具，管理数据库模式的版本控制。

### 7.2 迁移流程
1. 创建迁移脚本
2. 审查迁移脚本
3. 在开发环境测试迁移
4. 在生产环境执行迁移
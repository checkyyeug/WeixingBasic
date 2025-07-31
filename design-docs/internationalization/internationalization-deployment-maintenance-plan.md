# 国际化功能部署和维护方案

## 1. 概述

### 1.1 目的

本文档旨在提供国际化功能的部署和维护指南，确保国际化功能能够顺利部署到生产环境并保持稳定运行。通过详细的部署步骤、配置说明和维护策略，帮助运维团队高效管理国际化功能。

### 1.2 范围

本文档涵盖以下内容：

1. **部署环境要求**：国际化功能部署所需的硬件和软件环境
2. **部署步骤**：国际化功能的详细部署流程
3. **配置管理**：国际化功能的配置管理和最佳实践
4. **监控和日志**：国际化功能的监控策略和日志管理
5. **维护策略**：国际化功能的日常维护和更新策略
6. **故障处理**：国际化功能常见问题的诊断和解决方案
7. **性能优化**：国际化功能的性能优化建议

### 1.3 目标读者

本文档主要面向以下人员：

1. **运维工程师**：负责国际化功能的部署和维护
2. **系统管理员**：负责系统环境的配置和管理
3. **开发工程师**：协助解决国际化功能的技术问题
4. **项目经理**：了解国际化功能的部署和维护流程

## 2. 部署环境要求

### 2.1 硬件环境要求

#### 2.1.1 服务器要求

1. **应用服务器**：
   - CPU：4核及以上
   - 内存：8GB及以上
   - 存储：100GB及以上，SSD推荐
   - 网络：千兆网卡

2. **数据库服务器**：
   - CPU：8核及以上
   - 内存：16GB及以上
   - 存储：200GB及以上，SSD推荐
   - 网络：千兆网卡

3. **缓存服务器**：
   - CPU：4核及以上
   - 内存：8GB及以上
   - 存储：50GB及以上，SSD推荐
   - 网络：千兆网卡

#### 2.1.2 负载均衡要求

1. **负载均衡器**：
   - 支持HTTP/HTTPS协议
   - 支持会话保持
   - 支持健康检查
   - 支持SSL终止

2. **CDN要求**：
   - 支持静态资源缓存
   - 支持多地域分发
   - 支持HTTPS
   - 支持缓存刷新

### 2.2 软件环境要求

#### 2.2.1 操作系统要求

1. **服务器操作系统**：
   - Linux：Ubuntu 20.04 LTS或CentOS 8及以上
   - Windows Server：2019及以上（不推荐）

2. **客户端支持**：
   - 微信：最新稳定版
   - 浏览器：Chrome 90+、Firefox 88+、Safari 14+、Edge 90+
   - 移动设备：iOS 14+、Android 10+

#### 2.2.2 中间件要求

1. **Web服务器**：
   - Nginx：1.18及以上
   - Apache：2.4及以上（不推荐）

2. **应用服务器**：
   - Node.js：16.14.0及以上
   - Python：3.9及以上
   - PM2：5.2.0及以上（Node.js进程管理）

3. **数据库**：
   - PostgreSQL：13及以上
   - Redis：6.0及以上

4. **缓存**：
   - Redis：6.0及以上
   - Memcached：1.6及以上（可选）

#### 2.2.3 开发和部署工具

1. **版本控制**：
   - Git：2.30及以上

2. **CI/CD工具**：
   - Jenkins：2.300及以上
   - GitHub Actions：最新版本
   - GitLab CI：13.0及以上

3. **容器化工具**：
   - Docker：20.10及以上
   - Kubernetes：1.22及以上（可选）
   - Docker Compose：1.29及以上

### 2.3 网络环境要求

#### 2.3.1 网络拓扑

```
[用户] -> [CDN] -> [负载均衡器] -> [应用服务器集群] -> [数据库服务器]
                                      |
                                      v
                                 [缓存服务器]
```

#### 2.3.2 端口要求

1. **入站端口**：
   - 80（HTTP）
   - 443（HTTPS）
   - 22（SSH，仅限内网）

2. **出站端口**：
   - 80（HTTP）
   - 443（HTTPS）
   - 5432（PostgreSQL，仅限内网）
   - 6379（Redis，仅限内网）

#### 2.3.3 带宽要求

1. **互联网带宽**：
   - 峰值带宽：100Mbps及以上
   - 平均带宽：50Mbps及以上

2. **内网带宽**：
   - 应用服务器到数据库服务器：1Gbps及以上
   - 应用服务器到缓存服务器：1Gbps及以上

## 3. 部署步骤

### 3.1 部署前准备

#### 3.1.1 环境检查

1. **检查服务器环境**：
```bash
# 检查操作系统版本
cat /etc/os-release

# 检查CPU信息
lscpu

# 检查内存信息
free -h

# 检查磁盘空间
df -h

# 检查网络连接
ping -c 4 google.com
```

2. **检查软件环境**：
```bash
# 检查Node.js版本
node --version

# 检查Python版本
python --version

# 检查PostgreSQL版本
psql --version

# 检查Redis版本
redis-server --version
```

3. **检查网络环境**：
```bash
# 检查端口占用
netstat -tlnp | grep :80
netstat -tlnp | grep :443

# 检查防火墙状态
sudo ufw status
```

#### 3.1.2 数据准备

1. **准备翻译资源**：
```bash
# 创建翻译资源目录
mkdir -p /var/www/i18n/locales

# 复制翻译资源文件
cp -r ./i18n/locales/* /var/www/i18n/locales/

# 设置翻译资源权限
chown -R www-data:www-data /var/www/i18n
chmod -R 755 /var/www/i18n
```

2. **准备数据库**：
```sql
-- 创建国际化相关表
CREATE TABLE user_locales (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL UNIQUE,
    locale VARCHAR(10) NOT NULL DEFAULT 'zh-CN',
    is_default BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

CREATE TABLE translations (
    id BIGSERIAL PRIMARY KEY,
    key VARCHAR(255) NOT NULL,
    locale VARCHAR(10) NOT NULL,
    value TEXT NOT NULL,
    context VARCHAR(255),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    UNIQUE(key, locale)
);

-- 创建索引
CREATE INDEX idx_user_locales_user_id ON user_locales(user_id);
CREATE INDEX idx_user_locales_locale ON user_locales(locale);
CREATE INDEX idx_translations_key ON translations(key);
CREATE INDEX idx_translations_locale ON translations(locale);
CREATE INDEX idx_translations_key_locale ON translations(key, locale);

-- 创建更新时间触发器
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ language 'plpgsql';

CREATE TRIGGER update_user_locales_updated_at BEFORE UPDATE ON user_locales FOR EACH ROW EXECUTE PROCEDURE update_updated_at_column();
CREATE TRIGGER update_translations_updated_at BEFORE UPDATE ON translations FOR EACH ROW EXECUTE PROCEDURE update_updated_at_column();
```

3. **准备缓存配置**：
```bash
# 配置Redis缓存
cat > /etc/redis/redis.conf << EOF
maxmemory 512mb
maxmemory-policy allkeys-lru
save 900 1
save 300 10
save 60 10000
EOF

# 重启Redis服务
sudo systemctl restart redis
```

#### 3.1.3 配置文件准备

1. **应用配置文件**：
```javascript
// config/i18n.js
module.exports = {
  // 支持的语言列表
  supportedLocales: ['en', 'zh-CN', 'zh-TW'],
  
  // 默认语言
  defaultLocale: 'zh-CN',
  
  // 回退语言
  fallbackLocale: 'zh-CN',
  
  // 翻译资源配置
  translations: {
    // 本地翻译资源路径
    localPath: '/var/www/i18n/locales',
    
    // 远程翻译资源API
    remoteApi: {
      baseUrl: 'https://api.example.com/i18n',
      timeout: 5000,
      retries: 3
    },
    
    // 缓存配置
    cache: {
      enabled: true,
      ttl: 3600, // 1小时
      maxSize: 100 // 最大缓存语言数量
    }
  },
  
  // 语言切换配置
  languageSwitcher: {
    // 显示模式：'dropdown' | 'modal' | 'page'
    displayMode: 'dropdown',
    
    // 显示标志：true | false
    showFlags: true,
    
    // 显示语言名称：true | false
    showNames: true,
    
    // 自动检测用户语言：true | false
    autoDetect: true
  }
};
```

2. **数据库配置文件**：
```javascript
// config/database.js
module.exports = {
  // 数据库连接配置
  connection: {
    host: 'localhost',
    port: 5432,
    database: 'user_management',
    username: 'db_user',
    password: 'db_password',
    pool: {
      min: 2,
      max: 10,
      idle: 30000,
      acquire: 60000
    }
  },
  
  // 国际化相关表配置
  tables: {
    userLocales: 'user_locales',
    translations: 'translations'
  }
};
```

3. **缓存配置文件**：
```javascript
// config/cache.js
module.exports = {
  // Redis配置
  redis: {
    host: 'localhost',
    port: 6379,
    password: '',
    db: 0,
    
    // 连接池配置
    pool: {
      min: 2,
      max: 10
    },
    
    // 国际化缓存配置
    i18n: {
      prefix: 'i18n:',
      ttl: 3600, // 1小时
      maxSize: 100 // 最大缓存语言数量
    }
  }
};
```

4. **日志配置文件**：
```javascript
// config/logging.js
module.exports = {
  // 日志级别
  level: 'info',
  
  // 日志格式
  format: 'combined',
  
  // 日志输出
  transports: [
    // 控制台输出
    {
      type: 'console',
      level: 'info'
    },
    
    // 文件输出
    {
      type: 'file',
      level: 'info',
      filename: '/var/log/app/i18n.log',
      maxsize: 10485760, // 10MB
      maxFiles: 5
    }
  ],
  
  // 国际化相关日志配置
  i18n: {
    // 记录语言切换事件
    logLanguageSwitch: true,
    
    // 记录翻译资源加载事件
    logTranslationLoad: true,
    
    // 记录翻译缺失事件
    logMissingTranslation: true
  }
};
```

### 3.2 部署流程

#### 3.2.1 前端部署

1. **构建前端应用**：
```bash
# 安装依赖
npm install

# 构建生产版本
npm run build

# 构建多语言版本
npm run build:i18n
```

2. **部署前端资源**：
```bash
# 创建前端资源目录
mkdir -p /var/www/frontend

# 复制构建后的文件
cp -r dist/* /var/www/frontend/

# 设置权限
chown -R www-data:www-data /var/www/frontend
chmod -R 755 /var/www/frontend
```

3. **配置Nginx**：
```nginx
# /etc/nginx/sites-available/frontend
server {
    listen 80;
    server_name example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com;
    
    # SSL配置
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    
    # 前端资源
    root /var/www/frontend;
    index index.html;
    
    # 翻译资源
    location /i18n/ {
        alias /var/www/i18n/;
        expires 1h;
        add_header Cache-Control "public, immutable";
    }
    
    # 静态资源缓存
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    # 其他请求代理到后端
    location / {
        try_files $uri $uri/ /index.html;
    }
    
    # API代理
    location /api/ {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

4. **启用站点**：
```bash
# 启用站点
sudo ln -s /etc/nginx/sites-available/frontend /etc/nginx/sites-enabled/

# 测试Nginx配置
sudo nginx -t

# 重启Nginx
sudo systemctl restart nginx
```

#### 3.2.2 后端部署

1. **安装后端依赖**：
```bash
# 创建虚拟环境
python -m venv venv

# 激活虚拟环境
source venv/bin/activate

# 安装依赖
pip install -r requirements.txt
```

2. **配置环境变量**：
```bash
# /etc/environment.d/i18n.conf
export DB_HOST=localhost
export DB_PORT=5432
export DB_NAME=user_management
export DB_USER=db_user
export DB_PASSWORD=db_password

export REDIS_HOST=localhost
export REDIS_PORT=6379
export REDIS_PASSWORD=

export I18N_DEFAULT_LOCALE=zh-CN
export I18N_SUPPORTED_LOCALES=en,zh-CN,zh-TW
export I18N_FALLBACK_LOCALE=zh-CN

export LOG_LEVEL=info
export LOG_FILE=/var/log/app/i18n.log
```

3. **初始化数据库**：
```bash
# 运行数据库迁移
python manage.py migrate

# 加载初始翻译数据
python manage.py load_translations

# 创建超级用户
python manage.py createsuperuser
```

4. **配置应用服务**：
```ini
# /etc/systemd/system/i18n-backend.service
[Unit]
Description=I18n Backend Service
After=network.target

[Service]
Type=exec
User=www-data
Group=www-data
WorkingDirectory=/var/www/backend
Environment=PATH=/var/www/backend/venv/bin
ExecStart=/var/www/backend/venv/bin/gunicorn -w 4 -k uvicorn.workers.UvicornWorker main:app
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

5. **启动后端服务**：
```bash
# 重新加载systemd配置
sudo systemctl daemon-reload

# 启动服务
sudo systemctl start i18n-backend

# 设置开机自启
sudo systemctl enable i18n-backend

# 检查服务状态
sudo systemctl status i18n-backend
```

#### 3.2.3 缓存部署

1. **配置Redis服务**：
```ini
# /etc/systemd/system/redis-i18n.service
[Unit]
Description=Redis I18n Cache Server
After=network.target

[Service]
Type=exec
User=redis
Group=redis
ExecStart=/usr/bin/redis-server /etc/redis/redis-i18n.conf
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

2. **创建Redis配置文件**：
```bash
# /etc/redis/redis-i18n.conf
port 6379
bind 127.0.0.1
maxmemory 512mb
maxmemory-policy allkeys-lru
save 900 1
save 300 10
save 60 10000
```

3. **启动Redis服务**：
```bash
# 重新加载systemd配置
sudo systemctl daemon-reload

# 启动服务
sudo systemctl start redis-i18n

# 设置开机自启
sudo systemctl enable redis-i18n

# 检查服务状态
sudo systemctl status redis-i18n
```

### 3.3 部署后验证

#### 3.3.1 功能验证

1. **验证语言切换功能**：
```bash
# 测试语言切换API
curl -X POST https://example.com/api/user/locale \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{"locale": "en"}'

# 检查响应
{
  "code": 200,
  "message": "Locale updated successfully",
  "data": {
    "locale": "en"
  }
}
```

2. **验证翻译资源加载**：
```bash
# 测试翻译资源API
curl https://example.com/api/i18n/locales/en

# 检查响应
{
  "code": 200,
  "message": "Success",
  "data": {
    "common": {
      "ok": "OK",
      "cancel": "Cancel",
      ...
    },
    ...
  }
}
```

3. **验证多语言数据**：
```bash
# 测试多语言角色数据
curl https://example.com/api/roles?locale=en

# 检查响应
{
  "code": 200,
  "message": "Success",
  "data": [
    {
      "id": 1,
      "name": "User",
      "description": "Regular user with basic permissions",
      ...
    },
    ...
  ]
}
```

#### 3.3.2 性能验证

1. **验证语言切换性能**：
```bash
# 使用ab测试语言切换性能
ab -n 1000 -c 100 -H "Authorization: Bearer YOUR_TOKEN" \
  -p locale.json -T application/json \
  https://example.com/api/user/locale

# 检查结果
# 请求处理时间应小于500ms
# 成功率应大于99%
```

2. **验证翻译资源加载性能**：
```bash
# 使用ab测试翻译资源加载性能
ab -n 1000 -c 100 https://example.com/api/i18n/locales/en

# 检查结果
# 请求处理时间应小于1000ms
# 成功率应大于99%
```

3. **验证缓存效果**：
```bash
# 检查Redis缓存
redis-cli KEYS "i18n:*"

# 检查缓存命中率
redis-cli INFO stats | grep keyspace
```

## 4. 配置管理

### 4.1 配置文件管理

#### 4.1.1 配置文件结构

```
config/
├── app.js              # 应用主配置
├── i18n.js             # 国际化配置
├── database.js         # 数据库配置
├── cache.js            # 缓存配置
├── logging.js          # 日志配置
└── environments/       # 环境特定配置
    ├── development.js  # 开发环境配置
    ├── testing.js      # 测试环境配置
    └── production.js   # 生产环境配置
```

#### 4.1.2 配置文件示例

1. **应用主配置**：
```javascript
// config/app.js
const env = process.env.NODE_ENV || 'development';
const envConfig = require(`./environments/${env}`);

module.exports = {
  // 应用基础配置
  name: 'User Management System',
  version: '1.0.0',
  env: env,
  
  // 服务器配置
  server: {
    port: process.env.PORT || 3000,
    host: process.env.HOST || '0.0.0.0'
  },
  
  // 安全配置
  security: {
    jwtSecret: process.env.JWT_SECRET || 'your-secret-key',
    jwtExpiration: process.env.JWT_EXPIRATION || '24h',
    bcryptRounds: 12
  },
  
  // 国际化配置
  i18n: require('./i18n'),
  
  // 数据库配置
  database: require('./database'),
  
  // 缓存配置
  cache: require('./cache'),
  
  // 日志配置
  logging: require('./logging'),
  
  // 环境特定配置
  ...envConfig
};
```

2. **生产环境配置**：
```javascript
// config/environments/production.js
module.exports = {
  // 生产环境特定配置
  server: {
    port: 3000,
    host: '0.0.0.0'
  },
  
  // 安全配置
  security: {
    jwtSecret: process.env.JWT_SECRET,
    jwtExpiration: '24h',
    bcryptRounds: 12
  },
  
  // 国际化配置
  i18n: {
    translations: {
      cache: {
        enabled: true,
        ttl: 3600,
        maxSize: 100
      }
    }
  },
  
  // 数据库配置
  database: {
    connection: {
      host: process.env.DB_HOST,
      port: process.env.DB_PORT,
      database: process.env.DB_NAME,
      username: process.env.DB_USER,
      password: process.env.DB_PASSWORD,
      pool: {
        min: 5,
        max: 20,
        idle: 30000,
        acquire: 60000
      }
    }
  },
  
  // 缓存配置
  cache: {
    redis: {
      host: process.env.REDIS_HOST,
      port: process.env.REDIS_PORT,
      password: process.env.REDIS_PASSWORD,
      pool: {
        min: 5,
        max: 20
      }
    }
  },
  
  // 日志配置
  logging: {
    level: 'info',
    transports: [
      {
        type: 'file',
        level: 'info',
        filename: '/var/log/app/i18n.log',
        maxsize: 10485760,
        maxFiles: 5
      }
    ]
  }
};
```

### 4.2 环境变量管理

#### 4.2.1 环境变量列表

| 变量名 | 描述 | 默认值 | 生产环境示例 |
|-------|------|--------|------------|
| NODE_ENV | 运行环境 | development | production |
| PORT | 服务端口 | 3000 | 3000 |
| HOST | 服务主机 | 0.0.0.0 | 0.0.0.0 |
| JWT_SECRET | JWT密钥 | your-secret-key | complex-secret-key |
| JWT_EXPIRATION | JWT过期时间 | 24h | 24h |
| DB_HOST | 数据库主机 | localhost | db.example.com |
| DB_PORT | 数据库端口 | 5432 | 5432 |
| DB_NAME | 数据库名称 | user_management | user_management |
| DB_USER | 数据库用户名 | db_user | prod_db_user |
| DB_PASSWORD | 数据库密码 | db_password | complex-db-password |
| REDIS_HOST | Redis主机 | localhost | redis.example.com |
| REDIS_PORT | Redis端口 | 6379 | 6379 |
| REDIS_PASSWORD | Redis密码 | | complex-redis-password |
| I18N_DEFAULT_LOCALE | 默认语言 | zh-CN | zh-CN |
| I18N_SUPPORTED_LOCALES | 支持的语言列表 | en,zh-CN,zh-TW | en,zh-CN,zh-TW |
| I18N_FALLBACK_LOCALE | 回退语言 | zh-CN | zh-CN |
| LOG_LEVEL | 日志级别 | info | info |
| LOG_FILE | 日志文件路径 | ./logs/i18n.log | /var/log/app/i18n.log |

#### 4.2.2 环境变量管理最佳实践

1. **使用.env文件**：
```bash
# .env.production
NODE_ENV=production
PORT=3000
HOST=0.0.0.0
JWT_SECRET=complex-secret-key
JWT_EXPIRATION=24h
DB_HOST=db.example.com
DB_PORT=5432
DB_NAME=user_management
DB_USER=prod_db_user
DB_PASSWORD=complex-db-password
REDIS_HOST=redis.example.com
REDIS_PORT=6379
REDIS_PASSWORD=complex-redis-password
I18N_DEFAULT_LOCALE=zh-CN
I18N_SUPPORTED_LOCALES=en,zh-CN,zh-TW
I18N_FALLBACK_LOCALE=zh-CN
LOG_LEVEL=info
LOG_FILE=/var/log/app/i18n.log
```

2. **使用环境变量管理工具**：
```bash
# 安装dotenv
npm install dotenv

# 在应用入口加载环境变量
require('dotenv').config({ path: `.env.${process.env.NODE_ENV}` });
```

3. **敏感信息保护**：
```bash
# 将.env文件添加到.gitignore
echo ".env*" >> .gitignore

# 使用密钥管理服务存储敏感信息
# 例如：AWS Secrets Manager、HashiCorp Vault
```

### 4.3 配置版本控制

#### 4.3.1 配置版本控制策略

1. **配置文件分类**：
   - **共享配置**：所有环境共享的配置，提交到版本控制
   - **环境特定配置**：特定环境的配置，使用环境变量管理
   - **敏感配置**：包含敏感信息的配置，使用密钥管理服务

2. **配置文件模板**：
```javascript
// config/environments/production.js.template
module.exports = {
  database: {
    connection: {
      host: '{{DB_HOST}}',
      port: '{{DB_PORT}}',
      database: '{{DB_NAME}}',
      username: '{{DB_USER}}',
      password: '{{DB_PASSWORD}}',
      pool: {
        min: 5,
        max: 20,
        idle: 30000,
        acquire: 60000
      }
    }
  },
  
  cache: {
    redis: {
      host: '{{REDIS_HOST}}',
      port: '{{REDIS_PORT}}',
      password: '{{REDIS_PASSWORD}}',
      pool: {
        min: 5,
        max: 20
      }
    }
  }
};
```

3. **配置部署流程**：
```bash
# 1. 从版本控制拉取配置模板
git pull origin main

# 2. 使用环境变量渲染配置模板
node render-config.js

# 3. 部署应用
npm run deploy
```

#### 4.3.2 配置变更管理

1. **配置变更流程**：
   - 提交配置变更请求
   - 代码审查
   - 测试环境验证
   - 生产环境部署
   - 监控和回滚

2. **配置变更通知**：
   - 邮件通知
   - 即时消息通知
   - 监控告警

3. **配置变更回滚**：
   - 版本控制回滚
   - 环境变量回滚
   - 密钥管理服务回滚

## 5. 监控和日志

### 5.1 监控策略

#### 5.1.1 监控指标

1. **系统指标**：
   - CPU使用率
   - 内存使用率
   - 磁盘使用率
   - 网络流量

2. **应用指标**：
   - 请求响应时间
   - 请求成功率
   - 错误率
   - 并发连接数

3. **国际化特定指标**：
   - 语言切换请求量
   - 语言切换成功率
   - 翻译资源加载时间
   - 翻译资源缓存命中率
   - 多语言数据查询时间

#### 5.1.2 监控工具配置

1. **Prometheus配置**：
```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'i18n-backend'
    static_configs:
      - targets: ['localhost:3000']
    metrics_path: '/metrics'
    scrape_interval: 5s

  - job_name: 'redis'
    static_configs:
      - targets: ['localhost:6379']
    scrape_interval: 5s

  - job_name: 'postgres'
    static_configs:
      - targets: ['localhost:5432']
    scrape_interval: 5s
```

2. **Grafana仪表板配置**：
```json
{
  "dashboard": {
    "id": null,
    "title": "国际化功能监控",
    "tags": ["i18n"],
    "timezone": "Asia/Shanghai",
    "panels": [
      {
        "id": 1,
        "title": "语言切换请求量",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(i18n_language_switch_requests_total[5m])",
            "legendFormat": "{{locale}}"
          }
        ]
      },
      {
        "id": 2,
        "title": "语言切换成功率",
        "type": "singlestat",
        "targets": [
          {
            "expr": "rate(i18n_language_switch_success_total[5m]) / rate(i18n_language_switch_requests_total[5m]) * 100",
            "legendFormat": "成功率"
          }
        ]
      },
      {
        "id": 3,
        "title": "翻译资源加载时间",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(i18n_translation_load_duration_seconds_bucket[5m]))",
            "legendFormat": "95分位数"
          },
          {
            "expr": "histogram_quantile(0.50, rate(i18n_translation_load_duration_seconds_bucket[5m]))",
            "legendFormat": "50分位数"
          }
        ]
      },
      {
        "id": 4,
        "title": "翻译资源缓存命中率",
        "type": "singlestat",
        "targets": [
          {
            "expr": "rate(i18n_translation_cache_hits_total[5m]) / (rate(i18n_translation_cache_hits_total[5m]) + rate(i18n_translation_cache_misses_total[5m])) * 100",
            "legendFormat": "命中率"
          }
        ]
      }
    ]
  }
}
```

3. **应用监控集成**：
```javascript
// metrics.js
const client = require('prom-client');

// 创建默认注册表
const register = new client.Registry();

// 添加默认指标
client.collectDefaultMetrics({ register });

// 创建国际化特定指标
const languageSwitchRequests = new client.Counter({
  name: 'i18n_language_switch_requests_total',
  help: 'Total number of language switch requests',
  labelNames: ['locale', 'status']
});

const languageSwitchDuration = new client.Histogram({
  name: 'i18n_language_switch_duration_seconds',
  help: 'Duration of language switch requests in seconds',
  labelNames: ['locale'],
  buckets: [0.1, 0.5, 1, 2, 5]
});

const translationLoadRequests = new client.Counter({
  name: 'i18n_translation_load_requests_total',
  help: 'Total number of translation load requests',
  labelNames: ['locale', 'status']
});

const translationLoadDuration = new client.Histogram({
  name: 'i18n_translation_load_duration_seconds',
  help: 'Duration of translation load requests in seconds',
  labelNames: ['locale'],
  buckets: [0.1, 0.5, 1, 2, 5, 10]
});

const translationCacheHits = new client.Counter({
  name: 'i18n_translation_cache_hits_total',
  help: 'Total number of translation cache hits',
  labelNames: ['locale']
});

const translationCacheMisses = new client.Counter({
  name: 'i18n_translation_cache_misses_total',
  help: 'Total number of translation cache misses',
  labelNames: ['locale']
});

module.exports = {
  register,
  languageSwitchRequests,
  languageSwitchDuration,
  translationLoadRequests,
  translationLoadDuration,
  translationCacheHits,
  translationCacheMisses
};
```

### 5.2 日志管理

#### 5.2.1 日志策略

1. **日志级别**：
   - ERROR：严重错误，需要立即处理
   - WARN：警告信息，需要关注
   - INFO：一般信息，记录系统运行状态
   - DEBUG：调试信息，用于问题排查

2. **日志格式**：
```json
{
  "timestamp": "2023-07-20T10:30:00.000Z",
  "level": "info",
  "message": "Language switched successfully",
  "meta": {
    "userId": "12345",
    "fromLocale": "zh-CN",
    "toLocale": "en",
    "duration": 120,
    "ip": "192.168.1.100"
  }
}
```

3. **日志分类**：
   - 系统日志：系统运行状态和错误信息
   - 访问日志：用户访问记录
   - 业务日志：业务操作记录
   - 国际化日志：国际化相关操作记录

#### 5.2.2 日志配置

1. **日志配置文件**：
```javascript
// logging.js
const winston = require('winston');
const { combine, timestamp, printf, colorize, errors } = winston.format;

// 自定义日志格式
const logFormat = printf(({ level, message, timestamp, stack, ...meta }) => {
  return JSON.stringify({
    timestamp,
    level,
    message,
    stack,
    meta
  });
});

// 创建日志记录器
const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: combine(
    errors({ stack: true }),
    timestamp(),
    logFormat
  ),
  transports: [
    // 控制台输出
    new winston.transports.Console({
      format: combine(
        colorize(),
        errors({ stack: true }),
        timestamp(),
        printf(({ level, message, timestamp, stack }) => {
          return `${timestamp} [${level}]: ${stack || message}`;
        })
      )
    }),
    
    // 文件输出
    new winston.transports.File({
      filename: process.env.LOG_FILE || './logs/i18n.log',
      maxsize: 10485760, // 10MB
      maxFiles: 5
    })
  ]
});

// 国际化特定日志记录器
const i18nLogger = winston.createLogger({
  level: 'info',
  format: combine(
    errors({ stack: true }),
    timestamp(),
    logFormat
  ),
  transports: [
    new winston.transports.File({
      filename: process.env.I18N_LOG_FILE || './logs/i18n-specific.log',
      maxsize: 10485760, // 10MB
      maxFiles: 5
    })
  ]
});

module.exports = {
  logger,
  i18nLogger
};
```

2. **国际化日志记录**：
```javascript
// i18n-logger.js
const { i18nLogger } = require('./logging');

// 记录语言切换事件
function logLanguageSwitch(userId, fromLocale, toLocale, duration, ip) {
  i18nLogger.info('Language switched', {
    category: 'language_switch',
    userId,
    fromLocale,
    toLocale,
    duration,
    ip
  });
}

// 记录翻译资源加载事件
function logTranslationLoad(locale, source, duration, success) {
  i18nLogger.info('Translation loaded', {
    category: 'translation_load',
    locale,
    source,
    duration,
    success
  });
}

// 记录翻译缺失事件
function logMissingTranslation(key, locale) {
  i18nLogger.warn('Missing translation', {
    category: 'missing_translation',
    key,
    locale
  });
}

// 记录缓存命中事件
function logCacheHit(locale, key) {
  i18nLogger.debug('Cache hit', {
    category: 'cache',
    locale,
    key,
    type: 'hit'
  });
}

// 记录缓存未命中事件
function logCacheMiss(locale, key) {
  i18nLogger.debug('Cache miss', {
    category: 'cache',
    locale,
    key,
    type: 'miss'
  });
}

module.exports = {
  logLanguageSwitch,
  logTranslationLoad,
  logMissingTranslation,
  logCacheHit,
  logCacheMiss
};
```

#### 5.2.3 日志收集和分析

1. **ELK Stack配置**：
```yaml
# filebeat.yml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/app/i18n.log
    - /var/log/app/i18n-specific.log
  json.keys_under_root: true
  json.add_error_key: true

output.elasticsearch:
  hosts: ["elasticsearch:9200"]
  indices:
    - index: "i18n-logs-%{+yyyy.MM.dd}"
```

2. **日志分析查询**：
```json
// 查询语言切换事件
{
  "query": {
    "match": {
      "meta.category": "language_switch"
    }
  },
  "aggs": {
    "locales": {
      "terms": {
        "field": "meta.toLocale.keyword"
      }
    },
    "avg_duration": {
      "avg": {
        "field": "meta.duration"
      }
    }
  }
}

// 查询翻译资源加载事件
{
  "query": {
    "match": {
      "meta.category": "translation_load"
    }
  },
  "aggs": {
    "sources": {
      "terms": {
        "field": "meta.source.keyword"
      }
    },
    "avg_duration": {
      "avg": {
        "field": "meta.duration"
      }
    }
  }
}
```

3. **日志告警规则**：
```yaml
# elasticsearch-alert.yml
- name: "High language switch failure rate"
  type: "frequency"
  index: "i18n-logs-*"
  num_events: 10
  timeframe:
    minutes: 5
  filter:
  - query:
      query_string:
        query: "meta.category: language_switch AND meta.status: error"
  alert:
  - "email"
  email:
  - "admin@example.com"
```

## 6. 维护策略

### 6.1 日常维护

#### 6.1.1 定期检查

1. **系统健康检查**：
```bash
# 检查服务状态
sudo systemctl status i18n-backend
sudo systemctl status redis-i18n
sudo systemctl status nginx

# 检查磁盘空间
df -h

# 检查内存使用
free -h

# 检查CPU使用
top
```

2. **数据库维护**：
```sql
-- 检查表大小
SELECT
  schemaname,
  tablename,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

-- 检查索引使用情况
SELECT
  schemaname,
  tablename,
  indexname,
  idx_tup_read,
  idx_tup_fetch
FROM pg_stat_user_indexes
ORDER BY idx_tup_read DESC;

-- 更新统计信息
ANALYZE;

-- 清理过期数据
DELETE FROM user_locales WHERE updated_at < NOW() - INTERVAL '1 year';
DELETE FROM translations WHERE updated_at < NOW() - INTERVAL '1 year';
```

3. **缓存维护**：
```bash
# 检查Redis内存使用
redis-cli INFO memory

# 检查Redis键数量
redis-cli DBSIZE

# 清理过期缓存
redis-cli --scan --pattern "i18n:*" | xargs redis-cli DEL

# 检查缓存命中率
redis-cli INFO stats | grep keyspace_hits
redis-cli INFO stats | grep keyspace_misses
```

#### 6.1.2 性能优化

1. **数据库优化**：
```sql
-- 优化索引
CREATE INDEX CONCURRENTLY idx_user_locales_user_id_locale ON user_locales(user_id, locale);
CREATE INDEX CONCURRENTLY idx_translations_key_locale_value ON translations(key, locale, value);

-- 优化查询
EXPLAIN ANALYZE SELECT * FROM user_locales WHERE user_id = 12345;
EXPLAIN ANALYZE SELECT * FROM translations WHERE locale = 'en' AND key LIKE 'user.%';

-- 优化表结构
ALTER TABLE translations ALTER COLUMN value TYPE TEXT;
```

2. **缓存优化**：
```bash
# 优化Redis配置
cat > /etc/redis/redis-i18n-optimized.conf << EOF
port 6379
bind 127.0.0.1
maxmemory 1gb
maxmemory-policy allkeys-lru
save 900 1
save 300 10
save 60 10000
timeout 300
tcp-keepalive 60
EOF

# 重启Redis服务
sudo systemctl restart redis-i18n
```

3. **应用优化**：
```javascript
// 优化国际化管理器
class OptimizedI18nManager {
  constructor() {
    this.cache = new Map();
    this.maxCacheSize = 100;
    this.cacheTTL = 3600; // 1小时
  }
  
  // 优化翻译获取
  getTranslation(key, locale) {
    const cacheKey = `${locale}:${key}`;
    const cached = this.cache.get(cacheKey);
    
    if (cached && Date.now() - cached.timestamp < this.cacheTTL * 1000) {
      return cached.value;
    }
    
    const value = this.loadTranslation(key, locale);
    
    // LRU缓存策略
    if (this.cache.size >= this.maxCacheSize) {
      const oldestKey = this.cache.keys().next().value;
      this.cache.delete(oldestKey);
    }
    
    this.cache.set(cacheKey, {
      value,
      timestamp: Date.now()
    });
    
    return value;
  }
  
  // 批量加载翻译
  async loadTranslations(keys, locale) {
    // 批量查询数据库
    const translations = await this.db.query(
      'SELECT key, value FROM translations WHERE locale = ? AND key IN (?)',
      [locale, keys]
    );
    
    // 批量更新缓存
    const now = Date.now();
    translations.forEach(({ key, value }) => {
      const cacheKey = `${locale}:${key}`;
      this.cache.set(cacheKey, {
        value,
        timestamp: now
      });
    });
    
    return translations;
  }
}
```

### 6.2 更新和升级

#### 6.2.1 翻译资源更新

1. **翻译资源更新流程**：
```bash
# 1. 导出当前翻译资源
node scripts/export-translations.js --output ./translations-backup/

# 2. 更新翻译资源
node scripts/update-translations.js --source ./new-translations/

# 3. 验证翻译资源
node scripts/validate-translations.js --source ./translations/

# 4. 部署翻译资源
node scripts/deploy-translations.js --source ./translations/

# 5. 清理缓存
redis-cli --scan --pattern "i18n:*" | xargs redis-cli DEL
```

2. **翻译资源更新脚本**：
```javascript
// scripts/update-translations.js
const fs = require('fs');
const path = require('path');
const { TranslationService } = require('../services/translation');

async function updateTranslations(sourceDir) {
  const translationService = new TranslationService();
  const locales = ['en', 'zh-CN', 'zh-TW'];
  
  for (const locale of locales) {
    const translationFile = path.join(sourceDir, `${locale}.json`);
    
    if (fs.existsSync(translationFile)) {
      const translations = JSON.parse(fs.readFileSync(translationFile, 'utf8'));
      
      // 更新数据库中的翻译
      for (const [key, value] of Object.entries(flattenObject(translations))) {
        await translationService.updateTranslation(key, locale, value);
      }
      
      console.log(`Updated translations for locale: ${locale}`);
    }
  }
  
  // 清理缓存
  await translationService.clearCache();
  
  console.log('Translation update completed');
}

function flattenObject(obj, prefix = '') {
  const flattened = {};
  
  for (const key in obj) {
    if (typeof obj[key] === 'object' && obj[key] !== null) {
      Object.assign(flattened, flattenObject(obj[key], `${prefix}${key}.`));
    } else {
      flattened[`${prefix}${key}`] = obj[key];
    }
  }
  
  return flattened;
}

updateTranslations(process.argv[2]);
```

#### 6.2.2 系统升级

1. **系统升级流程**：
```bash
# 1. 备份数据库
pg_dump -h localhost -U db_user -d user_management > backup.sql

# 2. 备份翻译资源
cp -r /var/www/i18n /var/www/i18n-backup-$(date +%Y%m%d)

# 3. 停止服务
sudo systemctl stop i18n-backend

# 4. 更新代码
git pull origin main
npm install
npm run build

# 5. 运行数据库迁移
python manage.py migrate

# 6. 启动服务
sudo systemctl start i18n-backend

# 7. 验证服务
curl https://example.com/api/health
```

2. **零停机升级策略**：
```bash
# 1. 使用蓝绿部署
# 部署新版本到备用环境
deploy-to-blue-environment.sh

# 2. 验证新版本
test-blue-environment.sh

# 3. 切换流量
switch-traffic-to-blue.sh

# 4. 监控新版本
monitor-blue-environment.sh

# 5. 回滚策略（如果需要）
rollback-to-green-environment.sh
```

### 6.3 灾难恢复

#### 6.3.1 备份策略

1. **数据库备份**：
```bash
# 每日全量备份
0 2 * * * pg_dump -h localhost -U db_user -d user_management | gzip > /backup/db/daily-$(date +\%Y\%m\%d).sql.gz

# 每小时增量备份
0 * * * * pg_dump -h localhost -U db_user -d user_management --format=directory --file=/backup/db/hourly-$(date +\%Y\%m\%d-\%H)

# 保留7天的备份
0 3 * * * find /backup/db -name "*.sql.gz" -mtime +7 -delete
0 3 * * * find /backup/db -name "hourly-*" -mtime +7 -delete
```

2. **翻译资源备份**：
```bash
# 每日备份翻译资源
0 1 * * * cp -r /var/www/i18n /backup/i18n/daily-$(date +\%Y\%m\%d)

# 保留7天的备份
0 2 * * * find /backup/i18n -name "daily-*" -mtime +7 -exec rm -rf {} \;
```

3. **配置文件备份**：
```bash
# 每日备份配置文件
0 1 * * * tar -czf /backup/config/config-$(date +\%Y\%m\%d).tar.gz /etc/nginx/sites-available/ /var/www/config/

# 保留7天的备份
0 2 * * * find /backup/config -name "config-*.tar.gz" -mtime +7 -delete
```

#### 6.3.2 恢复策略

1. **数据库恢复**：
```bash
# 恢复全量备份
gunzip -c /backup/db/daily-20230720.sql.gz | psql -h localhost -U db_user -d user_management

# 恢复增量备份
pg_restore -h localhost -U db_user -d user_management /backup/db/hourly-20230720-14
```

2. **翻译资源恢复**：
```bash
# 恢复翻译资源
cp -r /backup/i18n/daily-20230720 /var/www/i18n

# 设置权限
chown -R www-data:www-data /var/www/i18n
chmod -R 755 /var/www/i18n

# 清理缓存
redis-cli --scan --pattern "i18n:*" | xargs redis-cli DEL
```

3. **配置文件恢复**：
```bash
# 恢复配置文件
tar -xzf /backup/config/config-20230720.tar.gz -C /

# 重启服务
sudo systemctl restart nginx
sudo systemctl restart i18n-backend
```

## 7. 故障处理

### 7.1 常见问题诊断

#### 7.1.1 语言切换问题

1. **问题现象**：用户无法切换语言或语言切换后界面未更新

2. **诊断步骤**：
```bash
# 检查后端服务状态
sudo systemctl status i18n-backend

# 检查后端日志
sudo tail -f /var/log/app/i18n.log | grep -i "language switch"

# 检查前端控制台错误
# 在浏览器开发者工具中查看Console和Network标签

# 检查数据库连接
psql -h localhost -U db_user -d user_management -c "SELECT * FROM user_locales WHERE user_id = 12345;"

# 检查缓存状态
redis-cli GET "i18n:user:12345:locale"
```

3. **解决方案**：
```javascript
// 检查国际化管理器状态
if (!i18nManager.isInitialized) {
  await i18nManager.init();
}

// 强制刷新翻译资源
await i18nManager.refreshTranslations();

// 检查用户语言设置
const userLocale = await i18nManager.getUserLocale();
if (!userLocale) {
  await i18nManager.setUserLocale(i18nManager.defaultLocale);
}

// 触发界面更新
i18nManager.notifyLocaleChange();
```

#### 7.1.2 翻译资源加载问题

1. **问题现象**：翻译资源加载失败或加载缓慢

2. **诊断步骤**：
```bash
# 检查翻译资源文件
ls -la /var/www/i18n/locales/

# 检查翻译资源文件权限
ls -la /var/www/i18n/locales/en.json

# 检查Nginx配置
sudo nginx -t
sudo tail -f /var/log/nginx/error.log

# 检查后端翻译资源API
curl https://example.com/api/i18n/locales/en

# 检查数据库翻译表
psql -h localhost -U db_user -d user_management -c "SELECT COUNT(*) FROM translations WHERE locale = 'en';"
```

3. **解决方案**：
```javascript
// 检查翻译资源文件是否存在
async function checkTranslationFiles() {
  const locales = ['en', 'zh-CN', 'zh-TW'];
  
  for (const locale of locales) {
    const filePath = path.join(i18nConfig.translations.localPath, `${locale}.json`);
    
    if (!fs.existsSync(filePath)) {
      console.error(`Translation file not found: ${filePath}`);
      // 尝试从远程下载
      await downloadTranslationFile(locale);
    }
  }
}

// 下载翻译资源文件
async function downloadTranslationFile(locale) {
  try {
    const response = await axios.get(`${i18nConfig.translations.remoteApi.baseUrl}/locales/${locale}`);
    const filePath = path.join(i18nConfig.translations.localPath, `${locale}.json`);
    
    fs.writeFileSync(filePath, JSON.stringify(response.data, null, 2));
    console.log(`Downloaded translation file: ${filePath}`);
  } catch (error) {
    console.error(`Failed to download translation file for locale: ${locale}`, error);
  }
}

// 预加载翻译资源
async function preloadTranslations() {
  const locales = ['en', 'zh-CN', 'zh-TW'];
  
  for (const locale of locales) {
    await i18nManager.loadTranslations(locale);
  }
}
```

#### 7.1.3 缓存问题

1. **问题现象**：缓存未命中或缓存数据不一致

2. **诊断步骤**：
```bash
# 检查Redis服务状态
sudo systemctl status redis-i18n

# 检查Redis内存使用
redis-cli INFO memory

# 检查Redis键数量
redis-cli DBSIZE

# 检查特定键
redis-cli GET "i18n:translations:en"

# 检查缓存命中率
redis-cli INFO stats | grep keyspace
```

3. **解决方案**：
```javascript
// 清理缓存
async function clearCache() {
  try {
    await redisClient.del('i18n:translations:*');
    console.log('Cache cleared successfully');
  } catch (error) {
    console.error('Failed to clear cache', error);
  }
}

// 预热缓存
async function warmUpCache() {
  const locales = ['en', 'zh-CN', 'zh-TW'];
  
  for (const locale of locales) {
    const translations = await translationService.getTranslations(locale);
    await redisClient.setex(`i18n:translations:${locale}`, 3600, JSON.stringify(translations));
    console.log(`Cache warmed for locale: ${locale}`);
  }
}

// 优化缓存策略
function optimizeCacheStrategy() {
  // 使用LRU缓存策略
  const cache = new LRUCache({
    max: 100,
    maxAge: 1000 * 60 * 60 // 1小时
  });
  
  // 使用多级缓存
  const memoryCache = new Map();
  const redisCache = redisClient;
  
  return {
    async get(key) {
      // 先从内存缓存获取
      const memoryValue = memoryCache.get(key);
      if (memoryValue) {
        return memoryValue;
      }
      
      // 再从Redis缓存获取
      const redisValue = await redisCache.get(key);
      if (redisValue) {
        memoryCache.set(key, JSON.parse(redisValue));
        return JSON.parse(redisValue);
      }
      
      return null;
    },
    
    async set(key, value, ttl) {
      // 设置内存缓存
      memoryCache.set(key, value);
      
      // 设置Redis缓存
      await redisCache.setex(key, ttl, JSON.stringify(value));
    },
    
    async del(key) {
      // 删除内存缓存
      memoryCache.delete(key);
      
      // 删除Redis缓存
      await redisCache.del(key);
    }
  };
}
```

### 7.2 故障恢复流程

#### 7.2.1 服务故障恢复

1. **故障检测**：
```bash
# 检查服务状态
sudo systemctl status i18n-backend

# 检查端口监听
netstat -tlnp | grep :3000

# 检查进程
ps aux | grep i18n-backend

# 检查日志
sudo tail -f /var/log/app/i18n.log
```

2. **故障恢复**：
```bash
# 重启服务
sudo systemctl restart i18n-backend

# 如果重启失败，检查配置
sudo nginx -t

# 如果配置有问题，恢复备份配置
sudo cp /etc/nginx/sites-available/frontend.backup /etc/nginx/sites-available/frontend

# 再次重启服务
sudo systemctl restart nginx
sudo systemctl restart i18n-backend

# 验证服务
curl https://example.com/api/health
```

3. **故障通知**：
```javascript
// 发送故障通知
async function sendAlert(message) {
  try {
    await axios.post('https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK', {
      text: `🚨 国际化服务故障: ${message}`,
      channel: '#alerts',
      username: 'i18n-bot'
    });
    
    // 发送邮件通知
    await transporter.sendMail({
      from: 'alerts@example.com',
      to: 'admin@example.com',
      subject: '国际化服务故障',
      text: message
    });
  } catch (error) {
    console.error('Failed to send alert', error);
  }
}

// 监控服务状态
async function monitorService() {
  try {
    const response = await axios.get('https://example.com/api/health', {
      timeout: 5000
    });
    
    if (response.status !== 200) {
      await sendAlert(`服务返回异常状态码: ${response.status}`);
    }
  } catch (error) {
    await sendAlert(`服务无法访问: ${error.message}`);
  }
}

// 定时监控
setInterval(monitorService, 60000); // 每分钟检查一次
```

#### 7.2.2 数据故障恢复

1. **数据一致性检查**：
```sql
-- 检查用户语言设置一致性
SELECT u.id, u.username, ul.locale
FROM users u
LEFT JOIN user_locales ul ON u.id = ul.user_id
WHERE ul.locale IS NULL;

-- 检查翻译资源完整性
SELECT locale, COUNT(*) as translation_count
FROM translations
GROUP BY locale
ORDER BY translation_count;

-- 检查重复翻译
SELECT key, locale, COUNT(*) as duplicate_count
FROM translations
GROUP BY key, locale
HAVING COUNT(*) > 1;
```

2. **数据修复**：
```sql
-- 修复用户语言设置
INSERT INTO user_locales (user_id, locale, is_default)
SELECT u.id, 'zh-CN', true
FROM users u
LEFT JOIN user_locales ul ON u.id = ul.user_id
WHERE ul.locale IS NULL;

-- 删除重复翻译
DELETE FROM translations
WHERE id IN (
  SELECT id FROM (
    SELECT id, ROW_NUMBER() OVER (
      PARTITION BY key, locale ORDER BY updated_at DESC
    ) as row_num
    FROM translations
  ) t
  WHERE t.row_num > 1
);
```

3. **数据恢复**：
```bash
# 从备份恢复数据
psql -h localhost -U db_user -d user_management < /backup/db/daily-20230720.sql

# 验证数据恢复
psql -h localhost -U db_user -d user_management -c "SELECT COUNT(*) FROM user_locales;"
psql -h localhost -U db_user -d user_management -c "SELECT COUNT(*) FROM translations;"

# 重启服务
sudo systemctl restart i18n-backend
```

## 8. 性能优化

### 8.1 前端性能优化

#### 8.1.1 翻译资源加载优化

1. **按需加载翻译资源**：
```javascript
// 按需加载翻译资源
class LazyI18nLoader {
  constructor() {
    this.loadedLocales = new Set();
    this.loadingPromises = new Map();
  }
  
  async loadLocale(locale) {
    if (this.loadedLocales.has(locale)) {
      return;
    }
    
    if (this.loadingPromises.has(locale)) {
      return this.loadingPromises.get(locale);
    }
    
    const loadPromise = this.doLoadLocale(locale);
    this.loadingPromises.set(locale, loadPromise);
    
    try {
      await loadPromise;
      this.loadedLocales.add(locale);
    } finally {
      this.loadingPromises.delete(locale);
    }
  }
  
  async doLoadLocale(locale) {
    const response = await fetch(`/i18n/locales/${locale}.json`);
    const translations = await response.json();
    
    // 合并翻译资源
    Object.assign(i18n.translations, translations);
    
    // 触发更新
    i18n.forceUpdate();
  }
}
```

2. **翻译资源预加载**：
```javascript
// 预加载翻译资源
async function preloadLocales() {
  const userLocale = await getUserLocale();
  const otherLocales = ['en', 'zh-CN', 'zh-TW'].filter(locale => locale !== userLocale);
  
  // 预加载用户当前语言
  await i18nLoader.loadLocale(userLocale);
  
  // 延迟预加载其他语言
  setTimeout(async () => {
    for (const locale of otherLocales) {
      i18nLoader.loadLocale(locale);
    }
  }, 2000);
}
```

3. **翻译资源压缩**：
```javascript
// 压缩翻译资源
function compressTranslations(translations) {
  const compressed = {};
  
  // 提取公共前缀
  function extractCommonPrefix(obj) {
    const keys = Object.keys(obj);
    if (keys.length === 0) return {};
    
    let prefix = keys[0];
    for (let i = 1; i < keys.length; i++) {
      let j = 0;
      while (j < prefix.length && j < keys[i].length && prefix[j] === keys[i][j]) {
        j++;
      }
      prefix = prefix.substring(0, j);
    }
    
    if (prefix.length < 2) return {};
    
    const result = {
      [`__prefix_${prefix}`]: {}
    };
    
    for (const key in obj) {
      const newKey = key.substring(prefix.length);
      result[`__prefix_${prefix}`][newKey] = obj[key];
    }
    
    return result;
  }
  
  // 递归压缩
  function compress(obj) {
    const result = {};
    
    for (const key in obj) {
      if (typeof obj[key] === 'object') {
        const compressed = extractCommonPrefix(obj[key]);
        if (Object.keys(compressed).length > 0) {
          Object.assign(result, compressed);
        } else {
          result[key] = compress(obj[key]);
        }
      } else {
        result[key] = obj[key];
      }
    }
    
    return result;
  }
  
  return compress(translations);
}
```

#### 8.1.2 语言切换性能优化

1. **语言切换优化**：
```javascript
// 优化语言切换
async function optimizedLanguageSwitch(newLocale) {
  // 显示加载状态
  showLoadingIndicator();
  
  try {
    // 并行加载翻译资源和更新用户设置
    const [translations, userSettings] = await Promise.all([
      i18nLoader.loadLocale(newLocale),
      updateUserLocale(newLocale)
    ]);
    
    // 更新应用语言
    i18n.locale = newLocale;
    
    // 触发界面更新
    i18n.forceUpdate();
    
    // 预加载其他语言
    preloadOtherLocales(newLocale);
    
    return true;
  } catch (error) {
    console.error('Language switch failed', error);
    showErrorNotification('Language switch failed');
    return false;
  } finally {
    hideLoadingIndicator();
  }
}
```

2. **界面更新优化**：
```javascript
// 使用虚拟DOM优化界面更新
function optimizedForceUpdate() {
  // 只更新需要更新的组件
  const components = getI18nComponents();
  
  components.forEach(component => {
    if (component.shouldUpdate()) {
      component.update();
    }
  });
}

// 使用requestAnimationFrame优化动画
function animateLanguageSwitch(callback) {
  const startTime = performance.now();
  const duration = 300; // 动画持续时间
  
  function animate(currentTime) {
    const elapsed = currentTime - startTime;
    const progress = Math.min(elapsed / duration, 1);
    
    // 更新动画进度
    updateAnimationProgress(progress);
    
    if (progress < 1) {
      requestAnimationFrame(animate);
    } else {
      callback();
    }
  }
  
  requestAnimationFrame(animate);
}
```

### 8.2 后端性能优化

#### 8.2.1 数据库查询优化

1. **查询优化**：
```sql
-- 优化用户语言设置查询
CREATE INDEX CONCURRENTLY idx_user_locales_user_id_locale ON user_locales(user_id, locale);

-- 优化翻译资源查询
CREATE INDEX CONCURRENTLY idx_translations_key_locale ON translations(key, locale);

-- 使用物化视图优化复杂查询
CREATE MATERIALIZED VIEW mv_translation_summary AS
SELECT locale, COUNT(*) as translation_count
FROM translations
GROUP BY locale;

-- 定期刷新物化视图
REFRESH MATERIALIZED VIEW mv_translation_summary;
```

2. **批量操作优化**：
```javascript
// 批量获取翻译资源
async function batchGetTranslations(keys, locale) {
  // 使用IN查询批量获取
  const translations = await db.query(
    'SELECT key, value FROM translations WHERE locale = ? AND key IN (?)',
    [locale, keys]
  );
  
  // 转换为Map便于查找
  const translationMap = new Map();
  translations.forEach(({ key, value }) => {
    translationMap.set(key, value);
  });
  
  return translationMap;
}

// 批量更新翻译资源
async function batchUpdateTranslations(updates) {
  // 使用事务确保数据一致性
  const transaction = await db.beginTransaction();
  
  try {
    for (const { key, locale, value } of updates) {
      await transaction.query(
        'INSERT INTO translations (key, locale, value) VALUES (?, ?, ?) ' +
        'ON CONFLICT (key, locale) DO UPDATE SET value = ?, updated_at = CURRENT_TIMESTAMP',
        [key, locale, value, value]
      );
    }
    
    await transaction.commit();
  } catch (error) {
    await transaction.rollback();
    throw error;
  }
}
```

#### 8.2.2 缓存优化

1. **多级缓存策略**：
```javascript
// 多级缓存实现
class MultiLevelCache {
  constructor() {
    this.l1Cache = new Map(); // 内存缓存
    this.l2Cache = redisClient; // Redis缓存
    this.l1TTL = 60000; // 1分钟
    this.l2TTL = 3600; // 1小时
  }
  
  async get(key) {
    // 先从L1缓存获取
    const l1Value = this.l1Cache.get(key);
    if (l1Value && Date.now() - l1Value.timestamp < this.l1TTL) {
      return l1Value.data;
    }
    
    // 再从L2缓存获取
    const l2Value = await this.l2Cache.get(key);
    if (l2Value) {
      const parsedValue = JSON.parse(l2Value);
      
      // 回填L1缓存
      this.l1Cache.set(key, {
        data: parsedValue,
        timestamp: Date.now()
      });
      
      return parsedValue;
    }
    
    return null;
  }
  
  async set(key, value) {
    // 设置L1缓存
    this.l1Cache.set(key, {
      data: value,
      timestamp: Date.now()
    });
    
    // 设置L2缓存
    await this.l2Cache.setex(key, this.l2TTL, JSON.stringify(value));
  }
  
  async del(key) {
    // 删除L1缓存
    this.l1Cache.delete(key);
    
    // 删除L2缓存
    await this.l2Cache.del(key);
  }
  
  async clear() {
    // 清除L1缓存
    this.l1Cache.clear();
    
    // 清除L2缓存
    const keys = await this.l2Cache.keys('i18n:*');
    if (keys.length > 0) {
      await this.l2Cache.del(...keys);
    }
  }
}
```

2. **缓存预热策略**：
```javascript
// 缓存预热
async function warmUpCache() {
  const locales = ['en', 'zh-CN', 'zh-TW'];
  const cache = new MultiLevelCache();
  
  for (const locale of locales) {
    // 获取热门翻译键
    const hotKeys = await getHotTranslationKeys(locale);
    
    // 批量获取翻译
    const translations = await batchGetTranslations(hotKeys, locale);
    
    // 批量设置缓存
    for (const [key, value] of translations.entries()) {
      await cache.set(`i18n:${locale}:${key}`, value);
    }
    
    console.log(`Cache warmed for locale: ${locale}`);
  }
}

// 获取热门翻译键
async function getHotTranslationKeys(locale) {
  // 从访问日志中分析热门翻译键
  const hotKeys = await db.query(
    'SELECT key, COUNT(*) as count FROM translation_access_log ' +
    'WHERE locale = ? AND accessed_at > NOW() - INTERVAL 7 DAY ' +
    'GROUP BY key ORDER BY count DESC LIMIT 100',
    [locale]
  );
  
  return hotKeys.map(row => row.key);
}
```

### 8.3 网络性能优化

#### 8.3.1 CDN优化

1. **CDN配置**：
```nginx
# Nginx配置CDN缓存
location /i18n/ {
    # 设置CDN缓存头
    add_header Cache-Control "public, max-age=3600";
    
    # 启用Brotli压缩
    brotli on;
    brotli_comp_level 6;
    brotli_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
    
    # 启用Gzip压缩
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_comp_level 6;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
    
    # 设置ETag
    etag on;
    
    # 启用Expires
    expires 1h;
}
```

2. **CDN缓存刷新**：
```javascript
// CDN缓存刷新
async function refreshCDNCache(urls) {
  const cdnProvider = 'cloudflare'; // 或其他CDN提供商
  
  switch (cdnProvider) {
    case 'cloudflare':
      await refreshCloudflareCache(urls);
      break;
    case 'akamai':
      await refreshAkamaiCache(urls);
      break;
    default:
      console.error('Unsupported CDN provider');
  }
}

// Cloudflare缓存刷新
async function refreshCloudflareCache(urls) {
  const response = await fetch('https://api.cloudflare.com/client/v4/zones/YOUR_ZONE_ID/purge_cache', {
    method: 'POST',
    headers: {
      'Authorization': 'Bearer YOUR_API_TOKEN',
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      files: urls
    })
  });
  
  const result = await response.json();
  
  if (result.success) {
    console.log('CDN cache refreshed successfully');
  } else {
    console.error('Failed to refresh CDN cache', result.errors);
  }
}
```

#### 8.3.2 API优化

1. **API响应优化**：
```javascript
// 启用HTTP/2
const http2 = require('http2');

// 创建HTTP/2服务器
const server = http2.createSecureServer({
  key: fs.readFileSync('server.key'),
  cert: fs.readFileSync('server.crt')
}, app);

// 启用响应压缩
app.use(compression({
  level: 6,
  threshold: 1024,
  filter: (req, res) => {
    if (req.headers['x-no-compression']) {
      return false;
    }
    return compression.filter(req, res);
  }
}));

// 启用ETag
app.set('etag', 'strong');

// 启用条件请求
app.get('/api/i18n/locales/:locale', cache('1 hour'), async (req, res) => {
  const { locale } = req.params;
  
  // 检查If-None-Match头
  if (req.fresh) {
    return res.status(304).end();
  }
  
  const translations = await translationService.getTranslations(locale);
  
  res.set('Cache-Control', 'public, max-age=3600');
  res.json(translations);
});
```

2. **API批量请求优化**：
```javascript
// 批量请求API
app.post('/api/i18n/locales/batch', async (req, res) => {
  const { locales } = req.body;
  
  if (!Array.isArray(locales) || locales.length === 0) {
    return res.status(400).json({
      code: 400,
      message: 'Invalid locales parameter'
    });
  }
  
  // 并行获取所有语言的翻译资源
  const translations = await Promise.all(
    locales.map(locale => translationService.getTranslations(locale))
  );
  
  // 转换为对象
  const result = {};
  locales.forEach((locale, index) => {
    result[locale] = translations[index];
  });
  
  res.json(result);
});
```

## 9. 总结

### 9.1 部署和维护要点

本文档详细介绍了国际化功能的部署和维护方案，包括以下关键要点：

1. **部署环境要求**：
   - 硬件环境：应用服务器、数据库服务器、缓存服务器的配置要求
   - 软件环境：操作系统、中间件、开发和部署工具的版本要求
   - 网络环境：网络拓扑、端口要求、带宽要求

2. **部署步骤**：
   - 部署前准备：环境检查、数据准备、配置文件准备
   - 部署流程：前端部署、后端部署、缓存部署
   - 部署后验证：功能验证、性能验证

3. **配置管理**：
   - 配置文件管理：配置文件结构、配置文件示例
   - 环境变量管理：环境变量列表、环境变量管理最佳实践
   - 配置版本控制：配置版本控制策略、配置变更管理

4. **监控和日志**：
   - 监控策略：监控指标、监控工具配置
   - 日志管理：日志策略、日志配置、日志收集和分析

5. **维护策略**：
   - 日常维护：定期检查、性能优化
   - 更新和升级：翻译资源更新、系统升级
   - 灾难恢复：备份策略、恢复策略

6. **故障处理**：
   - 常见问题诊断：语言切换问题、翻译资源加载问题、缓存问题
   - 故障恢复流程：服务故障恢复、数据故障恢复

7. **性能优化**：
   - 前端性能优化：翻译资源加载优化、语言切换性能优化
   - 后端性能优化：数据库查询优化、缓存优化
   - 网络性能优化：CDN优化、API优化

### 9.2 最佳实践建议

基于国际化功能的部署和维护经验，提出以下最佳实践建议：

1. **部署最佳实践**：
   - 使用蓝绿部署或金丝雀发布策略，减少部署风险
   - 自动化部署流程，减少人为错误
   - 部署前进行全面测试，确保功能正常
   - 部署后进行性能监控，及时发现性能问题

2. **配置管理最佳实践**：
   - 使用环境变量管理敏感配置
   - 配置文件模板化，便于不同环境部署
   - 配置变更进行版本控制和审查
   - 定期审计配置，确保配置安全

3. **监控和日志最佳实践**：
   - 建立全面的监控体系，覆盖系统、应用和业务指标
   - 设置合理的告警阈值，及时发现异常
   - 日志结构化，便于分析和查询
   - 建立日志归档策略，控制存储成本

4. **维护策略最佳实践**：
   - 定期进行系统健康检查，预防问题发生
   - 建立完善的备份策略，确保数据安全
   - 制定详细的故障恢复流程，减少故障影响
   - 定期进行性能优化，保持系统高效运行

5. **性能优化最佳实践**：
   - 使用多级缓存策略，提高数据访问速度
   - 优化数据库查询，减少数据库负载
   - 使用CDN加速静态资源访问
   - 定期进行性能测试，发现性能瓶颈

### 9.3 未来发展方向

随着业务的发展和技术的进步，国际化功能的部署和维护也将不断演进，未来可能的发展方向包括：

1. **自动化运维**：
   - 引入AIOps技术，实现智能故障预测和自动修复
   - 自动化扩缩容，根据负载动态调整资源
   - 自动化性能优化，根据监控数据自动调整配置

2. **多云部署**：
   - 支持多云部署，提高系统可用性
   - 跨云负载均衡，优化用户访问体验
   - 跨云数据同步，确保数据一致性

3. **边缘计算**：
   - 将国际化功能部署到边缘节点，减少网络延迟
   - 边缘缓存翻译资源，提高访问速度
   - 边缘计算语言切换，减轻中心服务器负担

4. **微服务架构**：
   - 将国际化功能拆分为独立的微服务
   - 微服务独立部署和扩展
   - 微服务间通过API网关统一管理

通过不断优化部署和维护策略，国际化功能将能够更好地支持业务发展，为用户提供更优质的多语言体验。
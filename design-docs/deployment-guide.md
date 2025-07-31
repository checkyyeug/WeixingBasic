# 部署指南

## 1. 概述

本文档详细描述了微信小程序用户管理系统的部署流程，包括环境要求、部署步骤、配置说明、启动方式等内容。

## 2. 环境要求

### 2.1 服务器要求
- 操作系统：Linux (推荐Ubuntu 20.04 LTS或CentOS 8) 或 Windows Server 2019
- CPU：至少2核
- 内存：至少4GB
- 磁盘空间：至少20GB可用空间
- 网络：公网IP地址，支持HTTPS

### 2.2 软件依赖
- Python >= 3.9
- PostgreSQL >= 12.0
- Redis >= 6.0 (可选，用于缓存)
- Nginx >= 1.18 (用于反向代理和静态资源服务)

### 2.3 微信小程序要求
- 微信开发者账号
- 已注册的小程序AppID
- 已备案的域名

## 3. 部署前准备

### 3.1 服务器初始化
1. 更新系统软件包
```bash
# Ubuntu/Debian
sudo apt update && sudo apt upgrade -y

# CentOS/RHEL
sudo yum update -y
```

2. 安装基础软件
```bash
# Ubuntu/Debian
sudo apt install -y git curl wget vim nginx

# CentOS/RHEL
sudo yum install -y git curl wget vim nginx
```

### 3.2 安装Python
```bash
# Ubuntu/Debian
sudo apt install -y python3 python3-pip python3-venv

# 验证安装
python3 --version
pip3 --version

# CentOS/RHEL
sudo yum install -y python3 python3-pip
```

### 3.3 安装PostgreSQL
```bash
# Ubuntu/Debian
sudo apt install -y postgresql postgresql-contrib

# 启动PostgreSQL服务
sudo systemctl start postgresql
sudo systemctl enable postgresql

# CentOS/RHEL
sudo yum install -y postgresql-server postgresql-contrib
sudo postgresql-setup initdb
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

### 3.4 安装Redis (可选)
```bash
# Ubuntu/Debian
sudo apt install -y redis-server

# 启动Redis服务
sudo systemctl start redis
sudo systemctl enable redis

# CentOS/RHEL
sudo yum install -y redis
sudo systemctl start redis
sudo systemctl enable redis
```

## 4. 获取源代码

### 4.1 克隆代码仓库
```bash
# 创建项目目录
sudo mkdir -p /var/www/user-management-system
sudo chown $USER:$USER /var/www/user-management-system

# 克隆代码
git clone https://github.com/your-org/user-management-system.git /var/www/user-management-system

# 进入项目目录
cd /var/www/user-management-system
```

### 4.2 创建虚拟环境并安装依赖
```bash
# 创建虚拟环境
python3 -m venv venv

# 激活虚拟环境
source venv/bin/activate

# 升级pip
pip install --upgrade pip

# 安装Python依赖
pip install -r requirements.txt
```

## 5. 数据库配置

### 5.1 创建数据库
```sql
-- 切换到postgres用户
sudo -u postgres psql

-- 创建数据库和用户
CREATE DATABASE user_management_system WITH ENCODING='UTF8';
CREATE USER ums_user WITH PASSWORD 'your_strong_password';
GRANT ALL PRIVILEGES ON DATABASE user_management_system TO ums_user;
\q
```

### 5.2 运行数据库迁移
```bash
# 激活虚拟环境
source venv/bin/activate

# 运行数据库迁移
alembic upgrade head
```

### 5.3 导入初始数据
```bash
# 激活虚拟环境
source venv/bin/activate

# 运行初始数据脚本
psql -U ums_user -d user_management_system -f design-docs/database/initial-data.sql
```

## 6. 配置文件设置

### 6.1 应用配置
创建配置文件 `.env`：
```bash
# 应用配置
APP_ENV=production
APP_DEBUG=false
APP_HOST=0.0.0.0
APP_PORT=8000

# 数据库配置
DATABASE_URL=postgresql://ums_user:your_strong_password@localhost:5432/user_management_system

# Redis配置
REDIS_URL=redis://localhost:6379/0

# JWT配置
JWT_SECRET_KEY=your_jwt_secret_key
JWT_ALGORITHM=HS256
JWT_ACCESS_TOKEN_EXPIRE_MINUTES=1440
JWT_REFRESH_TOKEN_EXPIRE_MINUTES=10080

# 微信配置
WECHAT_APP_ID=your_wechat_app_id
WECHAT_APP_SECRET=your_wechat_app_secret

# CORS配置
CORS_ORIGINS=["https://your-miniprogram-domain.com"]
```

### 6.2 Gunicorn配置
创建 `gunicorn.conf.py`：
```python
# Gunicorn配置文件
bind = "0.0.0.0:8000"
workers = 4
worker_class = "uvicorn.workers.UvicornWorker"
worker_connections = 1000
max_requests = 1000
max_requests_jitter = 100
timeout = 30
keepalive = 2
preload_app = True
```

## 7. Nginx配置

### 7.1 创建Nginx配置文件
创建 `/etc/nginx/sites-available/user-management-system`：
```nginx
server {
    listen 80;
    server_name your-api-domain.com;
    
    # 重定向到HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-api-domain.com;
    
    # SSL证书配置
    ssl_certificate /path/to/your/certificate.crt;
    ssl_certificate_key /path/to/your/private.key;
    
    # SSL安全配置
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;
    
    # 安全头
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;
    
    # API代理
    location /api/ {
        proxy_pass http://localhost:8000/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
    
    # 静态资源
    location / {
        root /var/www/user-management-system/public;
        try_files $uri $uri/ =404;
    }
    
    # 日志配置
    access_log /var/log/nginx/user-management-system.access.log;
    error_log /var/log/nginx/user-management-system.error.log;
}
```

### 7.2 启用Nginx配置
```bash
# 创建软链接
sudo ln -s /etc/nginx/sites-available/user-management-system /etc/nginx/sites-enabled/

# 测试配置
sudo nginx -t

# 重新加载Nginx
sudo systemctl reload nginx
```

## 8. SSL证书配置

### 8.1 使用Let's Encrypt获取免费SSL证书
```bash
# 安装Certbot
sudo apt install -y certbot python3-certbot-nginx

# 获取证书
sudo certbot --nginx -d your-api-domain.com

# 自动续期
sudo crontab -e
# 添加以下行：
# 0 12 * * * /usr/bin/certbot renew --quiet
```

## 9. 启动应用

### 9.1 使用Systemd管理应用
创建 `/etc/systemd/system/user-management-system.service`：
```ini
[Unit]
Description=User Management System
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/var/www/user-management-system
Environment=PATH=/var/www/user-management-system/venv/bin
ExecStart=/var/www/user-management-system/venv/bin/gunicorn -c gunicorn.conf.py app.main:app
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

```bash
# 重新加载systemd配置
sudo systemctl daemon-reload

# 启动应用
sudo systemctl start user-management-system

# 设置开机自启
sudo systemctl enable user-management-system

# 查看应用状态
sudo systemctl status user-management-system
```

### 9.2 使用Supervisor管理应用（可选）
```bash
# 安装Supervisor
sudo apt install -y supervisor

# 创建配置文件 /etc/supervisor/conf.d/user-management-system.conf
[program:user-management-system]
command=/var/www/user-management-system/venv/bin/gunicorn -c /var/www/user-management-system/gunicorn.conf.py app.main:app
directory=/var/www/user-management-system
user=www-data
autostart=true
autorestart=true
redirect_stderr=true
stdout_logfile=/var/log/user-management-system.log

# 重新加载Supervisor配置
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start user-management-system
```

## 10. 微信小程序配置

### 10.1 配置服务器域名
在微信公众平台小程序后台：
1. 登录微信公众平台
2. 进入"开发管理" -> "开发设置"
3. 在"服务器域名"中添加：
   - request合法域名：https://your-api-domain.com
   - socket合法域名：wss://your-api-domain.com
   - uploadFile合法域名：https://your-api-domain.com
   - downloadFile合法域名：https://your-api-domain.com

### 10.2 配置业务域名
在"开发管理" -> "开发设置"中配置业务域名：
- https://your-miniprogram-domain.com

## 11. 监控和日志

### 11.1 应用日志
```bash
# 查看Systemd日志
sudo journalctl -u user-management-system -f

# 查看Supervisor日志
sudo tail -f /var/log/user-management-system.log

# 查看Nginx日志
sudo tail -f /var/log/nginx/user-management-system.access.log
sudo tail -f /var/log/nginx/user-management-system.error.log
```

### 11.2 系统监控
```bash
# 安装htop用于系统监控
sudo apt install -y htop

# 查看系统资源使用情况
htop
```

## 12. 备份和恢复

### 12.1 数据库备份
```bash
# PostgreSQL备份
pg_dump -U ums_user -d user_management_system > backup_$(date +%Y%m%d_%H%M%S).sql

# PostgreSQL压缩备份
pg_dump -U ums_user -d user_management_system | gzip > backup_$(date +%Y%m%d_%H%M%S).sql.gz
```

### 12.2 应用代码备份
```bash
# 创建代码备份
tar -czf app_backup_$(date +%Y%m%d_%H%M%S).tar.gz /var/www/user-management-system
```

### 12.3 自动备份脚本
创建 `/usr/local/bin/backup.sh`：
```bash
#!/bin/bash

# 数据库备份
pg_dump -U ums_user -d user_management_system > /backup/db_backup_$(date +%Y%m%d_%H%M%S).sql

# 应用代码备份
tar -czf /backup/app_backup_$(date +%Y%m%d_%H%M%S).tar.gz /var/www/user-management-system

# 删除7天前的备份
find /backup -name "*.sql" -mtime +7 -delete
find /backup -name "*.tar.gz" -mtime +7 -delete
```

设置定时任务：
```bash
# 编辑crontab
crontab -e

# 添加每日凌晨2点执行备份
0 2 * * * /usr/local/bin/backup.sh
```

## 13. 故障排除

### 13.1 常见问题
1. **应用无法启动**
   - 检查端口是否被占用：`lsof -i :8000`
   - 检查数据库连接：`psql -U ums_user -d user_management_system`
   - 查看应用日志：`sudo journalctl -u user-management-system`

2. **API接口返回404**
   - 检查Nginx配置：`sudo nginx -t`
   - 检查应用是否正常运行：`sudo systemctl status user-management-system`
   - 查看Nginx错误日志：`sudo tail -f /var/log/nginx/error.log`

3. **数据库连接失败**
   - 检查数据库服务状态：`sudo systemctl status postgresql`
   - 检查数据库配置是否正确
   - 检查防火墙设置

### 13.2 性能优化
1. **数据库优化**
   - 定期分析和优化表
   - 添加适当的索引
   - 配置查询缓存

2. **应用优化**
   - 使用Redis缓存频繁访问的数据
   - 启用Gzip压缩
   - 使用CDN加速静态资源

3. **Nginx优化**
   - 调整worker_processes和worker_connections
   - 启用gzip压缩
   - 配置缓存策略

## 14. 升级和维护

### 14.1 应用升级
```bash
# 停止应用
sudo systemctl stop user-management-system

# 备份当前版本
tar -czf backup_$(date +%Y%m%d_%H%M%S).tar.gz /var/www/user-management-system

# 拉取最新代码
cd /var/www/user-management-system
git pull origin main

# 激活虚拟环境
source venv/bin/activate

# 安装新依赖
pip install -r requirements.txt

# 运行数据库迁移
alembic upgrade head

# 启动应用
sudo systemctl start user-management-system
```

### 14.2 数据库迁移
```bash
# 备份数据库
pg_dump -U ums_user -d user_management_system > backup_$(date +%Y%m%d_%H%M%S).sql

# 运行迁移脚本
alembic upgrade head
```

## 15. 安全加固

### 15.1 服务器安全
1. **防火墙配置**
```bash
# Ubuntu/Debian (UFW)
sudo ufw allow ssh
sudo ufw allow 'Nginx Full'
sudo ufw enable

# CentOS/RHEL (firewalld)
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

2. **SSH安全**
```bash
# 编辑SSH配置
sudo vim /etc/ssh/sshd_config

# 修改以下配置：
# Port 2222  # 修改默认端口
# PermitRootLogin no  # 禁止root登录
# PasswordAuthentication no  # 禁止密码登录，使用SSH密钥

# 重启SSH服务
sudo systemctl restart ssh
```

### 15.2 应用安全
1. **环境变量保护**
   - 不要在代码中硬编码敏感信息
   - 使用环境变量存储密钥和密码

2. **输入验证**
   - 对所有用户输入进行验证和过滤
   - 使用参数化查询防止SQL注入

3. **访问控制**
   - 实施严格的权限控制
   - 定期审查用户权限

## 16. 性能监控

### 16.1 使用Prometheus和Grafana监控
1. **安装Prometheus**
```bash
# 下载并安装Prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.30.0/prometheus-2.30.0.linux-amd64.tar.gz
tar -xzf prometheus-2.30.0.linux-amd64.tar.gz
sudo mv prometheus-2.30.0.linux-amd64 /opt/prometheus
```

2. **安装Grafana**
```bash
# Ubuntu/Debian
sudo apt-get install -y apt-transport-https
sudo apt-get install -y software-properties-common wget
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
sudo apt-get update
sudo apt-get install grafana

# 启动Grafana
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

### 16.2 使用第三方监控工具
1. **New Relic**
2. **Datadog**
3. **Sentry (错误追踪)**

## 17. 联系和支持

如有部署问题，请联系技术支持：
- 邮箱：support@your-company.com
- 电话：400-xxx-xxxx
- 在线支持：https://support.your-company.com
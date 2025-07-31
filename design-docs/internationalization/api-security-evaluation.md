# 后端API接口完整性和安全性评估

## 1. 现有API接口分析

### 1.1 API接口概述

根据文档分析，当前的国际化相关API接口主要包括：

1. **用户语言设置API**：
   - `GET /user/locale` - 获取用户语言设置
   - `PUT /user/locale` - 更新用户语言设置

2. **语言资源API**：
   - `GET /i18n/version` - 获取语言资源版本
   - `GET /i18n/locales/{locale}` - 获取指定语言的语言资源

### 1.2 API实现分析

#### 1.2.1 用户语言设置API

```python
# app/api/v1/i18n.py
from fastapi import APIRouter, Depends, HTTPException, Request
from sqlalchemy.orm import Session
from app.core.security import get_current_user
from app.crud.i18n import crud_user_locale
from app.schemas.i18n import UserLocaleCreate, UserLocaleUpdate
from app.models.user import User
from app.core.database import get_db

router = APIRouter()

@router.get("/user/locale")
async def get_user_locale(
    request: Request,
    current_user: User = Depends(get_current_user),
    db: Session = Depends(get_db)
):
    """获取用户语言设置"""
    try:
        user_locale = crud_user_locale.get_by_user_id(db, user_id=current_user.id)
        
        if not user_locale:
            # 如果没有设置，返回默认语言
            return {
                "code": 200,
                "message": "Success",
                "data": {
                    "locale": "zh-CN"
                }
            }
        
        return {
            "code": 200,
            "message": "Success",
            "data": {
                "locale": user_locale.locale
            }
        }
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@router.put("/user/locale")
async def update_user_locale(
    locale_data: UserLocaleUpdate,
    request: Request,
    current_user: User = Depends(get_current_user),
    db: Session = Depends(get_db)
):
    """更新用户语言设置"""
    try:
        # 验证语言代码
        supported_locales = ['en', 'zh-CN', 'zh-TW']
        if locale_data.locale not in supported_locales:
            return {
                "code": 400,
                "message": "Unsupported locale",
                "data": None
            }
        
        # 查找现有设置
        user_locale = crud_user_locale.get_by_user_id(db, user_id=current_user.id)
        
        if user_locale:
            # 更新现有设置
            updated_locale = crud_user_locale.update(
                db, db_obj=user_locale, obj_in=locale_data
            )
        else:
            # 创建新设置
            locale_in = UserLocaleCreate(
                user_id=current_user.id,
                locale=locale_data.locale
            )
            updated_locale = crud_user_locale.create(db, obj_in=locale_in)
        
        return {
            "code": 200,
            "message": "Locale updated successfully",
            "data": {
                "locale": updated_locale.locale
            }
        }
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

#### 1.2.2 语言资源API

```python
@router.get("/i18n/version")
async def get_locale_version():
    """获取语言资源版本"""
    try:
        # 这里可以从数据库或配置文件中获取版本号
        version = "1.0.0"
        
        return {
            "code": 200,
            "message": "Success",
            "data": {
                "version": version
            }
        }
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@router.get("/i18n/locales/{locale}")
async def get_locale_data(
    locale: str,
    request: Request,
    db: Session = Depends(get_db)
):
    """获取指定语言的语言资源"""
    try:
        # 验证语言代码
        supported_locales = ['en', 'zh-CN', 'zh-TW']
        if locale not in supported_locales:
            return {
                "code": 400,
                "message": "Unsupported locale",
                "data": None
            }
        
        # 从数据库获取翻译数据
        translations = crud_user_locale.get_translations_by_locale(db, locale=locale)
        
        # 转换为前端需要的格式
        locale_data = {}
        for translation in translations:
            # 支持嵌套键，如 'user.profile.title'
            keys = translation.key.split('.')
            current = locale_data
            
            for i, key in enumerate(keys):
                if i === len(keys) - 1:
                    current[key] = translation.value
                else:
                    if key not in current:
                        current[key] = {}
                    current = current[key]
        
        return {
            "code": 200,
            "message": "Success",
            "data": locale_data
        }
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

### 1.3 API安全性评估

#### 1.3.1 优点

1. **用户认证**：
   - 使用`get_current_user`依赖进行用户认证
   - 确保只有登录用户才能访问和修改语言设置

2. **输入验证**：
   - 验证语言代码是否在支持的列表中
   - 防止无效语言代码的输入

3. **错误处理**：
   - 使用try-catch捕获异常
   - 返回适当的错误响应

4. **数据访问控制**：
   - 用户只能访问和修改自己的语言设置
   - 通过用户ID进行数据隔离

#### 1.3.2 缺点

1. **API安全机制不完善**：
   - 缺少请求频率限制
   - 没有实现API密钥或令牌验证
   - 缺少请求签名验证

2. **数据保护不足**：
   - 没有对敏感数据进行加密
   - 缺少数据传输加密
   - 没有实现数据访问日志

3. **错误处理过于简单**：
   - 直接暴露异常详情，可能泄露系统信息
   - 没有统一的错误码和错误消息国际化
   - 缺少详细的错误日志记录

4. **权限控制不精细**：
   - 没有基于角色的访问控制
   - 缺少细粒度的权限管理
   - 没有实现权限检查中间件

## 2. API完整性评估

### 2.1 功能完整性

#### 2.1.1 现有功能

1. **用户语言设置管理**：
   - 获取用户语言设置
   - 更新用户语言设置

2. **语言资源管理**：
   - 获取语言资源版本
   - 获取指定语言的语言资源

#### 2.1.2 缺失功能

1. **语言资源管理**：
   - 缺少添加、修改、删除语言资源的API
   - 缺少语言资源批量导入/导出功能
   - 缺少语言资源版本历史管理

2. **用户语言设置批量管理**：
   - 缺少批量获取用户语言设置的API
   - 缺少批量更新用户语言设置的API
   - 缺少重置用户语言设置的API

3. **系统语言配置**：
   - 缺少获取系统支持的语言列表API
   - 缺少获取系统默认语言设置API
   - 缺少修改系统默认语言设置API

4. **翻译管理**：
   - 缺少翻译质量检查API
   - 缺少翻译覆盖率统计API
   - 缺少翻译一致性检查API

### 2.2 API设计规范性

#### 2.2.1 优点

1. **RESTful设计**：
   - 使用HTTP方法表示操作类型
   - 使用URL路径表示资源

2. **统一响应格式**：
   - 使用统一的响应结构
   - 包含状态码、消息和数据

3. **参数验证**：
   - 使用Pydantic模型进行参数验证
   - 确保输入数据的合法性

#### 2.2.2 缺点

1. **API版本控制**：
   - 没有明确的API版本控制策略
   - URL中没有包含版本信息

2. **文档不完整**：
   - 缺少API文档生成
   - 缺少接口参数详细说明
   - 缺少示例请求和响应

3. **状态码使用不规范**：
   - 所有成功响应都使用200状态码
   - 没有充分利用HTTP状态码语义

4. **分页和过滤**：
   - 缺少分页机制
   - 缺少数据过滤和排序功能

## 3. 安全性改进建议

### 3.1 认证和授权增强

#### 3.1.1 实现JWT认证

```python
# app/core/security.py
from datetime import datetime, timedelta
from typing import Optional
from jose import JWTError, jwt
from passlib.context import CryptContext
from fastapi import HTTPException, status
from app.core.config import settings

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def create_access_token(data: dict, expires_delta: Optional[timedelta] = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, settings.SECRET_KEY, algorithm=settings.ALGORITHM)
    return encoded_jwt

def verify_token(token: str):
    try:
        payload = jwt.decode(token, settings.SECRET_KEY, algorithms=[settings.ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise HTTPException(
                status_code=status.HTTP_401_UNAUTHORIZED,
                detail="Could not validate credentials",
                headers={"WWW-Authenticate": "Bearer"},
            )
        return username
    except JWTError:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Could not validate credentials",
            headers={"WWW-Authenticate": "Bearer"},
        )

def verify_password(plain_password, hashed_password):
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password):
    return pwd_context.hash(password)
```

#### 3.1.2 实现基于角色的访问控制

```python
# app/core/rbac.py
from enum import Enum
from typing import List, Optional
from fastapi import HTTPException, status
from app.models.user import User
from app.models.role import Role
from app.models.permission import Permission

class PermissionEnum(str, Enum):
    USER_VIEW = "USER_VIEW"
    USER_EDIT = "USER_EDIT"
    USER_DELETE = "USER_DELETE"
    CONTENT_VIEW = "CONTENT_VIEW"
    CONTENT_CREATE = "CONTENT_CREATE"
    CONTENT_EDIT = "CONTENT_EDIT"
    CONTENT_DELETE = "CONTENT_DELETE"
    SYSTEM_CONFIG = "SYSTEM_CONFIG"
    ROLE_MANAGE = "ROLE_MANAGE"
    PERMISSION_MANAGE = "PERMISSION_MANAGE"
    I18N_MANAGE = "I18N_MANAGE"  # 国际化管理权限

def has_permission(user: User, permission: PermissionEnum) -> bool:
    """检查用户是否具有指定权限"""
    # 超级管理员拥有所有权限
    if user.is_superuser:
        return True
    
    # 检查用户角色是否具有指定权限
    for role in user.roles:
        for role_permission in role.permissions:
            if role_permission.permission.code == permission.value:
                return True
    
    return False

def require_permission(permission: PermissionEnum):
    """装饰器：要求用户具有指定权限"""
    def decorator(func):
        async def wrapper(*args, **kwargs):
            # 从kwargs中获取current_user
            current_user = kwargs.get('current_user')
            if not current_user:
                raise HTTPException(
                    status_code=status.HTTP_401_UNAUTHORIZED,
                    detail="Authentication required"
                )
            
            if not has_permission(current_user, permission):
                raise HTTPException(
                    status_code=status.HTTP_403_FORBIDDEN,
                    detail="Insufficient permissions"
                )
            
            return await func(*args, **kwargs)
        return wrapper
    return decorator
```

#### 3.1.3 增强API路由安全性

```python
# app/api/v1/i18n.py
from fastapi import APIRouter, Depends, HTTPException, Request, Security
from sqlalchemy.orm import Session
from app.core.security import get_current_user, verify_token
from app.core.rbac import require_permission, PermissionEnum
from app.crud.i18n import crud_user_locale
from app.schemas.i18n import UserLocaleCreate, UserLocaleUpdate
from app.models.user import User
from app.core.database import get_db
from app.core.rate_limiter import RateLimiter

router = APIRouter()

# 创建速率限制器
locale_rate_limiter = RateLimiter(requests=10, window=60)  # 每分钟最多10次请求

@router.get("/user/locale")
async def get_user_locale(
    request: Request,
    current_user: User = Security(get_current_user),
    db: Session = Depends(get_db)
):
    """获取用户语言设置"""
    try:
        # 应用速率限制
        await locale_rate_limiter.check_rate_limit(request)
        
        user_locale = crud_user_locale.get_by_user_id(db, user_id=current_user.id)
        
        if not user_locale:
            # 如果没有设置，返回默认语言
            return {
                "code": 200,
                "message": "Success",
                "data": {
                    "locale": "zh-CN"
                }
            }
        
        return {
            "code": 200,
            "message": "Success",
            "data": {
                "locale": user_locale.locale
            }
        }
    except Exception as e:
        # 记录错误日志
        log_error(request, e)
        raise HTTPException(status_code=500, detail="Internal server error")

@router.put("/user/locale")
async def update_user_locale(
    locale_data: UserLocaleUpdate,
    request: Request,
    current_user: User = Security(get_current_user),
    db: Session = Depends(get_db)
):
    """更新用户语言设置"""
    try:
        # 应用速率限制
        await locale_rate_limiter.check_rate_limit(request)
        
        # 验证语言代码
        supported_locales = ['en', 'zh-CN', 'zh-TW']
        if locale_data.locale not in supported_locales:
            raise HTTPException(
                status_code=status.HTTP_400_BAD_REQUEST,
                detail="Unsupported locale"
            )
        
        # 查找现有设置
        user_locale = crud_user_locale.get_by_user_id(db, user_id=current_user.id)
        
        if user_locale:
            # 更新现有设置
            updated_locale = crud_user_locale.update(
                db, db_obj=user_locale, obj_in=locale_data
            )
        else:
            # 创建新设置
            locale_in = UserLocaleCreate(
                user_id=current_user.id,
                locale=locale_data.locale
            )
            updated_locale = crud_user_locale.create(db, obj_in=locale_in)
        
        # 记录操作日志
        log_action(
            user_id=current_user.id,
            action="update_locale",
            details=f"Updated locale to {locale_data.locale}",
            request=request
        )
        
        return {
            "code": 200,
            "message": "Locale updated successfully",
            "data": {
                "locale": updated_locale.locale
            }
        }
    except HTTPException:
        raise
    except Exception as e:
        # 记录错误日志
        log_error(request, e)
        raise HTTPException(status_code=500, detail="Internal server error")

# 管理员API - 需要I18N_MANAGE权限
@router.get("/admin/locales")
@require_permission(PermissionEnum.I18N_MANAGE)
async def get_all_locales(
    request: Request,
    current_user: User = Security(get_current_user),
    db: Session = Depends(get_db)
):
    """获取所有语言资源（管理员功能）"""
    try:
        # 应用速率限制
        await locale_rate_limiter.check_rate_limit(request)
        
        # 获取所有支持的语言
        locales = ['en', 'zh-CN', 'zh-TW']
        locale_data = {}
        
        for locale in locales:
            translations = crud_user_locale.get_translations_by_locale(db, locale=locale)
            formatted_data = {}
            
            for translation in translations:
                keys = translation.key.split('.')
                current = formatted_data
                
                for i, key in enumerate(keys):
                    if i === len(keys) - 1:
                        current[key] = translation.value
                    else:
                        if key not in current:
                            current[key] = {}
                        current = current[key]
            
            locale_data[locale] = formatted_data
        
        return {
            "code": 200,
            "message": "Success",
            "data": locale_data
        }
    except Exception as e:
        # 记录错误日志
        log_error(request, e)
        raise HTTPException(status_code=500, detail="Internal server error")
```

### 3.2 实现速率限制

```python
# app/core/rate_limiter.py
import time
from typing import Dict
from fastapi import HTTPException, Request, status
from collections import defaultdict

class RateLimiter:
    def __init__(self, requests: int, window: int):
        self.requests = requests
        self.window = window
        self.requests_cache: Dict[str, list] = defaultdict(list)
    
    async def check_rate_limit(self, request: Request):
        """检查请求是否超过速率限制"""
        # 获取客户端标识（IP地址或用户ID）
        client_id = self._get_client_id(request)
        
        # 获取当前时间戳
        current_time = time.time()
        
        # 清理过期的请求记录
        self._clean_expired_requests(client_id, current_time)
        
        # 检查请求数量是否超过限制
        if len(self.requests_cache[client_id]) >= self.requests:
            raise HTTPException(
                status_code=status.HTTP_429_TOO_MANY_REQUESTS,
                detail="Rate limit exceeded"
            )
        
        # 记录当前请求
        self.requests_cache[client_id].append(current_time)
    
    def _get_client_id(self, request: Request) -> str:
        """获取客户端标识"""
        # 如果用户已认证，使用用户ID
        if hasattr(request.state, 'user') and request.state.user:
            return f"user:{request.state.user.id}"
        
        # 否则使用IP地址
        client_host = request.client.host if request.client else "unknown"
        return f"ip:{client_host}"
    
    def _clean_expired_requests(self, client_id: str, current_time: float):
        """清理过期的请求记录"""
        if client_id not in self.requests_cache:
            return
        
        # 保留窗口期内的请求记录
        valid_requests = [
            timestamp for timestamp in self.requests_cache[client_id]
            if current_time - timestamp <= self.window
        ]
        
        self.requests_cache[client_id] = valid_requests
```

### 3.3 实现请求签名验证

```python
# app/core/signature.py
import hmac
import hashlib
import time
from fastapi import HTTPException, Request, status
from app.core.config import settings

def verify_request_signature(request: Request) -> bool:
    """验证请求签名"""
    # 获取请求参数
    method = request.method
    path = request.url.path
    query_string = request.url.query
    body = await request.body()
    
    # 获取请求头中的签名信息
    timestamp = request.headers.get('X-Timestamp')
    nonce = request.headers.get('X-Nonce')
    signature = request.headers.get('X-Signature')
    
    if not all([timestamp, nonce, signature]):
        return False
    
    # 检查时间戳是否在有效期内（5分钟）
    current_time = int(time.time())
    request_time = int(timestamp)
    
    if abs(current_time - request_time) > 300:
        return False
    
    # 构建签名字符串
    sign_string = f"{method}\n{path}\n{query_string}\n{body.decode('utf-8')}\n{timestamp}\n{nonce}"
    
    # 计算签名
    computed_signature = hmac.new(
        settings.API_SECRET_KEY.encode('utf-8'),
        sign_string.encode('utf-8'),
        hashlib.sha256
    ).hexdigest()
    
    # 比较签名
    return hmac.compare_digest(computed_signature, signature)

def require_signature():
    """装饰器：要求请求签名验证"""
    def decorator(func):
        async def wrapper(request: Request, *args, **kwargs):
            if not await verify_request_signature(request):
                raise HTTPException(
                    status_code=status.HTTP_401_UNAUTHORIZED,
                    detail="Invalid request signature"
                )
            
            return await func(request, *args, **kwargs)
        return wrapper
    return decorator
```

### 3.4 实现日志记录

```python
# app/core/logging.py
import logging
import json
from datetime import datetime
from typing import Dict, Any, Optional
from fastapi import Request
from app.models.user import User

# 配置日志
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

def log_error(request: Request, error: Exception, user: Optional[User] = None):
    """记录错误日志"""
    error_data = {
        "timestamp": datetime.utcnow().isoformat(),
        "type": "error",
        "error_type": type(error).__name__,
        "error_message": str(error),
        "request": {
            "method": request.method,
            "url": str(request.url),
            "headers": dict(request.headers),
            "client": {
                "host": request.client.host if request.client else None,
                "port": request.client.port if request.client else None
            }
        }
    }
    
    if user:
        error_data["user"] = {
            "id": user.id,
            "username": user.username
        }
    
    logger.error(json.dumps(error_data, ensure_ascii=False))

def log_action(user_id: int, action: str, details: str, request: Request):
    """记录用户操作日志"""
    action_data = {
        "timestamp": datetime.utcnow().isoformat(),
        "type": "action",
        "user_id": user_id,
        "action": action,
        "details": details,
        "request": {
            "method": request.method,
            "url": str(request.url),
            "client": {
                "host": request.client.host if request.client else None,
                "port": request.client.port if request.client else None
            }
        }
    }
    
    logger.info(json.dumps(action_data, ensure_ascii=False))

def log_security_event(event_type: str, details: Dict[str, Any], request: Request):
    """记录安全事件日志"""
    security_data = {
        "timestamp": datetime.utcnow().isoformat(),
        "type": "security",
        "event_type": event_type,
        "details": details,
        "request": {
            "method": request.method,
            "url": str(request.url),
            "headers": dict(request.headers),
            "client": {
                "host": request.client.host if request.client else None,
                "port": request.client.port if request.client else None
            }
        }
    }
    
    logger.warning(json.dumps(security_data, ensure_ascii=False))
```

## 4. API完整性改进建议

### 4.1 扩展API功能

#### 4.1.1 语言资源管理API

```python
# app/api/v1/i18n.py

@router.post("/admin/locales")
@require_permission(PermissionEnum.I18N_MANAGE)
async def create_translation(
    translation_data: TranslationCreate,
    request: Request,
    current_user: User = Security(get_current_user),
    db: Session = Depends(get_db)
):
    """创建翻译（管理员功能）"""
    try:
        # 验证语言代码
        if translation_data.locale not in ['en', 'zh-CN', 'zh-TW']:
            raise HTTPException(
                status_code=status.HTTP_400_BAD_REQUEST,
                detail="Unsupported locale"
            )
        
        # 检查翻译是否已存在
        existing_translation = crud_translation.get_by_key_and_locale(
            db, key=translation_data.key, locale=translation_data.locale
        )
        
        if existing_translation:
            raise HTTPException(
                status_code=status.HTTP_409_CONFLICT,
                detail="Translation already exists"
            )
        
        # 创建翻译
        translation = crud_translation.create(db, obj_in=translation_data)
        
        # 记录操作日志
        log_action(
            user_id=current_user.id,
            action="create_translation",
            details=f"Created translation for key {translation_data.key} in locale {translation_data.locale}",
            request=request
        )
        
        return {
            "code": 201,
            "message": "Translation created successfully",
            "data": {
                "id": translation.id,
                "key": translation.key,
                "locale": translation.locale,
                "value": translation.value
            }
        }
    except HTTPException:
        raise
    except Exception as e:
        log_error(request, e, current_user)
        raise HTTPException(status_code=500, detail="Internal server error")

@router.put("/admin/locales/{translation_id}")
@require_permission(PermissionEnum.I18N_MANAGE)
async def update_translation(
    translation_id: int,
    translation_data: TranslationUpdate,
    request: Request,
    current_user: User = Security(get_current_user),
    db: Session = Depends(get_db)
):
    """更新翻译（管理员功能）"""
    try:
        # 获取现有翻译
        translation = crud_translation.get(db, id=translation_id)
        
        if not translation:
            raise HTTPException(
                status_code=status.HTTP_404_NOT_FOUND,
                detail="Translation not found"
            )
        
        # 更新翻译
        updated_translation = crud_translation.update(
            db, db_obj=translation, obj_in=translation_data
        )
        
        # 记录操作日志
        log_action(
            user_id=current_user.id,
            action="update_translation",
            details=f"Updated translation for key {translation.key} in locale {translation.locale}",
            request=request
        )
        
        return {
            "code": 200,
            "message": "Translation updated successfully",
            "data": {
                "id": updated_translation.id,
                "key": updated_translation.key,
                "locale": updated_translation.locale,
                "value": updated_translation.value
            }
        }
    except HTTPException:
        raise
    except Exception as e:
        log_error(request, e, current_user)
        raise HTTPException(status_code=500, detail="Internal server error")

@router.delete("/admin/locales/{translation_id}")
@require_permission(PermissionEnum.I18N_MANAGE)
async def delete_translation(
    translation_id: int,
    request: Request,
    current_user: User = Security(get_current_user),
    db: Session = Depends(get_db)
):
    """删除翻译（管理员功能）"""
    try:
        # 获取现有翻译
        translation = crud_translation.get(db, id=translation_id)
        
        if not translation:
            raise HTTPException(
                status_code=status.HTTP_404_NOT_FOUND,
                detail="Translation not found"
            )
        
        # 删除翻译
        crud_translation.remove(db, id=translation_id)
        
        # 记录操作日志
        log_action(
            user_id=current_user.id,
            action="delete_translation",
            details=f"Deleted translation for key {translation.key} in locale {translation.locale}",
            request=request
        )
        
        return {
            "code": 200,
            "message": "Translation deleted successfully",
            "data": None
        }
    except HTTPException:
        raise
    except Exception as e:
        log_error(request, e, current_user)
        raise HTTPException(status_code=500, detail="Internal server error")
```

#### 4.1.2 批量操作API

```python
# app/api/v1/i18n.py

@router.post("/admin/locales/batch")
@require_permission(PermissionEnum.I18N_MANAGE)
async def batch_create_translations(
    translations_data: List[TranslationCreate],
    request: Request,
    current_user: User = Security(get_current_user),
    db: Session = Depends(get_db)
):
    """批量创建翻译（管理员功能）"""
    try:
        created_translations = []
        failed_translations = []
        
        for translation_data in translations_data:
            try:
                # 验证语言代码
                if translation_data.locale not in ['en', 'zh-CN', 'zh-TW']:
                    failed_translations.append({
                        "data": translation_data,
                        "error": "Unsupported locale"
                    })
                    continue
                
                # 检查翻译是否已存在
                existing_translation = crud_translation.get_by_key_and_locale(
                    db, key=translation_data.key, locale=translation_data.locale
                )
                
                if existing_translation:
                    failed_translations.append({
                        "data": translation_data,
                        "error": "Translation already exists"
                    })
                    continue
                
                # 创建翻译
                translation = crud_translation.create(db, obj_in=translation_data)
                created_translations.append(translation)
            except Exception as e:
                failed_translations.append({
                    "data": translation_data,
                    "error": str(e)
                })
        
        # 记录操作日志
        log_action(
            user_id=current_user.id,
            action="batch_create_translations",
            details=f"Created {len(created_translations)} translations, failed {len(failed_translations)} translations",
            request=request
        )
        
        return {
            "code": 201,
            "message": "Batch translation creation completed",
            "data": {
                "created": [
                    {
                        "id": t.id,
                        "key": t.key,
                        "locale": t.locale,
                        "value": t.value
                    }
                    for t in created_translations
                ],
                "failed": failed_translations
            }
        }
    except Exception as e:
        log_error(request, e, current_user)
        raise HTTPException(status_code=500, detail="Internal server error")

@router.post("/admin/locales/import")
@require_permission(PermissionEnum.I18N_MANAGE)
async def import_translations(
    file: UploadFile = File(...),
    request: Request,
    current_user: User = Security(get_current_user),
    db: Session = Depends(get_db)
):
    """导入翻译文件（管理员功能）"""
    try:
        # 检查文件类型
        if not file.filename.endswith('.json'):
            raise HTTPException(
                status_code=status.HTTP_400_BAD_REQUEST,
                detail="Only JSON files are supported"
            )
        
        # 读取文件内容
        content = await file.read()
        try:
            translations_data = json.loads(content)
        except json.JSONDecodeError:
            raise HTTPException(
                status_code=status.HTTP_400_BAD_REQUEST,
                detail="Invalid JSON format"
            )
        
        # 处理导入
        import_results = await _process_import_translations(
            db, translations_data, current_user.id, request
        )
        
        return {
            "code": 200,
            "message": "Translations imported successfully",
            "data": import_results
        }
    except HTTPException:
        raise
    except Exception as e:
        log_error(request, e, current_user)
        raise HTTPException(status_code=500, detail="Internal server error")

async def _process_import_translations(db, translations_data, user_id, request):
    """处理翻译导入"""
    created_count = 0
    updated_count = 0
    failed_count = 0
    
    for locale, translations in translations_data.items():
        # 验证语言代码
        if locale not in ['en', 'zh-CN', 'zh-TW']:
            failed_count += len(translations)
            continue
        
        for key, value in translations.items():
            try:
                # 检查翻译是否已存在
                existing_translation = crud_translation.get_by_key_and_locale(
                    db, key=key, locale=locale
                )
                
                if existing_translation:
                    # 更新现有翻译
                    crud_translation.update(
                        db,
                        db_obj=existing_translation,
                        obj_in={"value": value}
                    )
                    updated_count += 1
                else:
                    # 创建新翻译
                    translation_data = TranslationCreate(
                        key=key,
                        locale=locale,
                        value=value
                    )
                    crud_translation.create(db, obj_in=translation_data)
                    created_count += 1
            except Exception as e:
                failed_count += 1
    
    # 记录操作日志
    log_action(
        user_id=user_id,
        action="import_translations",
        details=f"Imported translations: created={created_count}, updated={updated_count}, failed={failed_count}",
        request=request
    )
    
    return {
        "created": created_count,
        "updated": updated_count,
        "failed": failed_count
    }
```

### 4.2 API设计规范化

#### 4.2.1 实现API版本控制

```python
# app/main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from app.api.v1.api import api_router
from app.core.config import settings

app = FastAPI(
    title=settings.PROJECT_NAME,
    openapi_url=f"{settings.API_V1_STR}/openapi.json"
)

# 设置CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.BACKEND_CORS_ORIGINS,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# 包含API路由
app.include_router(api_router, prefix=settings.API_V1_STR)
```

#### 4.2.2 统一响应格式和错误处理

```python
# app/utils/response.py
from fastapi import HTTPException, status
from typing import Any, Dict, Optional

class APIResponse:
    """统一API响应格式"""
    
    @staticmethod
    def success(
        data: Any = None,
        message: str = "Success",
        code: int = 200,
        **kwargs
    ) -> Dict[str, Any]:
        """成功响应"""
        response = {
            "code": code,
            "message": message,
            "data": data
        }
        response.update(kwargs)
        return response
    
    @staticmethod
    def error(
        message: str,
        code: int = 400,
        details: Optional[Any] = None,
        **kwargs
    ) -> Dict[str, Any]:
        """错误响应"""
        response = {
            "code": code,
            "message": message,
            "data": None
        }
        
        if details is not None:
            response["details"] = details
        
        response.update(kwargs)
        return response

class APIException(HTTPException):
    """自定义API异常"""
    
    def __init__(
        self,
        message: str,
        code: int = 400,
        details: Optional[Any] = None,
        headers: Optional[Dict[str, Any]] = None
    ):
        super().__init__(status_code=code, detail=message)
        self.details = details
        self.headers = headers

class ErrorCode:
    """错误代码"""
    
    # 通用错误 (1000-1999)
    VALIDATION_ERROR = 1001
    AUTHENTICATION_ERROR = 1002
    AUTHORIZATION_ERROR = 1003
    NOT_FOUND = 1004
    CONFLICT = 1005
    RATE_LIMIT_EXCEEDED = 1006
    
    # 国际化相关错误 (2000-2999)
    UNSUPPORTED_LOCALE = 2001
    TRANSLATION_NOT_FOUND = 2002
    INVALID_TRANSLATION_FORMAT = 2003
    IMPORT_ERROR = 2004
    EXPORT_ERROR = 2005
```

#### 4.2.3 实现API文档生成

```python
# app/api/v1/api.py
from fastapi import APIRouter
from app.api.v1.endpoints import i18n

api_router = APIRouter()

api_router.include_router(i18n.router, prefix="/i18n", tags=["internationalization"])
```

```python
# app/api/v1/endpoints/i18n.py
from fastapi import APIRouter, Depends, HTTPException, Request, Security, UploadFile, File
from sqlalchemy.orm import Session
from typing import List
from app.core.security import get_current_user
from app.core.rbac import require_permission, PermissionEnum
from app.crud import crud_translation, crud_user_locale
from app.schemas import TranslationCreate, TranslationUpdate
from app.models.user import User
from app.core.database import get_db
from app.core.rate_limiter import RateLimiter
from app.utils.response import APIResponse, ErrorCode, APIException

router = APIRouter()

# 创建速率限制器
locale_rate_limiter = RateLimiter(requests=10, window=60)  # 每分钟最多10次请求

@router.get(
    "/user/locale",
    response_model=dict,
    summary="获取用户语言设置",
    description="获取当前登录用户的语言设置，如果没有设置则返回默认语言"
)
async def get_user_locale(
    request: Request,
    current_user: User = Security(get_current_user),
    db: Session = Depends(get_db)
):
    """获取用户语言设置"""
    try:
        # 应用速率限制
        await locale_rate_limiter.check_rate_limit(request)
        
        user_locale = crud_user_locale.get_by_user_id(db, user_id=current_user.id)
        
        if not user_locale:
            # 如果没有设置，返回默认语言
            return APIResponse.success(
                data={"locale": "zh-CN"},
                message="No locale set, using default"
            )
        
        return APIResponse.success(data={"locale": user_locale.locale})
    except Exception as e:
        raise APIException(
            message="Failed to get user locale",
            code=ErrorCode.INTERNAL_ERROR,
            details=str(e)
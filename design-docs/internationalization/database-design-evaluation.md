# 数据库设计多语言存储需求评估

## 1. 现有数据库设计分析

### 1.1 数据库表结构概述

根据文档分析，当前的国际化相关数据库表主要包括：

1. **用户语言设置表 (user_locales)**：
   - 存储用户的语言偏好设置
   - 支持用户自定义语言选择

2. **翻译表 (translations)**：
   - 存储键值对形式的翻译内容
   - 支持多语言版本的翻译

3. **多语言内容表 (localized_contents)**：
   - 存储实体内容的多语言版本
   - 支持角色、权限等实体的多语言字段

### 1.2 数据库表设计分析

#### 1.2.1 用户语言设置表

```sql
-- 用户语言设置表
CREATE TABLE user_locales (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL UNIQUE,
    locale VARCHAR(10) NOT NULL DEFAULT 'zh-CN',
    is_default BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- 创建索引
CREATE INDEX idx_user_locales_user_id ON user_locales(user_id);
CREATE INDEX idx_user_locales_locale ON user_locales(locale);

-- 创建更新时间触发器
CREATE TRIGGER update_user_locales_updated_at BEFORE UPDATE ON user_locales FOR EACH ROW EXECUTE PROCEDURE update_updated_at_column();
```

#### 1.2.2 翻译表

```sql
-- 翻译表
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
CREATE INDEX idx_translations_key ON translations(key);
CREATE INDEX idx_translations_locale ON translations(locale);
CREATE INDEX idx_translations_key_locale ON translations(key, locale);

-- 创建更新时间触发器
CREATE TRIGGER update_translations_updated_at BEFORE UPDATE ON translations FOR EACH ROW EXECUTE PROCEDURE update_updated_at_column();
```

#### 1.2.3 多语言内容表

```sql
-- 多语言内容表
CREATE TABLE localized_contents (
    id BIGSERIAL PRIMARY KEY,
    content_type VARCHAR(50) NOT NULL,  -- 内容类型，如role, permission等
    content_id BIGINT NOT NULL,          -- 内容ID
    locale VARCHAR(10) NOT NULL,         -- 语言代码
    field_name VARCHAR(50) NOT NULL,    -- 字段名
    field_value TEXT NOT NULL,           -- 字段值
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    UNIQUE(content_type, content_id, locale, field_name)
);

-- 创建索引
CREATE INDEX idx_localized_contents_content_type ON localized_contents(content_type);
CREATE INDEX idx_localized_contents_content_id ON localized_contents(content_id);
CREATE INDEX idx_localized_contents_locale ON localized_contents(locale);
CREATE INDEX idx_localized_contents_field_name ON localized_contents(field_name);
CREATE INDEX idx_localized_contents_content_locale ON localized_contents(content_type, content_id, locale);

-- 创建更新时间触发器
CREATE TRIGGER update_localized_contents_updated_at BEFORE UPDATE ON localized_contents FOR EACH ROW EXECUTE PROCEDURE update_updated_at_column();
```

### 1.3 数据模型设计分析

#### 1.3.1 用户语言设置模型

```python
# app/models/i18n.py
from sqlalchemy import Column, Integer, String, DateTime, ForeignKey, Boolean
from sqlalchemy.orm import relationship
from app.models.base import Base
from datetime import datetime

class UserLocale(Base):
    """用户语言设置"""
    __tablename__ = 'user_locales'

    id = Column(Integer, primary_key=True, index=True)
    user_id = Column(Integer, ForeignKey('users.id'), unique=True, nullable=False)
    locale = Column(String(10), nullable=False, default='zh-CN')
    is_default = Column(Boolean, default=False)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

    # 关系
    user = relationship("User", back_populates="locale")
```

#### 1.3.2 翻译模型

```python
class Translation(Base):
    """翻译表"""
    __tablename__ = 'translations'

    id = Column(Integer, primary_key=True, index=True)
    key = Column(String(255), nullable=False)
    locale = Column(String(10), nullable=False)
    value = Column(Text, nullable=False)
    context = Column(String(255))
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

    __table_args__ = (
        # 确保同一语言下key唯一
        UniqueConstraint('key', 'locale', name='uq_translations_key_locale'),
    )
```

#### 1.3.3 多语言内容模型

```python
class LocalizedContent(Base):
    """多语言内容表"""
    __tablename__ = 'localized_contents'

    id = Column(Integer, primary_key=True, index=True)
    content_type = Column(String(50), nullable=False)  # 内容类型，如role, permission等
    content_id = Column(Integer, nullable=False)          # 内容ID
    locale = Column(String(10), nullable=False)         # 语言代码
    field_name = Column(String(50), nullable=False)    # 字段名
    field_value = Column(Text, nullable=False)         # 字段值
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

    __table_args__ = (
        # 确保同一内容、同一语言、同一字段的唯一性
        UniqueConstraint('content_type', 'content_id', 'locale', 'field_name', 
                         name='uq_localized_contents_content_locale_field'),
    )
```

### 1.4 数据库设计评估

#### 1.4.1 优点

1. **表结构清晰**：
   - 用户语言设置、翻译和多语言内容分离存储
   - 每个表职责明确，便于维护

2. **索引设计合理**：
   - 为常用查询字段创建了索引
   - 复合索引优化了多条件查询性能

3. **数据完整性保障**：
   - 使用唯一约束防止数据重复
   - 使用外键约束确保数据一致性

4. **扩展性好**：
   - 支持动态添加新语言
   - 支持动态添加新的翻译键

5. **时间戳管理**：
   - 记录创建和更新时间
   - 使用触发器自动更新时间戳

#### 1.4.2 缺点

1. **查询性能问题**：
   - 多语言内容查询需要多次连接或子查询
   - 复杂的多语言查询可能性能较差

2. **数据冗余**：
   - 翻译表和多语言内容表有部分功能重叠
   - 没有充分利用PostgreSQL的JSONB字段

3. **缺少版本控制**：
   - 翻译内容没有版本管理
   - 无法追踪翻译历史变更

4. **缺少缓存机制**：
   - 没有设计翻译内容的缓存策略
   - 频繁查询可能影响数据库性能

5. **缺少数据验证**：
   - 没有在数据库层面验证语言代码有效性
   - 没有验证翻译内容的完整性

## 2. 多语言存储需求分析

### 2.1 功能需求

#### 2.1.1 用户语言管理

1. **用户语言设置**：
   - 支持用户设置首选语言
   - 支持用户设置多种备用语言
   - 支持系统默认语言设置

2. **语言自动检测**：
   - 根据用户系统语言自动设置
   - 根据用户地理位置自动设置
   - 支持手动覆盖自动设置

#### 2.1.2 翻译内容管理

1. **翻译键值对管理**：
   - 支持动态添加翻译键
   - 支持翻译内容的CRUD操作
   - 支持翻译内容的批量导入导出

2. **翻译版本管理**：
   - 支持翻译内容的版本控制
   - 支持翻译历史记录查询
   - 支持翻译内容回滚

#### 2.1.3 多语言内容管理

1. **实体多语言化**：
   - 支持用户角色和权限的多语言名称和描述
   - 支持系统配置的多语言内容
   - 支持动态内容的多语言版本

2. **内容关联管理**：
   - 支持内容与多语言版本的关联
   - 支持多语言内容的批量更新
   - 支持多语言内容的完整性检查

### 2.2 性能需求

#### 2.2.1 查询性能

1. **快速获取翻译**：
   - 支持按语言和键快速获取翻译内容
   - 支持批量获取多个翻译内容
   - 支持获取嵌套结构的翻译内容

2. **高效多语言查询**：
   - 支持高效查询实体的多语言内容
   - 支持多语言内容的关联查询
   - 支持多语言内容的分页查询

#### 2.2.2 更新性能

1. **批量更新支持**：
   - 支持批量更新翻译内容
   - 支持批量更新多语言内容
   - 支持事务性更新操作

2. **索引优化**：
   - 为常用查询路径优化索引
   - 减少索引对写入性能的影响
   - 支持部分索引和条件索引

### 2.3 扩展性需求

#### 2.3.1 语言扩展

1. **动态语言支持**：
   - 支持动态添加新语言
   - 支持语言区域设置（如en-US, en-GB）
   - 支持语言的启用和禁用

2. **语言继承**：
   - 支持语言间的继承关系
   - 支持缺失翻译的回退机制
   - 支持自定义语言回退顺序

#### 2.3.2 内容扩展

1. **内容类型扩展**：
   - 支持动态添加新的内容类型
   - 支持自定义内容的多语言字段
   - 支持内容类型的版本管理

2. **存储结构扩展**：
   - 支持JSONB字段存储复杂多语言结构
   - 支持全文搜索多语言内容
   - 支持多语言内容的标签和分类

## 3. 数据库设计改进建议

### 3.1 优化表结构设计

#### 3.1.1 引入语言表

```sql
-- 语言表
CREATE TABLE languages (
    id SERIAL PRIMARY KEY,
    code VARCHAR(10) NOT NULL UNIQUE,         -- 语言代码，如en, zh-CN, zh-TW
    name VARCHAR(100) NOT NULL,             -- 语言名称，如English, 简体中文
    native_name VARCHAR(100) NOT NULL,       -- 本地语言名称
    flag_icon VARCHAR(255),                  -- 国旗图标URL
    is_active BOOLEAN DEFAULT TRUE,         -- 是否启用
    is_default BOOLEAN DEFAULT FALSE,        -- 是否默认语言
    sort_order INTEGER DEFAULT 0,            -- 排序顺序
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- 创建索引
CREATE INDEX idx_languages_code ON languages(code);
CREATE INDEX idx_languages_active ON languages(is_active);
CREATE INDEX idx_languages_sort ON languages(sort_order);

-- 创建更新时间触发器
CREATE TRIGGER update_languages_updated_at BEFORE UPDATE ON languages FOR EACH ROW EXECUTE PROCEDURE update_updated_at_column();

-- 插入初始数据
INSERT INTO languages (code, name, native_name, is_default, sort_order) VALUES
('en', 'English', 'English', FALSE, 1),
('zh-CN', 'Simplified Chinese', '简体中文', TRUE, 2),
('zh-TW', 'Traditional Chinese', '繁體中文', FALSE, 3);
```

#### 3.1.2 优化用户语言设置表

```sql
-- 用户语言设置表
CREATE TABLE user_locales (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL,
    language_id INTEGER NOT NULL REFERENCES languages(id) ON DELETE CASCADE,
    priority INTEGER DEFAULT 0,             -- 优先级，0为首选语言
    is_active BOOLEAN DEFAULT TRUE,         -- 是否启用
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    UNIQUE(user_id, language_id)
);

-- 创建索引
CREATE INDEX idx_user_locales_user_id ON user_locales(user_id);
CREATE INDEX idx_user_locales_language_id ON user_locales(language_id);
CREATE INDEX idx_user_locales_priority ON user_locales(user_id, priority);

-- 创建更新时间触发器
CREATE TRIGGER update_user_locales_updated_at BEFORE UPDATE ON user_locales FOR EACH ROW EXECUTE PROCEDURE update_updated_at_column();
```

#### 3.1.3 优化翻译表

```sql
-- 翻译表
CREATE TABLE translations (
    id BIGSERIAL PRIMARY KEY,
    language_id INTEGER NOT NULL REFERENCES languages(id) ON DELETE CASCADE,
    key VARCHAR(255) NOT NULL,
    value TEXT NOT NULL,
    context VARCHAR(255),
    version INTEGER DEFAULT 1,              -- 版本号
    is_active BOOLEAN DEFAULT TRUE,         -- 是否启用
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    UNIQUE(language_id, key)
);

-- 创建索引
CREATE INDEX idx_translations_language_id ON translations(language_id);
CREATE INDEX idx_translations_key ON translations(key);
CREATE INDEX idx_translations_language_key ON translations(language_id, key);
CREATE INDEX idx_translations_active ON translations(is_active);

-- 创建更新时间触发器
CREATE TRIGGER update_translations_updated_at BEFORE UPDATE ON translations FOR EACH ROW EXECUTE PROCEDURE update_updated_at_column();

-- 翻译历史表
CREATE TABLE translation_history (
    id BIGSERIAL PRIMARY KEY,
    translation_id BIGINT NOT NULL REFERENCES translations(id) ON DELETE CASCADE,
    value TEXT NOT NULL,
    version INTEGER NOT NULL,
    changed_by BIGINT REFERENCES users(id),
    changed_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    comment TEXT
);

-- 创建索引
CREATE INDEX idx_translation_history_translation_id ON translation_history(translation_id);
CREATE INDEX idx_translation_history_version ON translation_history(translation_id, version);
```

#### 3.1.4 优化多语言内容表

```sql
-- 多语言内容表
CREATE TABLE localized_contents (
    id BIGSERIAL PRIMARY KEY,
    content_type VARCHAR(50) NOT NULL,      -- 内容类型，如role, permission等
    content_id BIGINT NOT NULL,             -- 内容ID
    language_id INTEGER NOT NULL REFERENCES languages(id) ON DELETE CASCADE,
    field_name VARCHAR(50) NOT NULL,        -- 字段名
    field_value TEXT NOT NULL,               -- 字段值
    field_value_json JSONB,                 -- 字段值（JSON格式，用于复杂结构）
    version INTEGER DEFAULT 1,              -- 版本号
    is_active BOOLEAN DEFAULT TRUE,          -- 是否启用
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    UNIQUE(content_type, content_id, language_id, field_name)
);

-- 创建索引
CREATE INDEX idx_localized_contents_content_type ON localized_contents(content_type);
CREATE INDEX idx_localized_contents_content_id ON localized_contents(content_id);
CREATE INDEX idx_localized_contents_language_id ON localized_contents(language_id);
CREATE INDEX idx_localized_contents_field_name ON localized_contents(field_name);
CREATE INDEX idx_localized_contents_content_language ON localized_contents(content_type, content_id, language_id);
CREATE INDEX idx_localized_contents_active ON localized_contents(is_active);

-- 创建JSONB字段的GIN索引，支持JSON查询
CREATE INDEX idx_localized_contents_field_value_json ON localized_contents USING GIN(field_value_json);

-- 创建更新时间触发器
CREATE TRIGGER update_localized_contents_updated_at BEFORE UPDATE ON localized_contents FOR EACH ROW EXECUTE PROCEDURE update_updated_at_column();

-- 多语言内容历史表
CREATE TABLE localized_content_history (
    id BIGSERIAL PRIMARY KEY,
    localized_content_id BIGINT NOT NULL REFERENCES localized_contents(id) ON DELETE CASCADE,
    field_value TEXT NOT NULL,
    field_value_json JSONB,
    version INTEGER NOT NULL,
    changed_by BIGINT REFERENCES users(id),
    changed_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    comment TEXT
);

-- 创建索引
CREATE INDEX idx_localized_content_history_content_id ON localized_content_history(localized_content_id);
CREATE INDEX idx_localized_content_history_version ON localized_content_history(localized_content_id, version);
```

### 3.2 引入JSONB字段支持复杂多语言结构

#### 3.2.1 创建复杂多语言内容表

```sql
-- 复杂多语言内容表
CREATE TABLE complex_localized_contents (
    id BIGSERIAL PRIMARY KEY,
    content_type VARCHAR(50) NOT NULL,      -- 内容类型
    content_id BIGINT NOT NULL,             -- 内容ID
    language_id INTEGER NOT NULL REFERENCES languages(id) ON DELETE CASCADE,
    content_data JSONB NOT NULL,            -- 内容数据（JSON格式）
    version INTEGER DEFAULT 1,              -- 版本号
    is_active BOOLEAN DEFAULT TRUE,          -- 是否启用
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    UNIQUE(content_type, content_id, language_id)
);

-- 创建索引
CREATE INDEX idx_complex_localized_contents_content_type ON complex_localized_contents(content_type);
CREATE INDEX idx_complex_localized_contents_content_id ON complex_localized_contents(content_id);
CREATE INDEX idx_complex_localized_contents_language_id ON complex_localized_contents(language_id);
CREATE INDEX idx_complex_localized_contents_content_language ON complex_localized_contents(content_type, content_id, language_id);
CREATE INDEX idx_complex_localized_contents_active ON complex_localized_contents(is_active);

-- 创建JSONB字段的GIN索引，支持JSON查询
CREATE INDEX idx_complex_localized_contents_content_data ON complex_localized_contents USING GIN(content_data);

-- 创建更新时间触发器
CREATE TRIGGER update_complex_localized_contents_updated_at BEFORE UPDATE ON complex_localized_contents FOR EACH ROW EXECUTE PROCEDURE update_updated_at_column();
```

#### 3.2.2 使用JSONB存储嵌套翻译

```sql
-- 嵌套翻译表
CREATE TABLE nested_translations (
    id BIGSERIAL PRIMARY KEY,
    language_id INTEGER NOT NULL REFERENCES languages(id) ON DELETE CASCADE,
    namespace VARCHAR(100) NOT NULL,        -- 命名空间，用于分组翻译
    key_path VARCHAR(500) NOT NULL,        -- 键路径，如'user.profile.name'
    value JSONB NOT NULL,                   -- 翻译值（支持复杂类型）
    context VARCHAR(255),
    version INTEGER DEFAULT 1,              -- 版本号
    is_active BOOLEAN DEFAULT TRUE,         -- 是否启用
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    
    UNIQUE(language_id, namespace, key_path)
);

-- 创建索引
CREATE INDEX idx_nested_translations_language_id ON nested_translations(language_id);
CREATE INDEX idx_nested_translations_namespace ON nested_translations(namespace);
CREATE INDEX idx_nested_translations_key_path ON nested_translations(key_path);
CREATE INDEX idx_nested_translations_namespace_key ON nested_translations(namespace, key_path);
CREATE INDEX idx_nested_translations_active ON nested_translations(is_active);

-- 创建JSONB字段的GIN索引，支持JSON查询
CREATE INDEX idx_nested_translations_value ON nested_translations USING GIN(value);

-- 创建更新时间触发器
CREATE TRIGGER update_nested_translations_updated_at BEFORE UPDATE ON nested_translations FOR EACH ROW EXECUTE PROCEDURE update_updated_at_column();
```

### 3.3 实现缓存机制

#### 3.3.1 创建翻译缓存表

```sql
-- 翻译缓存表
CREATE TABLE translation_cache (
    id BIGSERIAL PRIMARY KEY,
    cache_key VARCHAR(500) NOT NULL UNIQUE, -- 缓存键
    cache_value JSONB NOT NULL,              -- 缓存值（JSON格式）
    language_id INTEGER NOT NULL REFERENCES languages(id) ON DELETE CASCADE,
    expires_at TIMESTAMP WITH TIME ZONE NOT NULL, -- 过期时间
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    accessed_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- 创建索引
CREATE INDEX idx_translation_cache_key ON translation_cache(cache_key);
CREATE INDEX idx_translation_cache_language_id ON translation_cache(language_id);
CREATE INDEX idx_translation_cache_expires_at ON translation_cache(expires_at);
CREATE INDEX idx_translation_cache_accessed_at ON translation_cache(accessed_at);

-- 创建更新时间触发器
CREATE TRIGGER update_translation_cache_accessed_at BEFORE UPDATE ON translation_cache FOR EACH ROW EXECUTE PROCEDURE update_accessed_at_column();
```

#### 3.3.2 创建缓存刷新触发器

```sql
-- 翻译表更新触发器
CREATE OR REPLACE FUNCTION refresh_translation_cache()
RETURNS TRIGGER AS $$
BEGIN
    -- 当翻译内容更新时，清除相关缓存
    DELETE FROM translation_cache 
    WHERE language_id = NEW.language_id 
    AND cache_key LIKE '%' || NEW.key || '%';
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER tr_translations_refresh_cache
AFTER INSERT OR UPDATE OR DELETE ON translations
FOR EACH ROW
EXECUTE FUNCTION refresh_translation_cache();

-- 多语言内容表更新触发器
CREATE OR REPLACE FUNCTION refresh_localized_content_cache()
RETURNS TRIGGER AS $$
BEGIN
    -- 当多语言内容更新时，清除相关缓存
    DELETE FROM translation_cache 
    WHERE language_id = NEW.language_id 
    AND cache_key LIKE '%' || NEW.content_type || '%' || NEW.content_id || '%';
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER tr_localized_contents_refresh_cache
AFTER INSERT OR UPDATE OR DELETE ON localized_contents
FOR EACH ROW
EXECUTE FUNCTION refresh_localized_content_cache();
```

### 3.4 实现数据验证和完整性检查

#### 3.4.1 创建翻译完整性检查函数

```sql
-- 检查翻译完整性
CREATE OR REPLACE FUNCTION check_translation_completeness(source_language_id INTEGER, target_language_id INTEGER)
RETURNS TABLE(key VARCHAR(255), source_value TEXT, target_value TEXT, status VARCHAR(20)) AS $$
BEGIN
    RETURN QUERY
    SELECT 
        t1.key,
        t1.value AS source_value,
        COALESCE(t2.value, '') AS target_value,
        CASE 
            WHEN t2.value IS NULL THEN 'missing'
            WHEN t2.value = '' THEN 'empty'
            ELSE 'complete'
        END AS status
    FROM 
        translations t1
    LEFT JOIN 
        translations t2 ON t1.key = t2.key AND t2.language_id = target_language_id
    WHERE 
        t1.language_id = source_language_id
        AND t1.is_active = TRUE;
END;
$$ LANGUAGE plpgsql;

-- 检查多语言内容完整性
CREATE OR REPLACE FUNCTION check_localized_content_completeness(content_type VARCHAR, source_language_id INTEGER, target_language_id INTEGER)
RETURNS TABLE(content_id BIGINT, field_name VARCHAR(50), source_value TEXT, target_value TEXT, status VARCHAR(20)) AS $$
BEGIN
    RETURN QUERY
    SELECT 
        lc1.content_id,
        lc1.field_name,
        lc1.field_value AS source_value,
        COALESCE(lc2.field_value, '') AS target_value,
        CASE 
            WHEN lc2.field_value IS NULL THEN 'missing'
            WHEN lc2.field_value = '' THEN 'empty'
            ELSE 'complete'
        END AS status
    FROM 
        localized_contents lc1
    LEFT JOIN 
        localized_contents lc2 ON lc1.content_id = lc2.content_id 
        AND lc1.field_name = lc2.field_name 
        AND lc2.language_id = target_language_id
    WHERE 
        lc1.content_type = content_type
        AND lc1.language_id = source_language_id
        AND lc1.is_active = TRUE;
END;
$$ LANGUAGE plpgsql;
```

#### 3.4.2 创建数据验证触发器

```sql
-- 验证语言代码
CREATE OR REPLACE FUNCTION validate_language_id()
RETURNS TRIGGER AS $$
BEGIN
    -- 检查语言ID是否存在且启用
    IF NOT EXISTS (
        SELECT 1 FROM languages 
        WHERE id = NEW.language_id AND is_active = TRUE
    ) THEN
        RAISE EXCEPTION 'Invalid or inactive language_id: %', NEW.language_id;
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- 为翻译表添加验证触发器
CREATE TRIGGER tr_translations_validate_language_id
BEFORE INSERT OR UPDATE ON translations
FOR EACH ROW
EXECUTE FUNCTION validate_language_id();

-- 为多语言内容表添加验证触发器
CREATE TRIGGER tr_localized_contents_validate_language_id
BEFORE INSERT OR UPDATE ON localized_contents
FOR EACH ROW
EXECUTE FUNCTION validate_language_id();

-- 验证翻译内容不为空
CREATE OR REPLACE FUNCTION validate_translation_value()
RETURNS TRIGGER AS $$
BEGIN
    -- 检查翻译内容是否为空
    IF NEW.value IS NULL OR TRIM(NEW.value) = '' THEN
        RAISE EXCEPTION 'Translation value cannot be empty';
    END IF;
    
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- 为翻译表添加验证触发器
CREATE TRIGGER tr_translations_validate_value
BEFORE INSERT OR UPDATE ON translations
FOR EACH ROW
EXECUTE FUNCTION validate_translation_value();
```

## 4. 数据库查询优化建议

### 4.1 创建物化视图

#### 4.1.1 创建翻译物化视图

```sql
-- 创建翻译物化视图
CREATE MATERIALIZED VIEW mv_translations_summary AS
SELECT 
    l.id AS language_id,
    l.code AS language_code,
    l.name AS language_name,
    COUNT(t.id) AS translation_count,
    MAX(t.updated_at) AS last_updated
FROM 
    languages l
LEFT JOIN 
    translations t ON l.id = t.language_id AND t.is_active = TRUE
WHERE 
    l.is_active = TRUE
GROUP BY 
    l.id, l.code, l.name;

-- 创建索引
CREATE UNIQUE INDEX idx_mv_translations_summary_language_id ON mv_translations_summary(language_id);
CREATE INDEX idx_mv_translations_summary_language_code ON mv_translations_summary(language_code);

-- 创建刷新函数
CREATE OR REPLACE FUNCTION refresh_mv_translations_summary()
RETURNS VOID AS $$
BEGIN
    REFRESH MATERIALIZED VIEW mv_translations_summary;
END;
$$ LANGUAGE plpgsql;
```

#### 4.1.2 创建多语言内容物化视图

```sql
-- 创建多语言内容物化视图
CREATE MATERIALIZED VIEW mv_localized_contents_summary AS
SELECT 
    lc.content_type,
    lc.content_id,
    l.id AS language_id,
    l.code AS language_code,
    l.name AS language_name,
    COUNT(lc.id) AS field_count,
    MAX(lc.updated_at) AS last_updated
FROM 
    localized_contents lc
JOIN 
    languages l ON lc.language_id = l.id
WHERE 
    lc.is_active = TRUE
    AND l.is_active = TRUE
GROUP BY 
    lc.content_type, lc.content_id, l.id, l.code, l.name;

-- 创建索引
CREATE UNIQUE INDEX idx_mv_localized_contents_summary_content_language ON mv_localized_contents_summary(content_type, content_id, language_id);
CREATE INDEX idx_mv_localized_contents_summary_content_type ON mv_localized_contents_summary(content_type);
CREATE INDEX idx_mv_localized_contents_summary_content_id ON mv_localized_contents_summary(content_id);
CREATE INDEX idx_mv_localized_contents_summary_language_code ON mv_localized_contents_summary(language_code);

-- 创建刷新函数
CREATE OR REPLACE FUNCTION refresh_mv_localized_contents_summary()
RETURNS VOID AS $$
BEGIN
    REFRESH MATERIALIZED VIEW mv_localized_contents_summary;
END;
$$ LANGUAGE plpgsql;
```

### 4.2 优化复杂查询

#### 4.2.1 创建多语言内容查询函数

```sql
-- 获取实体的多语言内容
CREATE OR REPLACE FUNCTION get_entity_localized_contents(
    content_type VARCHAR, 
    content_id BIGINT, 
    language_id INTEGER,
    fallback_language_id INTEGER DEFAULT NULL
)
RETURNS TABLE(field_name VARCHAR(50), field_value TEXT, field_value_json JSONB) AS $$
BEGIN
    -- 尝试获取指定语言的内容
    RETURN QUERY
    SELECT 
        lc.field_name,
        lc.field_value,
        lc.field_value_json
    FROM 
        localized_contents lc
    WHERE 
        lc.content_type = content_type
        AND lc.content_id = content_id
        AND lc.language_id = language_id
        AND lc.is_active = TRUE;
    
    -- 如果没有找到且提供了回退语言，尝试获取回退语言的内容
    IF NOT FOUND AND fallback_language_id IS NOT NULL THEN
        RETURN QUERY
        SELECT 
            lc.field_name,
            lc.field_value,
            lc.field_value_json
        FROM 
            localized_contents lc
        WHERE 
            lc.content_type = content_type
            AND lc.content_id = content_id
            AND lc.language_id = fallback_language_id
            AND lc.is_active = TRUE;
    END IF;
END;
$$ LANGUAGE plpgsql;

-- 获取实体的多语言内容（JSON格式）
CREATE OR REPLACE FUNCTION get_entity_localized_contents_json(
    content_type VARCHAR, 
    content_id BIGINT, 
    language_id INTEGER,
    fallback_language_id INTEGER DEFAULT NULL
)
RETURNS JSONB AS $$
DECLARE
    result JSONB := '{}'::JSONB;
BEGIN
    -- 尝试获取指定语言的内容
    SELECT 
        jsonb_object_agg(field_name, COALESCE(field_value_json::JSONB, to_jsonb(field_value)))
    INTO 
        result
    FROM 
        localized_contents lc
    WHERE 
        lc.content_type = content_type
        AND lc.content_id = content_id
        AND lc.language_id = language_id
        AND lc.is_active = TRUE;
    
    -- 如果没有找到且提供了回退语言，尝试获取回退语言的内容
    IF result IS NULL AND fallback_language_id IS NOT NULL THEN
        SELECT 
            jsonb_object_agg(field_name, COALESCE(field_value_json::JSONB, to_jsonb(field_value)))
        INTO 
            result
        FROM 
            localized_contents lc
        WHERE 
            lc.content_type = content_type
            AND lc.content_id = content_id
            AND lc.language_id = fallback_language_id
            AND lc.is_active = TRUE;
    END IF;
    
    RETURN COALESCE(result, '{}'::JSONB);
END;
$$ LANGUAGE plpgsql;
```

#### 4.2.2 创建翻译查询函数

```sql
-- 获取嵌套翻译（JSON格式）
CREATE OR REPLACE FUNCTION get_nested_translations(
    language_id INTEGER,
    namespace VARCHAR DEFAULT 'common',
    fallback_language_id INTEGER DEFAULT NULL
)
RETURNS JSONB AS $$
DECLARE
    result JSONB := '{}'::JSONB;
BEGIN
    -- 递归构建嵌套JSON对象
    WITH RECURSIVE translation_tree AS (
        SELECT 
            key_path,
            value,
            ARRAY[key_path] AS path_array
        FROM 
            nested_translations
        WHERE 
            language_id = language_id
            AND namespace = namespace
            AND is_active = TRUE
            AND (key_path NOT LIKE '%.%' OR key_path ~ '^[^.]+\.[^.]+$')
        
        UNION ALL
        
        SELECT 
            nt.key_path,
            nt.value,
            tt.path_array || nt.key_path
        FROM 
            nested_translations nt
        JOIN 
            translation_tree tt ON nt.key_path LIKE tt.key_path || '.%'
        WHERE 
            nt.language_id = language_id
            AND nt.namespace = namespace
            AND nt.is_active = TRUE
            AND nt.key_path ~ '^[^.]+\.[^.]+$'
            AND NOT EXISTS (
                SELECT 1 FROM translation_tree 
                WHERE nt.key_path = ANY(path_array)
            )
    )
    SELECT 
        jsonb_object_agg(
            REPLACE(key_path, '.', '_'),  -- 简单处理，实际应用中需要更复杂的逻辑
            value
        )
    INTO 
        result
    FROM 
        translation_tree;
    
    -- 如果没有找到且提供了回退语言，尝试获取回退语言的翻译
    IF result IS NULL AND fallback_language_id IS NOT NULL THEN
        WITH RECURSIVE translation_tree AS (
            SELECT 
                key_path,
                value,
                ARRAY[key_path] AS path_array
            FROM 
                nested_translations
            WHERE 
                language_id = fallback_language_id
                AND namespace = namespace
                AND is_active = TRUE
                AND (key_path NOT LIKE '%.%' OR key_path ~ '^[^.]+\.[^.]+$')
            
            UNION ALL
            
            SELECT 
                nt.key_path,
                nt.value,
                tt.path_array || nt.key_path
            FROM 
                nested_translations nt
            JOIN 
                translation_tree tt ON nt.key_path LIKE tt.key_path || '.%'
            WHERE 
                nt.language_id = fallback_language_id
                AND nt.namespace = namespace
                AND nt.is_active = TRUE
                AND nt.key_path ~ '^[^.]+\.[^.]+$'
                AND NOT EXISTS (
                    SELECT 1 FROM translation_tree 
                    WHERE nt.key_path = ANY(path_array)
                )
        )
        SELECT 
            jsonb_object_agg(
                REPLACE(key_path, '.', '_'),  -- 简单处理，实际应用中需要更复杂的逻辑
                value
            )
        INTO 
            result
        FROM 
            translation_tree;
    END IF;
    
    RETURN COALESCE(result, '{}'::JSONB);
END;
$$ LANGUAGE plpgsql;
```

## 5. 总结

### 5.1 现有数据库设计评估

现有的国际化数据库设计具有以下优点：

1. **表结构清晰**：用户语言设置、翻译和多语言内容分离存储，每个表职责明确。
2. **索引设计合理**：为常用查询字段创建了索引，复合索引优化了多条件查询性能。
3. **数据完整性保障**：使用唯一约束防止数据重复，使用外键约束确保数据一致性。
4. **扩展性好**：支持动态添加新语言，支持动态添加新的翻译键。
5. **时间戳管理**：记录创建和更新时间，使用触发器自动更新时间戳。

但同时也存在以下缺点：

1. **查询性能问题**：多语言内容查询需要多次连接或子查询，复杂的多语言查询可能性能较差。
2. **数据冗余**：翻译表和多语言内容表有部分功能重叠，没有充分利用PostgreSQL的JSONB字段。
3. **缺少版本控制**：翻译内容没有版本管理，无法追踪翻译历史变更。
4. **缺少缓存机制**：没有设计翻译内容的缓存策略，频繁查询可能影响数据库性能。
5. **缺少数据验证**：没有在数据库层面验证语言代码有效性，没有验证翻译内容的完整性。

### 5.2 改进建议总结

针对现有数据库设计的不足，提出了以下改进建议：

1. **优化表结构设计**：
   - 引入语言表，统一管理语言信息
   - 优化用户语言设置表，支持多语言和优先级
   - 优化翻译表，增加版本控制和历史记录
   - 优化多语言内容表，增加JSONB字段支持

2. **引入JSONB字段支持复杂多语言结构**：
   - 创建复杂多语言内容表，使用JSONB存储复杂结构
   - 创建嵌套翻译表，支持嵌套键值对结构
   - 利用JSONB索引提高查询性能

3. **实现缓存机制**：
   - 创建翻译缓存表，提高频繁查询的性能
   - 实现缓存刷新触发器，确保缓存数据的一致性

4. **实现数据验证和完整性检查**：
   - 创建翻译完整性检查函数，监控翻译完成度
   - 创建数据验证触发器，确保数据有效性

5. **优化复杂查询**：
   - 创建物化视图，提高复杂查询的性能
   - 创建多语言内容查询函数，简化应用层代码
   - 创建翻译查询函数，支持嵌套结构查询

通过这些改进，可以显著提高数据库设计的多语言存储能力，满足系统对国际化功能的需求，同时提高查询性能和数据完整性。
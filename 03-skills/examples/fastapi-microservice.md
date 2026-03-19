# 使用场景：使用 Python FastAPI 快速构建微服务

这个示例展示如何结合使用 ECC 的 Python 技能和代理来快速构建一个高性能的 FastAPI 微服务。

## 场景描述

**任务**: 构建一个用户认证和授权的微服务

**功能要求**:
- 用户注册和登录
- JWT Token 认证
- OAuth 2.0 社交登录
- 密码重置
- 邮件验证
- 速率限制
- 完整的测试覆盖
- OpenAPI 文档

**技术栈**: FastAPI + Python 3.11 + PostgreSQL + Redis + SQLAlchemy

## ECC 辅助开发流程

### 步骤 1: 项目初始化

```
用户: 创建一个新的 FastAPI 项目，包含用户认证功能

ECC 检测到:
- 项目类型: Python (requirements.txt 或 pyproject.toml)
- 框架: FastAPI (main.py 或 app.py)
- 激活技能: python-patterns, python-testing, backend-patterns
```

根据 `python-patterns` 和 `backend-patterns` 技能的建议：

```python
# 项目结构
fastapi-auth-service/
├── app/
│   ├── __init__.py
│   ├── main.py                 # FastAPI 应用入口
│   ├── config.py               # 配置管理
│   ├── dependencies.py         # 依赖注入
│   ├── core/
│   │   ├── __init__.py
│   │   ├── security.py        # JWT 和密码处理
│   │   ├── email.py           # 邮件发送
│   │   └── cache.py           # Redis 缓存
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user.py            # SQLAlchemy 模型
│   │   └── oauth.py           # OAuth 模型
│   ├── schemas/
│   │   ├── __init__.py
│   │   ├── user.py            # Pydantic schemas
│   │   └── auth.py            # 认证 schemas
│   ├── crud/
│   │   ├── __init__.py
│   │   ├── user.py            # CRUD 操作
│   │   └── oauth.py           # OAuth CRUD
│   ├── api/
│   │   ├── __init__.py
│   │   ├── deps.py            # FastAPI 依赖
│   │   └── v1/
│   │       ├── __init__.py
│   │       ├── endpoints/
│   │       │   ├── __init__.py
│   │       │   ├── auth.py     # 认证端点
│   │       │   ├── users.py    # 用户端点
│   │       │   └── oauth.py    # OAuth 端点
│   │       └── api.py          # API 路由
│   └── tests/
│       ├── __init__.py
│       ├── conftest.py         # Pytest 配置
│       ├── test_auth.py
│       ├── test_users.py
│       └── test_oauth.py
├── alembic/                    # 数据库迁移
├── scripts/
│   └── init_db.py
├── requirements.txt
├── pyproject.toml
├── Dockerfile
├── docker-compose.yml
└── .env.example
```

---

### 步骤 2: 核心模型和 Schema

根据 `python-patterns` 技能（PEP 8、类型注解、Pydantic）：

```python
# app/models/user.py
from typing import Optional
from datetime import datetime
from sqlalchemy import Column, Integer, String, Boolean, DateTime, Index
from sqlalchemy.orm import relationship
from app.core.database import Base


class User(Base):
    """用户模型"""
    __tablename__ = "users"

    id = Column(Integer, primary_key=True, index=True)
    email = Column(String, unique=True, index=True, nullable=False)
    username = Column(String, unique=True, index=True, nullable=False)
    hashed_password = Column(String, nullable=True)  # OAuth 用户可以为空
    full_name = Column(String)
    is_active = Column(Boolean, default=True)
    is_superuser = Column(Boolean, default=False)
    is_verified = Column(Boolean, default=False)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

    # 关系
    oauth_accounts = relationship("OAuthAccount", back_populates="user")

    __table_args__ = (
        Index('ix_users_email_lower', 'email'),  # 不区分大小写索引
        Index('ix_users_username_lower', 'username'),
    )
```

```python
# app/schemas/user.py
from typing import Optional
from datetime import datetime
from pydantic import BaseModel, EmailStr, Field, validator


class UserBase(BaseModel):
    """用户基础 Schema"""
    email: EmailStr
    username: str = Field(..., min_length=3, max_length=50)
    full_name: Optional[str] = None


class UserCreate(UserBase):
    """用户注册 Schema"""
    password: str = Field(..., min_length=8, max_length=100)

    @validator('password')
    def validate_password_strength(cls, v):
        if not any(c.isupper() for c in v):
            raise ValueError('密码必须包含至少一个大写字母')
        if not any(c.islower() for c in v):
            raise ValueError('密码必须包含至少一个小写字母')
        if not any(c.isdigit() for c in v):
            raise ValueError('密码必须包含至少一个数字')
        return v


class UserUpdate(BaseModel):
    """用户更新 Schema"""
    full_name: Optional[str] = None
    email: Optional[EmailStr] = None
    password: Optional[str] = None


class UserInDB(UserBase):
    """数据库中的用户 Schema"""
    id: int
    is_active: bool
    is_superuser: bool
    is_verified: bool
    created_at: datetime
    updated_at: datetime

    class Config:
        from_attributes = True  # 从 ORM 模型创建


class User(UserInDB):
    """用户响应 Schema（不包含敏感信息）"""
    pass


class UserWithToken(User):
    """用户和 Token Schema"""
    access_token: str
    token_type: str = "bearer"
```

**python-patterns 技能提供的建议**:
- ✅ 使用 Pydantic 进行数据验证
- ✅ 添加类型注解提高代码可读性
- ✅ 使用 Field 定义约束
- ✅ 使用 validator 添加自定义验证
- ✅ 遵循 PEP 8 命名约定

---

### 步骤 3: CRUD 操作

根据 `python-patterns` 和 `backend-patterns` 技能：

```python
# app/crud/user.py
from typing import Optional
from sqlalchemy.orm import Session
from app.core.security import get_password_hash, verify_password
from app.models.user import User
from app.schemas.user import UserCreate, UserUpdate


def get_user(db: Session, user_id: int) -> Optional[User]:
    """根据 ID 获取用户"""
    return db.query(User).filter(User.id == user_id).first()


def get_user_by_email(db: Session, email: str) -> Optional[User]:
    """根据邮箱获取用户（不区分大小写）"""
    return db.query(User).filter(
        func.lower(User.email) == func.lower(email)
    ).first()


def get_user_by_username(db: Session, username: str) -> Optional[User]:
    """根据用户名获取用户（不区分大小写）"""
    return db.query(User).filter(
        func.lower(User.username) == func.lower(username)
    ).first()


def get_users(db: Session, skip: int = 0, limit: int = 100) -> list[User]:
    """获取用户列表（分页）"""
    return db.query(User).offset(skip).limit(limit).all()


def create_user(db: Session, user: UserCreate) -> User:
    """创建新用户"""
    # 检查查邮箱和用户名是否已存在
    existing_user = get_user_by_email(db, user.email)
    if existing_user:
        raise ValueError("邮箱已被使用")

    existing_user = get_user_by_username(db, user.username)
    if existing_user:
        raise ValueError("用户名已被使用")

    # 创建用户
    hashed_password = get_password_hash(user.password)
    db_user = User(
        email=user.email,
        username=user.username,
        hashed_password=hashed_password,
        full_name=user.full_name,
    )
    db.add(db_user)
    db.commit()
    db.refresh(db_user)
    return db_user


def update_user(db: Session, db_user: User, user_update: UserUpdate) -> User:
    """更新用户"""
    update_data = user_update.dict(exclude_unset=True)

    for field, value in update_data.items():
        if field == "password":
            # 如果更新密码，需要哈希
            setattr(db_user, field, get_password_hash(value))
        else:
            setattr(db_user, field, value)

    db.commit()
    db.refresh(db_user)
    return db_user


def delete_user(db: Session, user_id: int) -> bool:
    """删除用户"""
    db_user = get_user(db, user_id)
    if db_user:
        db.delete(db_user)
        db.commit()
        return True
    return False
```

**backend-patterns 技能提供的建议**:
- ✅ 使用 Repository Pattern 封装数据访问
- ✅ 添加输入验证
- ✅ 处理重复数据
- ✅ 使用类型注解
- ✅ 添加文档字符串

---

### 步骤 4: FastAPI 端点

根据 `python-patterns` 和 `backend-patterns` 技能：

```python
# app/api/v1/endpoints/users.py
from typing import List
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.orm import Session

from app.api.deps import get_current_active_user, get_db
from app.crud.user import (
    get_user,
    get_users,
    create_user,
    update_user,
    delete_user,
)
from app.models.user import User
from app.schemas.user import User, UserCreate, UserUpdate


router = APIRouter()


@router.post("/", response_model=User, status_code=status.HTTP_201_CREATED)
def create_user_endpoint(
    *,
    db: Session = Depends(get_db),
    user_in: UserCreate,
) -> User:
    """
    创建新用户

    - **email**: 用户邮箱（必须唯一）
    - **username**: 用户名（必须唯一）
    - **password**: 密码（至少8个字符，包含大小写和数字）
    - **full_name**: 用户全名（可选）
    """
    try:
        user = create_user(db=db, user=user_in)
        return user
    except ValueError as e:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail=str(e),
        )


@router.get("/", response_model=List[User])
def read_users(
    db: Session = Depends(get_db),
    skip: int = 0,
    limit: int = 100,
) -> List[User]:
    """
    获取用户列表（分页）

    - **skip**: 跳过的记录数（默认：0）
    - **limit**: 返回的记录数（默认：100）
    """
    users = get_users(db, skip=skip, limit=limit)
    return users


@router.get("/{user_id}", response_model=User)
def read_user(
    user_id: int,
    db: Session = Depends(get_db),
) -> User:
    """
    获取特定用户

    - **user_id**: 用户 ID
    """
    user = get_user(db, user_id=user_id)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="用户不存在",
        )
    return user


@router.put("/{user_id}", response_model=User)
def update_user_endpoint(
    *,
    user_id: int,
    user_in: UserUpdate,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_active_user),
) -> User:
    """
    更新用户信息

    - **user_id**: 要更新的用户 ID
    - 用户只能更新自己的信息（除非是超级用户）
    """
    # 验证权限
    if user_id != current_user.id and not current_user.is_superuser:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="无权更新此用户信息",
        )

    db_user = get_user(db, user_id=user_id)
    if not db_user:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="用户不存在",
        )

    try:
        user = update_user(db=db, db_user=db_user, user_update=user_in)
        return user
    except ValueError as e:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail=str(e),
        )


@router.delete("/{user_id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_user_endpoint(
    *,
    user_id: int,
    db: Session = Depends(get_db),
    current_user: User = Depends(get_current_active_user),
):
    """
    删除用户

    - **user_id**: 要删除的用户 ID
    - 用户只能删除自己（除非是超级用户）
    """
    # 验证权限
    if user_id != current_user.id and not current_user.is_superuser:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="无权删除此用户",
        )

    success = delete_user(db, user_id=user_id)
    if not success:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="用户不存在",
        )
```

**python-patterns 技能提供的建议**:
- ✅ 使用 FastAPI 依赖注入
- ✅ 添加 OpenAPI 文档
- ✅ 返回适当的 HTTP 状态码
- ✅ 使用类型注解
- ✅ 添加详细的文档字符串

---

### 步骤 5: 使用 python-testing 技能编写测试

```
用户: 我需要为用户 CRUD 操作编写测试

ECC 自动激活: python-testing 技能
```

根据 `python-testing` 技能（pytest、fixtures、测试模式）：

```python
# app/tests/test_users.py
import pytest
from fastapi.testclient import TestClient
from sqlalchemy.orm import Session

from app.main import app
from app.api.deps import get_db
from app.core.database import get_engine, TestingSessionLocal
from app.crud.user import create_user, get_user_by_email
from app.schemas.user import UserCreate


# Pytest fixture
@pytest.fixture
def db():
    """创建测试数据库会话"""
    engine = get_engine()
    TestingSessionLocal.configure(bind=engine)
    db = TestingSessionLocal()
    try:
        yield db
    finally:
        db.close()


@pytest.fixture
def client(db):
    """创建测试客户端"""
    def override_get_db():
        try:
            yield db
        finally:
            pass

    app.dependency_overrides[get_db] = override_get_db
    with TestClient(app) as test_client:
        yield test_client
    app.dependency_overrides = {}


@pytest.fixture
def test_user(db: Session):
    """创建测试用户"""
    user_in = UserCreate(
        email="test@example.com",
        username="testuser",
        password="TestPass123",
        full_name="Test User",
    )
    user = create_user(db=db, user=user_in)
    return user


class TestCreateUser:
    """测试创建用户"""

    def test_create_user_success(self, client: TestClient):
        """测试成功创建用户"""
        response = client.post(
            "/api/v1/users/",
            json={
                "email": "new@example.com",
                "username": "newuser",
                "password": "NewPass123",
                "full_name": "New User",
            },
        )
        assert response.status_code == 201
        data = response.json()
        "id" in data
        assert data["email"] == "new@example.com"
        assert data["username"] == "newuser"
        assert data["full_name"] == "New User"
        assert "password" not in data  # 密码不应返回

    def test_create_user_duplicate_email(self, client: TestClient, test_user):
        """测试重复邮箱"""
        response = client.post(
            "/api/v1/users/",
            json={
                "email": test_user.email,  # 重复邮箱
                "username": "different",
                "password": "TestPass123",
            },
        )
        assert response.status_code == 400
        assert "邮箱已被使用" in response.json()["detail"]

    def test_create_user_weak_password(self, client: TestClient):
        """测试弱密码"""
        response = client.post(
            "/api/v1/users/",
            json={
                "email": "weak@example.com",
                "username": "weakuser",
                "password": "weak",  # 弱密码
            },
        )
        assert response.status_code == 422  # Validation error

    def test_create_user_invalid_email(self, client: TestClient):
        """测试无效邮箱"""
        response = client
.post(
            "/api/v1/users/",
            json={
                "email": "invalid-email",  # 无效邮箱
                "username": "test",
                "password": "TestPass123",
            },
        )
        assert response.status_code == 422


class TestReadUsers:
    """测试读取用户"""

    def test_read_users_empty(self, client: TestClient):
        """测试空用户列表"""
        response = client.get("/api/v1/users/")
        assert response.status_code == 200
        assert response.json() == []

    def test_read_users_with_data(self, client: TestClient, test_user):
        """测试有数据的用户列表"""
        response = client.get("/api/v1/users/")
        assert response.status_code == 200
        users = response.json()
        assert len(users) == 1
        assert users[0]["email"] == test_user.email

    def test_read_users_pagination(self, client: TestClient, db: Session):
        """测试分页"""
        # 创建多个用户
        for i in range(5):
            user_in = UserCreate(
                email=f"user{i}@example.com",
                username=f"user{i}",
                password="TestPass123",
            )
            create_user(db=db, user=user_in)

        # 测试分页
        response = client.get("/api/v1/users/?skip=0&limit=2")
        assert response.status_code == 200
        users = response.json()
        assert len(users) == 2


class TestReadUser:
    """测试读取单个用户"""

    def test_read_user_success(self, client: TestClient, test_user):
        """测试成功读取用户"""
        response = client.get(f"/api/v1/users/{test_user.id}")
        assert response.status_code == 200
        data = response.json()
        assert data["id"] == test_user.id
        assert data["email"] == test_user.email

    def test_read_user_not_found(self, client: TestClient):
        """测试用户不存在"""
        response = client.get("/api/v1/users/999999")
        assert response.status_code == 404
        assert "不存在" in response.json()["detail"]


class TestUpdateUser:
    """测试更新用户"""

    def test_update_user_success(self, client: TestClient, test_user, db: Session):
        """测试成功更新用户"""
        response = client.put(
            f"/api/v1/users/{test_user.id}",
            json={
                "full_name": "Updated Name",
            },
            headers={"Authorization": f"Bearer {test_user.access_token}"},
        )
        assert response.status_code == 200
        data = response.json()
        assert data["full_name"] == "Updated Name"

    def test_update_user_unauthorized(self, client: TestClient, test_user):
        """测试未授权更新"""
        # 创建另一个用户
        other_user = create_user(
            db=db,
            user=UserCreate(
                email="other@example.com",
                username="other",
                password="TestPass123",
            ),
        )

        # 尝试用 test_user 的 token 更新 other_user
        response = client.put(
            f"/api/v1/users/{other_user.id}",
            json={"full_name": "Hacked"},
            headers={"Authorization": f"Bearer {test_user.access_token}"},
        )
        assert response.status_code == 403
        assert "无权" in response.json()["detail"]


class TestDeleteUser:
    """测试删除用户"""

    def test_delete_user_success(self, client: TestClient, test_user):
        """测试成功删除用户"""
        response = client.delete(
            f"/api/v1/users/{test_user.id}",
            headers={"Authorization": f"Bearer {test_user.access_token}"},
        )
        assert response.status_code == 204

    def test_delete_user_unauthorized(self, client: TestClient, test_user, db: Session):
        """测试未授权删除"""
        # 创建另一个用户
        other_user = create_user(
            db=db,
            user=UserCreate(
                email="other@example.com",
                username="other",
                password="TestPass123",
            ),
        )

        # 尝试用 test_user 的 token 删除 other_user
        response = client.delete(
            f"/api/v1/users/{other_user.id}",
            headers={"Authorization": f"Bearer {test_user.access_token}"},
        )
"        assert response.status_code == 403
        assert "无权" in response.json()["detail"]
```

**python-testing 技能提供的建议**:
- ✅ 使用 pytest 作为测试框架
- ✅ 使用 fixtures 管理测试数据
- ✅ 使用 TestClient 测试 FastAPI 端点
- ✅ 使用 AAA 模式（Arrange-Act-Assert）
- ✅ 测试成功和失败场景
- ✅ 测试边界条件

---

### 步骤 6: Docker 化

根据 `docker-patterns` 技能：

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

# 安装系统依赖
RUN apt-get update && apt-get install -y \
    gcc \
    postgresql-client \
    && rm -rf /var/lib/apt/lists/*

# 复制依赖文件
COPY requirements.txt .

# 安装 Python 依赖
RUN pip install --no-cache-dir -r requirements.txt

# 复制应用代码
COPY . .

# 暴露端口
EXPOSE 8000

# 运行命令
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  api:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/fastapi_auth
      - REDIS_URL=redis://redis:6379/0
      - SECRET_KEY=your-secret-key-here
    depends_on:
      - db
      - redis
    volumes:
      - .:/app
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=fastapi_auth
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

  tests:
    build: .
    environment:
      - DATABASE_URL=postgresql://user:password@test-db:5432/fastapi_auth_test
      - REDIS_URL=redis://redis:6379/0
      - SECRET_KEY=test-secret-key
    depends_on:
      - test-db
      - redis
    command: pytest -v

  test-db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=fastapi_auth_test

volumes:
  postgres_data:
  redis_data:
```

**docker-patterns 技能提供的建议**:
- ✅ 使用多阶段构建优化镜像大小
- ✅ 使用 docker-compose 管理多服务
- ✅ 分离开发和测试环境
- ✅ 使用环境变量管理配置
- ✅ 使用数据卷持久化数据

---

### 步骤 7: 性能优化

根据 `python-patterns` 技能：

```python
# app/core/cache.py
from typing import Optional, Any
import json
import redis.asyncio as redis
from app.config import settings


class RedisCache:
    """Redis 缓存客户端"""

    def __init__(self, redis_url: str):
        self.redis = redis.from_url(redis_url, encoding="utf-8", decode_responses=True)

    async def get(self, key: str) -> Optional[Any]:
        """从缓存获取值"""
        value = await self.redis.get(key)
        if value is None:
            return None
        return json.loads(value)

    async def set(self, key: str, value: Any, ttl: int = 3600) -> None:
        """设置缓存值"""
        await self.redis.setex(key, ttl, json.dumps(value))

    async def delete(self, key: str) -> None:
        """删除缓存值"""
        await self.redis.delete(key)

    async def exists(self, key: str) -> bool:
        """检查键是否存在"""
        return await self.redis.exists(key) > 0


# app/api/deps.py
from fastapi import Depends
from app.core.cache import RedisCache
from app.config import settings


# 创建 Redis 缓存实例
cache = RedisCache(settings.REDIS_URL)


async def get_cached_user(user_id: int) -> Optional[dict]:
    """从缓存获取用户"""
    cache_key = f"user:{user_id}"
    return await cache.get(cache_key)


async def set_cached_user(user_id: int, user_data: dict, ttl: int = 3600) -> None:
    """缓存用户数据"""
    cache_key = f"user:{user_id}"
    await cache.set(cache_key, user_data, ttl=ttl)
```

```python
# 在端点中使用缓存
@router.get("/{user_id}", response_model=User)
async def read_user(
    user_id: int,
    db: Session = Depends(get_db),
) -> User:
    """
    获取特定用户（带缓存）
    """
    # 尝试从缓存获取
    cached_user = await get_cached_user(user_id)
    if cached_user:
        return User(**cached_user)

    # 从数据库获取
    user = get_user(db, user_id=user_id)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="用户不存在",
        )

    # 缓存用户数据
    user_data = {
        "id": user.id,
        "email": user.email,
        "username": user.username,
        "full_name": user.full_name,
        "is_active": user.is_active,
        "is_superuser": user.is_superuser,
        "is_verified": user.is_verified,
        "created_at": user.created_at.isoformat(),
        "updated_at": user.updated_at.iso(),
    }
    await set_cached_user(user_id, user_data)

    return user
```

---

## 测试运行和覆盖率

```bash
# 运行所有测试
pytest -v

# 运行特定测试文件
pytest app/tests/test_users.py -v

# 运行特定测试类
pytest app/tests/test_users.py::TestCreateUser -v

# 运行特定测试方法
pytest app/tests/test_users.py::TestCreateUser::test_create_user_success -v

# 生成覆盖率报告
pytest --cov=app --cov-report=html --cov-report=term

# 使用 pytest-xdist 并行运行测试
pytest -n auto

# 使用 pytest-bdd 运行 BDD 测试
behave app/tests/features/

# 使用 pytest-prof 进行性能分析
pytest --profile=default app/tests/
```

---

## ECC 技能和代理的协作

```
用户: 创建 FastAPI 用户认证服务
    ↓
python-patterns (项目结构、编码规范）
    ↓
backend-patterns (架构设计、API 设计）
    ↓
python-testing (测试策略、测试模板）
    ↓
docker-patterns (容器化、编排）
    ↓
code-reviewer (自动审查)
    ↓
python-reviewer (Python 特定审查)
```

---

## 关键优势

| 传统开发 | 使用 ECC |
|----------|---------|
| 手动设计项目结构 | python-patterns 自动生成结构 |
| 查阅文档编写代码 | backend-patterns 实时提供建议 |
| 手动编写测试 | python-testing 生成测试模板 |
| 手动配置 Docker | docker-patterns 生成配置 |
| 提交后人工审查 | code-reviewer 自动审查 |
| 迭代多次 | 一次完成高质量代码 |

---

## 扩展场景

### 添加 OAuth 2.0 社交登录

```
用户: 添加 Google 和 GitHub 社交登录

ECC 使用 skills:
- python-patterns: OAuth 实现模式
- backend-patterns: JWT 集成
```

### 添加速率限制

```
用户: 添加 API 速率限制

ECC 使用 skills:
- backend-patterns: 速率限制模式
- python-patterns: Redis 集成
```

### 添加邮件验证

```
用户: 添加邮箱验证流程

ECC 使用 skills:
- backend-patterns: 邮件服务模式
- python-patterns: 异步任务处理
```

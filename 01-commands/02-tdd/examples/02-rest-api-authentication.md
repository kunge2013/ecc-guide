# 场景 2: REST API 开发 - 用户认证接口

这个场景展示了 TDD 如何帮助设计清晰、可靠的 API 接口。

## 需求

实现用户注册和登录的 REST API：
- 用户注册：用户名、邮箱、密码
- 密码强度验证
- 邮箱格式验证
- 防止重复注册
- JWT Token 返回

## 测试驱动开发过程

### Step 1: 写测试（RED）- test_auth_api.py

```python
import pytest
import json
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)

def test_register_user_success():
    """测试成功注册新用户"""
    response = client.post("/api/auth/register", json={
        "username": "testuser",
        "email": "test@example.com",
        "password": "SecurePass123!"
    })

    assert response.status_code == 201
    data = response.json()
    assert "user_id" in data
    assert data["username"] == "testuser"
    assert "password" not in data  # 不返回密码
    assert "token" in data  # 返回 JWT token

def test_register_duplicate_email():
    """测试重复邮箱注册"""
    # 第一次注册
    client.post("/api/auth/register", json={
        "username": "user1",
        "email": "duplicate@example.com",
        "password": "SecurePass123!"
    })

    # 第二次注册相同邮箱
    response = client.post("/api/auth/register", json={
        "username": "user2",
        "email": "duplicate@example.com",
        "password": "SecurePass123!"
    })

    assert response.status_code == 409  # Conflict
    data = response.json()
    assert "email" in data["detail"].lower()

def test_register_weak_password():
    """测试弱密码拒绝"""
    response = client.post("/api/auth/register", json={
        "username": "testuser",
        "email": "test@example.com",
        "password": "123"  # 太短
    })

    assert response.status_code == 400
    data = response.json()
    assert "password" in data["detail"].lower()

def test_register_invalid_email():
    """测试无效邮箱格式"""
    response = client.post("/api/auth/register", json={
        "username": "testuser",
        "email": "invalid-email",
        "password": "SecurePass123!"
    })

    assert response.status_code == 400
    data = response.json()
    assert "email" in data["detail"].lower()

def test_login_success():
    """测试成功登录"""
    # 先注册
    client.post("/api/auth/register", json={
        "username": "loginuser",
        "email": "login@example.com",
        "password": "LoginPass123!"
    })

    # 登录
    response = client.post("/api/auth/login", json={
        "email": "login@example.com",
        "password": "LoginPass123!"
    })

    assert response.status_code == 200
    data = response.json()
    assert "token" in data
    assert "user" in data

def test_login_wrong_password():
    """测试密码错误"""
    response = client.post("/api/auth/login", json={
        "email": "login@example.com",
        "password": "WrongPassword!"
    })

    assert response.status_code == 401

def test_login_nonexistent_user():
    """测试不存在的用户"""
    response = client.post("/api/auth/login", json={
        "email": "nonexistent@example.com",
        "password": "AnyPassword!"
    })

    assert response.status_code == 404

def test_protected_endpoint_without_token():
    """测试未授权访问受保护端点"""
    response = client.get("/api/users/profile")
    assert response.status_code == 401

def test_protected_endpoint_with_token():
    """测试带 token 访问受保护端点"""
    # 注册并获取 token
    register_response = client.post("/api/auth/register", json={
        "username": "authuser",
        "email": "auth@example.com",
        "password": "AuthPass123!"
    })
    token = register_response.json()["token"]

    # 访问受保护端点
    response = client.get(
        "/api/users/profile",
        headers={"Authorization": f"Bearer {token}"}
    )

    assert response.status_code == 200
    data = response.json()
    assert data["username"] == "authuser"
```

### Step 2: 实现代码（GREEN）- auth.py

```python
from fastapi import FastAPI, HTTPException, Depends, status
from pydantic import BaseModel, EmailStr, validator
import jwt
import bcrypt
from datetime import datetime, timedelta
from typing import Optional

app = FastAPI()

# 模拟数据库
users_db = {}

class UserRegister(BaseModel):
    username: str
    email: EmailStr
    password: str

    @validator('password')
    def validate_password(cls, v):
        if len(v) < 8:
            raise ValueError('密码至少8个字符')
        if not any(c.isupper() for c in v):
            raise ValueError('密码必须包含大写字母')
        if not any(c.islower() for c in v):
            raise ValueError('密码必须包含小写字母')
        if not any(c.isdigit() for c in v):
            raise ValueError('密码必须包含数字')
        if not any(c in '!@#$%^&*' for c in v):
            raise ValueError('密码必须包含特殊字符')
        return v

class UserLogin(BaseModel):
    email: EmailStr
    password: str

class UserResponse(BaseModel):
    user_id: int
    username: str
    email: EmailStr
    token: str

def hash_password(password: str) -> str:
    return bcrypt.hashpw(password.encode(), bcrypt.gensalt()).decode()

def verify_password(password: str, hashed: str) -> bool:
    return bcrypt.checkpw(password.encode(), hashed.encode())

def create_jwt_token(user_id: int, username: str) -> str:
    payload = {
        'user_id': user_id,
        'username': username,
        'exp': datetime.utcnow() + timedelta(hours=24)
    }
    return jwt.encode(payload, 'secret-key', algorithm='HS256')

def verify_jwt_token(token: str):
    try:
        payload = jwt.decode(token, 'secret-key', algorithms=['HS256'])
        return payload
    except jwt.ExpiredSignatureError:
        raise HTTPException(status_code=401, detail="Token已过期")
    except jwt.InvalidTokenError:
        raise HTTPException(status_code=401, detail="无效的Token")

@app.post("/api/auth/register", status_code=201)
async def register(user: UserRegister):
    # 检查邮箱是否已存在
    for existing_user in users_db.values():
        if existing_user['email'] == user.email:
            raise HTTPException(
                status_code=409,
                detail="该邮箱已被注册"
            )

    # 创建用户
    user_id = len(users_db) + 1
    hashed_password = hash_password(user.password)
    token = create_jwt_token(user_id, user.username)

    users_db[user_id] = {
        'user_id': user_id,
        'username': user.username,
        'email': user.email,
        'password': hashed_password
    }

    return {
        'user_id': user_id,
        'username': user.username,
        'email': user.email,
        'token': token
    }

@app.post("/api/auth/login")
async def login(user: UserLogin):
    # 查找用户
    user_data = None
    for existing_user in users_db.values():
        if existing_user['email'] == user.email:
            user_data = existing_user
            break

    if not user_data:
        raise HTTPException(
            status_code=404,
            detail="用户不存在"
        )

    # 验证密码
    if not verify_password(user.password, user_data['password']):
        raise HTTPException(
            status_code=401,
            detail="密码错误"
        )

    # 生成新 token
    token = create_jwt_token(user_data['user_id'], user_data['username'])

    return {
        'token': token,
        'user': {
            'user_id': user_data['user_id'],
            'username': user_data['username'],
            'email': user_data['email']
        }
    }

async def get_current_user(authorization: str = Depends(lambda: None)):
    if not authorization:
        raise HTTPException(status_code=401, detail="未提供认证令牌")

    token = authorization.replace('Bearer ', '')
    payload = verify_jwt_token(token)
    return payload

@app.get("/api/users/profile")
async def get_profile(current_user = Depends(get_current_user)):
    user_id = current_user['user_id']
    if user_id not in users_db:
        raise HTTPException(status_code=404, detail="用户不存在")

    user_data = users_db[user_id]
    return {
        'user_id': user_data['user_id'],
        'username': user_data['username'],
        'email': user_data['email']
    }
```

### Step 3: 重构（REFACTOR）

提取密码验证器：

```python
class PasswordValidator:
    MIN_LENGTH = 8
    REQUIRED_SPECIAL = '!@#$%^&*'

    @classmethod
    def validate(cls, password: str) -> list[str]:
        errors = []

        if len(password) < cls.MIN_LENGTH:
            errors.append(f'密码至少{cls.MIN_LENGTH}个字符')

        if not any(c.isupper() for c in password):
            errors.append('密码必须包含大写字母')

        if not any(c.islower() for c in password):
            errors.append('密码必须包含小写字母')

        if not any(c.isdigit() for c in password):
            errors.append('密码必须包含数字')

        if not any(c in cls.REQUIRED_SPECIAL for c in password):
            errors.append('密码必须包含特殊字符')

        return errors

# 在 UserRegister 中使用
@validator('password')
def validate_password(cls, v):
    errors = PasswordValidator.validate(v)
    if errors:
        raise ValueError('; '.join(errors))
    return v
```

## 关键学习点

1. **API 契约优先设计** - 测试定义了清晰的 API 响应格式和错误码
2. **输入验证前置** - 在业务逻辑之前验证所有输入
3. **安全测试覆盖** - 密码验证、授权、Token 验证都被测试覆盖
4. **HTTP 状态码正确性** - 测试确保使用正确的状态码（201, 409, 401, 404）

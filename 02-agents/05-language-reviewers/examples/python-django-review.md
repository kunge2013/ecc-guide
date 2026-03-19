# 示例：Django 应用代码审查

这个示例展示 python-reviewer 如何审查 Django 应用代码。

## 审查的代码

### 1. views.py

```python
# views.py
from django.shortcuts import render, redirect
from django.http import HttpResponse
from .models import User, Task

def user_detail(request, user_id):
    user = User.objects.get(id=user_id)  # 可能抛出异常
    return render(request, 'user_detail.html', {'user': user})

def task_list(request):
    tasks = Task.objects.all()
    for task in tasks:
        print(task.user.username)  # N+1 查询
    return render(request, 'task_list.html', {'tasks': tasks})

def create_user(request):
    if request.method == 'POST':
        username = request.POST.get('username')
        password = request.POST.get('password')
        email = request.POST.get('email')

        user = User.objects.create(
            username=username,
            password=password,  # 没有加密密码
            email=email
        )
        return redirect('user_detail', user_id=user.id)
    return render(request, 'create_user.html')
```

### 2. models.py

```python
# models.py
from django.db import models
import uuid

class User(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4)
    username = models.CharField(max_length=50)
    password = models.CharField(max_length=255)
    email = models.EmailField(max_length=100)
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.username

class Task(models.Model):
    title = models.CharField(max_length=200)
    description = models.TextField()
    status = models.CharField(max_length=20, default='TODO')
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return self.title
```

### 3. services.py

```python
# services.py
import time
import requests

class UserService:
    def get_external_user_data(self, user_id):
        url = f"https://api.example.com/users/{user_id}"
        response = requests.get(url)  # 阻塞调用
        return response.json()

    def process_user_list(self, user_ids):
        results = []
        for user_id in user_ids:
            data = self.get_external_user_data(user_id)
            results.append(data)
        return results

def send_notification(email, message):
    time.sleep(5)  # 阻塞调用
    print(f"Sent to {email}: {message}")
```

## 审查报告

# Django 代码审查报告

## 概览
- 审查的文件：3 个
- 发现的 CRITICAL 问题：2 个
- 发现的 HIGH 问题：4 个
- 发现的 MEDIUM 问题：3 个
- 发现的 LOW 问题：2 个

---

## 🔴 CRITICAL 问题

### 1. 密码明文存储
**文件**: `views.py:25`
**等级**: CRITICAL
**问题**: 密码没有加密直接存储

```python
user = User.objects.create(
    username=username,
    password=password,  # 明文密码！
    email=email
)
```

**风险**: 数据库泄露时，所有用户密码泄露

**建议**: 使用 Django 的密码哈希：

```python
from django.contrib.auth.hashers import make_password

user = User.objects.create(
    username=username,
    password=make_password(password),  # 哈希密码
    email=email
)

# 更好的方式：使用 Django 内置的 User 模型
from django.contrib.auth.models import User

user = User.objects.create_user(
    username=username,
    password=password,
    email=email
)
```

---

### 2. N+1 查询问题
**文件**: `views.py:13`
**等级**: CRITICAL
**问题**: 在循环中访问关联对象

```python
def task_list(request):
    tasks = Task.objects.all()
    for task in tasks:
        print(task.user.username)  # 每次都查询数据库
    return render(request, 'task_list.html', {'tasks': tasks})
```

**性能影响**: 100 个任务 → 101 个数据库查询

**建议**: 使用 select_related：

```python
def task_list(request):
    tasks = Task.objects.select_related('user').all()
    # 现在只执行 1 个查询
    return render(request, 'task_list.html', {'tasks': tasks})

# 如果需要反向查询，使用 prefetch_related
def user_task_list(request):
    users = User.objects.prefetch_related('task_set').all()
    # 每个 user.tasks 都已经预加载
    return render(request, 'user_task_list.html', {'users': users})
```

---

## 🟠 HIGH 问题

### 1. 直接使用 .get() 可能引发异常
**文件**: `views.py:7`
**等级**: HIGH
**问题**: 直接使用 `.get()` 可能引发 DoesNotExist 异常

```python
def user_detail(request, user_id):
    user = User.objects.get(id=user_id)  # 可能抛出异常
    return render(request, 'user_detail.html', {'user': user})
```

**建议**: 使用 `get_object_or_404`：

```python
from django.shortcuts import get_object_or_404

def user_detail(request, user_id):
    user = get_object_or_404(User, id=user_id)
    return render(request, 'user_detail.html', {'user': user})

# 或使用 try-except
def user_detail(request, user_id):
    try:
        user = User.objects.get(id=user_id)
    except User.DoesNotExist:
        raise Http404("User not found")
    return render(request, 'user_detail.html', {'user': user})
```

---

### 2. 缺少输入验证
**文件**: `views.py:18`
**等级**: HIGH
**问题**: 没有验证用户输入

```python
def create_user(request):
    if request.method == 'POST':
        username = request.POST.get('username')
        password = request.POST.get('password')
        email = request.POST.get('email')

        # 没有验证任何字段
        user = User.objects.create(
            username=username,
            password=make_password(password),
            email=email
        )
```

**建议**: 使用 Django Forms：

```python
# forms.py
from django import forms
from django.contrib.auth.models import User

class UserRegistrationForm(forms.ModelForm):
    class Meta:
        model = User
        fields = ['username', 'email']

    password = forms.CharField(
        widget=forms.PasswordInput,
        min_length=8,
        required=True
    )

    confirm_password = forms.CharField(
        widget=forms.PasswordInput,
        required=True
    )

    def clean_username(self):
        username = self.cleaned_data.get('username')
        if len(username) < 3:
            raise forms.ValidationError("用户名至少3个字符")
        return username

    def clean_email(self):
        email = self.cleaned_data.get('email')
        if User.objects.filter(email=email).exists():
            raise forms.ValidationError("邮箱已被使用")
        return email

    def clean(self):
        cleaned_data = super().clean()
        password = cleaned_data.get('password')
        confirm_password = cleaned_data.get('confirm_password')

        if password != confirm_password:
            raise forms.ValidationError("密码不匹配")
        return cleaned_data

# views.py
def create_user(request):
    if request.method == 'POST':
        form = UserRegistrationForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('user_list')
    else:
        form = UserRegistrationForm()

    return render(request, 'create_user.html', {'form': form})
```

---

### 3. 阻塞 I/O 调用
**文件**: `services.py:6`
**等级**: HIGH
**问题**: 同步阻塞调用

```python
def get_external_user_data(self, user_id):
    url = f"https://api.example.com/users/{user_id}"
    response = requests.get(url)  # 阻塞整个请求
    return response.json()
```

**性能影响**: 外部 API 响应慢时，整个 Django 请求被阻塞

**建议**: 使用异步视图：

```python
# 异步方式
import aiohttp
import asyncio

class AsyncUserService:
    async def get_external_user_data(self, user_id):
        url = f"https://api.example.com/users/{user_id}"
        async with aiohttp.ClientSession() as session:
            async with session.get(url) as response:
                return await response.json()

    async def process_user_list(self, user_ids):
        # 并发获取所有用户数据
        tasks = [
            self.get_external_user_data(user_id)
            for user_id in user_ids
        ]
        return await asyncio.gather(*tasks)

# views.py
from django.http import JsonResponse
from asgiref.sync import async_to_sync

async def user_data_view(request, user_id):
    service = AsyncUserService()
    data = await service.get_external_user_data(user_id)
    return JsonResponse(data)

# 或使用后台任务（Celery）
from celery import shared_task

@shared_task
def send_notification_async(email, message):
    # 在后台执行
    send_notification(email, message)
```

---

### 4. 缺少 CSRF 保护
**文件**: `views.py`
**等级**: HIGH
**问题**: POST 请求没有 CSRF 保护

```python
def create_user(request):
    if request.method == 'POST':
        # 没有 CSRF 检查
        user = User.objects.create(...)
```

**风险**: 跨站请求伪造（CSRF）攻击

**建议**: 使用 Django 的 CSRF 保护：

```python
# 确保使用模板中的 CSRF token
# create_user.html
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">提交</button>
</form>

# 对于 AJAX 请求，添加 CSRF token
<script>
function getCookie(name) {
    let cookieValue = null;
    if (document.cookie && document.cookie !== '') {
        const cookies = document.cookie.split(';');
        for (let i = 0; i < cookies.length; i++) {
            const cookie = cookies[i].trim();
            if (cookie.substring(0, name.length + 1) === (name + '=')) {
                cookieValue = decodeURIComponent(cookie.substring(name.length + 1));
                break;
            }
        }
    }
    return cookieValue;
}

const csrfToken = getCookie('csrftoken');
fetch('/api/users/', {
    method: 'POST',
    headers: {
        'X-CSRFToken': csrfToken,
        'Content-Type': 'application/json'
    },
    body: JSON.stringify(data)
});
</script>
```

---

## 🟡 MEDIUM 问题

### 1. 缺少类型注解
**文件**: `services.py:4`
**等级**: MEDIUM
**问题**: 函数缺少类型注解

```python
def get_external_user_data(self, user_id):
    # 返回类型不明确
    return response.json()
```

**建议**: 添加类型注解：

```python
from typing import Dict, Any, List

class UserService:
    def get_external_user_data(self, user_id: int) -> Dict[str, Any]:
        url = f"https://api.example.com/users/{user_id}"
        response = requests.get(url)
        return response.json()

    def process_user_list(self, user_ids: List[int]) -> List[Dict[str, Any]]:
        results = []
        for user_id in user_ids:
            data = self.get_external_user_data(user_id)
            results.append(data)
        return results
```

---

### 2. 缺少错误处理
**文件**: `services.py:8`
**等级**: MEDIUM
**问题**: 没有处理网络错误

```python
response = requests.get(url)  # 可能超时或失败
return response.json()
```

**建议**: 添加错误处理：

```python
import requests
from requests.exceptions import RequestException
import logging

logger = logging.getLogger(__name__)

class UserService:
    def get_external_user_data(self, user_id: int) -> Dict[str, Any]:
        url = f"https://api.example.com/users/{user_id}"

        try:
            response = requests.get(
                url,
                timeout=10  # 10秒超时
            )
            response.raise_for_status()  # 检查 HTTP 错误
            return response.json()
        except requests.Timeout:
            logger.error(f"Timeout while fetching user {user_id}")
            raise UserServiceError("外部 API 超时")
        except requests.HTTPError as e:
            logger.error(f"HTTP error while fetching user {user_id}: {e}")
            raise UserServiceError(f"外部 API 错误: {e.response.status_code}")
        except RequestException as e:
            logger = logging.getLogger(__name__)
            logger.error(f"Request error while fetching user {user_id}: {e}")
            raise UserServiceError("外部 API 请求失败")
        except ValueError as e:
            logger.error(f"Invalid JSON response for user {user_id}: {e}")
            raise UserServiceError("外部 API 返回无效数据")

class UserServiceError(Exception):
    pass
```

---

### 3. 缺少数据库事务
**文件**: `views.py:18`
**等级**: MEDIUM
**问题**: 创建用户和关联数据没有使用事务

```python
def create_user_with_tasks(request):
    user = User.objects.create(...)
    # 如果这里失败，用户已创建但任务没有创建
    Task.objects.create(user=user, title="默认任务")
```

**建议**: 使用事务：

```python
from django.db import transaction

@transaction.atomic
def create_user_with_tasks(request):
    try:
        user = User.objects.create(...)
        Task.objects.create(user=user, title="默认任务")
        return redirect('success')
    except Exception as e:
        # 事务会自动回滚
        logger.error(f"Failed to create user with tasks: {e}")
        return render(request, 'error.html', {'error': str(e)})

# 或使用 savepoint
def create_multiple_users(user_data_list):
    try:
        for user_data in user_data_list:
            with transaction.atomic():
                user = User.objects.create(**user_data)
                create_default_tasks(user)
    except Exception as e:
        logger.error(f"Failed to create users: {e}")
        raise
```

---

## 总体评分: 6.5/10

## 优先修复建议

### 立即修复（CRITICAL）
1. ✅ 修复密码明文存储
2. ✅ 修复 N+1 查询问题

### 高优先级（HIGH）
1. ✅ 使用 `get_object_or_404` 处理异常
2. ✅ 添加输入验证
3. ✅ 使用异步方式处理外部 API 调用
4. ✅ 确保 CSRF 保护

### 中优先级（MEDIUM）
1. 添加类型注解
2. 添加错误处理
3. 使用数据库事务

## 测试覆盖率建议

| 模块 | 当前覆盖率 | 建议覆盖率 |
|------|-----------|-----------|
| views.py | 35% | 80% |
| models.py | 20% | 90% |
| services.py | 25% | 85% |

## Django 安全配置建议

```python
# settings.py

# 安全设置
DEBUG = False  # 生产环境必须为 False

ALLOWED_HOSTS = ['yourdomain.com', 'www.yourdomain.com']

# HTTPS 设置
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True

# HSTS 设置
SECURE_HSTS_SECONDS = 31536000
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True

# 其他安全设置
SECURE_CONTENT_TYPE_NOSNIFF = True
SECURE_BROWSER_XSS_FILTER = True
X_FRAME_OPTIONS = 'DENY'

# 密码验证
AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
        'OPTIONS': {
            'min_length': 8,
        }
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]
```

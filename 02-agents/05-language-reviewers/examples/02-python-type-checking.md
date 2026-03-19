# 场景 2: Python 类型检查

**语言：** Python
**工具：** mypy
**优先级：** HIGH (类型安全)

## 错误场景

类型注解缺失、不匹配和类型安全问题。

### 问题代码

```python
# file: src/user/user_service.py

from typing import List, Dict, Optional

class User:
    def __init__(self, id: int, name: str, email: str):
        self.id = id
        self.name = name
        self.email = email

class UserService:
    def __init__(self, db):
        self.db = db

    # ❌ 缺少返回类型注解
    def create_user(self, username, email, password):
        user = User(
            id=len(self.db.users) + 1,
            name=username,
            email=email
        )
        self.db.users.append(user)
        return user

    # ❌ 参数类型不明确
    def get_user_by_id(self, user_id):
        for user in self.db.users:
            if user.id == user_id:
                return user
        return None

    # ❌ 返回类型与注解不匹配
    def search_users(self, query: str) -> List[User]:
        results = []
        for user in self.db.users:
            if query.lower() in user.name.lower():
                results.append(user)

        # ❌ 可能返回 None
        if not results:
            return None
        return results

    # ❌ 复杂类型未定义
    def process_batch(self, items):
        results = []
        for item in items:
            # ❌ 不知道 item 是什么类型
            processed = self._process_item(item)
            results.append(processed)
        return results

    def _process_item(self, item):
        # ❌ 没有类型信息
        return {
            'id': item.id,
            'data': item.data,
            'timestamp': datetime.now()
        }
```

### 类型检查错误

```bash
$ mypy src/user/user_service.py --strict

src/user/user_service.py:15: error: Missing return type annotation
src/user/user_service.py:15: error: Function is missing a type annotation
src/user/user_service.py:21: error: Function is missing a type annotation for one or more arguments
src/user/user_service.py:29: error: Incompatible return value type (got "Optional[List[User]]", expected "List[User]")
src/user/user_service.py:38: error: Function is missing a type annotation
src/user/user_service.py:42: error: Function is missing a type annotation
src/user/user_service.py:45: error: Function is missing a type annotation
src/user/user_service.py:47: error: Name 'datetime' is not defined

Found 8 errors in 1 file.
```

### 修复方案

```python
# file: src/user/user_service.py (已修复)

from typing import List, Dict, Optional, Any
from datetime import datetime
from dataclasses import dataclass

@dataclass
class User:
    id: int
    name: str
    email: str

class UserService:
    def __init__(self, db: Any):  # ✅ 明确 db 类型（实际应用中使用具体类型）
        self.db = db

    # ✅ 添加完整的类型注解
    def create_user(
        self,
        username: str,
        email: str,
        password: str
    ) -> User:
        user = User(
            id=len(self.db.users) + 1,
            name=username,
            email=email
        )
        self.db.users.append(user)
        return user

    # ✅ 明确参数和返回类型
    def get_user_by_id(self, user_id: int) -> Optional[User]:
        for user in self.db.users:
            if user.id == user_id:
                return user
        return None

    # ✅ 返回类型与注解匹配
    def search_users(self, query: str) -> Optional[List[User]]:
        results: List[User] = []
        for user in self.db.users:
            if query.lower() in user.name.lower():
                results.append(user)

        return results if results else None

    # ✅ 使用泛型处理不同类型
    def process_batch(self, items: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
        results: List[Dict[str, Any]] = []
        for item in items:
            processed = self._process_item(item)
            results.append(processed)
        return results

    # ✅ 明确返回类型
    def _process_item(self, item: Dict[str, Any]) -> Dict[str, Any]:
        return {
            'id': item.get('id'),
            'data': item.get('data'),
            'timestamp': datetime.now()
        }
```

### 验证修复

```bash
$ mypy src/user/user_service.py --strict
# ✅ Success: no issues found in 1 source file
```

## 关键要点

### 类型注解最佳实践

1. **函数类型注解**
   ```python
   # ✅ 完整类型注解
   def function_name(param1: str, param2: int) -> bool:
       return True

   # ✅ 可选参数
   def optional_param(param: Optional[str] = None) -> str:
       return param or "default"
   ```

2. **复杂类型**
   ```python
   from typing import List, Dict, Tuple, Union

   # 列表类型
   items: List[int] = [1, 2, 3]

   # 字典类型
   user_data: Dict[str, Any] = {"name": "John"}

   # 元组类型
   coordinates: Tuple[float, float] = (1.0, 2.0)

   # 联合类型
   value: Union[str, int] = "hello"  # 或 123
   ```

3. **类和接口**
   ```python
   from dataclasses import dataclass

   @dataclass
   class User:
       id: int
       name: str
       email: str

   # 使用类型别名
   UserId = int
   def get_user(user_id: UserId) -> Optional[User]:
       pass
   ```

### 常见类型错误

| 错误类型 | 原因 | 修复方法 |
|---------|------|----------|
| Missing return annotation | 函数缺少返回类型注解 | 添加 `-> Type` |
| Incompatible return type | 返回值类型不匹配 | 修正返回类型或返回值 |
| Missing type annotation | 参数缺少类型注解 | 添加 `param: Type` |
| Name not defined | 使用了未导入的类型 | 导入必要的类型 |

### 配置文件

```toml
# file: mypy.ini

[mypy]
python_version = 3.9
warn_return_any = True
warn_unused_configs = True
disallow_untyped_defs = True
disallow_incomplete_defs = True
check_untyped_defs = True
no_implicit_optional = True
warn_redundant_casts = True
warn_unused_ignores = True
warn_no_return = True
strict_equality = True

# 指定类型检查路径
[mypy-src.user]
disallow_untyped_defs = True

# 排除第三方库
[mypy-tests.*]
ignore_errors = True
```

### 工具集成

```yaml
# .github/workflows/type-check.yml

name: Type Check

on: [push, pull_request]

jobs:
  mypy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.9'

      - name: Install dependencies
        run: |
          pip install mypy

      - name: Run mypy
        run: mypy src/
```

---

**场景价值：** Python 是动态类型语言，类型注解可以在开发时捕获错误，提高代码质量和维护性。

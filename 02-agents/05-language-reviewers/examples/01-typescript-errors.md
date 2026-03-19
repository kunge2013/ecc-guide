# 场景 1: TypeScript 类型错误

**语言：** TypeScript
**工具：** tsc
**优先级：** HIGH (阻止编译)

## 错误场景

类型不匹配和类型安全问题导致编译失败。

### 问题代码

```typescript
// file: src/user/UserService.ts

interface User {
  id: number;
  name: string;
  email: string;
}

interface UserData {
  name: string;
  email: string;
}

class UserService {
  async createUser(userData: any) {
    // ❌ any 类型绕过了类型检查
    const user: User = {
      id: Math.random(),
      name: userData.username, // ❌ 属性名不匹配
      email: userData.email
    };
    return user;
  }

  async getUserById(id: string) {
    // ❌ id 参数类型不匹配（应该是 number）
    return { id, name: 'John', email: 'john@example.com' };
  }

  async filterUsers(users: User[], criteria: any) {
    // ❌ criteria 类型不明确
    return users.filter(user => {
      if (criteria.minAge && user.age < criteria.minAge) {
        // ❌ User 接口没有 age 属性
        return false;
      }
      return true;
    });
  }
}
```

### 编译错误

```bash
$ npx tsc --noEmit

src/user/UserService.ts:12:19 - error TS2339: Property 'username' does not exist on type 'any'.
12       name: userData.username,
                     ~~~~~~~~~

src/user/UserService.ts:17:30 - error TS2345: Argument of type 'string' is not assignable to parameter of type 'number'.
17   async getUserById(id: string) {
                              ~~~~

src/user/UserService.ts:27:38 - error TS2339: Property 'age' does not exist on type 'User'.
27       if (criteria.minAge && user.age < criteria.minAge) {
                                       ~~~~

Found 3 errors.
```

### 修复方案

```typescript
// file: src/user/UserService.ts (已修复)

interface User {
  id: number;
  name: string;
  email: string;
  age?: number; // ✅ 添加可选属性
}

interface UserData {
  name: string;
  email: string;
}

interface FilterCriteria {
  minAge?: number;
  maxAge?: number;
  name?: string;
}

class UserService {
  // ✅ 使用明确的类型
  async createUser(userData: UserData): Promise<User> {
    const user: User = {
      id: Math.random(),
      name: userData.name, // ✅ 属性名匹配
      email: userData.email
    };
    return user;
  }

  // ✅ 修正参数类型
  async getUserById(id: number): Promise<User> {
    return { id, name: 'John', email: 'john@example.com' };
  }

  // ✅ 使用明确的 criteria 类型
  async filterUsers(
    users: User[],
    criteria: FilterCriteria
  ): Promise<User[]> {
    return users.filter(user => {
      if (criteria.minAge && user.age && user.age < criteria.minAge) {
        return false;
      }
      if (criteria.maxAge && user.age && user.age > criteria.maxAge) {
        return false;
      }
      if (criteria.name && !user.name.includes(criteria.name)) {
        return false;
      }
      return true;
    });
  }
}
```

### 验证修复

```bash
$ npx tsc --noEmit
# ✅ 无错误
```

## 关键要点

### TypeScript 错误类型

| 错误代码 | 含义 | 常见原因 |
|---------|------|----------|
| TS2339 | 属性不存在 | 类型定义不完整 |
| TS2345 | 类型不匹配 | 参数类型错误 |
| TS7006 | 隐式 any | 缺少类型注解 |
| TS2554 | 参数错误 | 参数数量或类型不对 |
| TS2663 | 可能为 null | 没有处理 null/undefined |

### 修复策略

1. **使用明确类型**
   ```typescript
   // ❌ 避免
   function processData(data: any) { }

   // ✅ 推荐
   interface Data { /* ... */ }
   function processData(data: Data) { }
   ```

2. **接口完整性**
   ```typescript
   // 添加所有必要的属性
   interface User {
     id: number;
     name: string;
     email: string;
     age?: number; // 可选属性
   }
   ```

3. **类型守卫**
   ```typescript
   function isUser(obj: any): obj is User {
     return obj && typeof obj.id === 'number';
   }

   if (isUser(data)) {
     // data 现在是 User 类型
   }
   ```

### 最佳实践

- ✅ 启用 strict 模式
- ✅ 避免 any 类型
- ✅ 使用明确的接口
- ✅ 处理可选类型
- ✅ 使用类型守卫

---

**场景价值：** TypeScript 类型错误在编译时就能发现，是构建失败最常见的原因之一。及时修复能保证类型安全。

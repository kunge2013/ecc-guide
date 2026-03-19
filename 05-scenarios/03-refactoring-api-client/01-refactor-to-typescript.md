# 使用场景 1: 重构为 TypeScript

## 重构目标

将 JavaScript API 客户端重构为 TypeScript，提高类型安全和可维护性。

## 重构前（JavaScript）

```javascript
// file: src/api/user.js

class UserAPI {
  constructor(baseURL) {
    this.baseURL = baseURL;
  }

  async getUsers(filters) {
    const params = new URLSearchParams();
    if (filters) {
      if (filters.status) {
        params.append('status', filters.status);
      }
      if (filters.role) {
        params.append('role', filters.role);
      }
    }

    const response = await fetch(`${this.baseURL}/users?${params}`);
    const data = await response.json();
    return data.users;
  }

  async createUser(userData) {
    const response = await fetch(`${this.baseURL}/users`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(userData)
    });

    const data = await response.json();
    return data.user;
  }

  async updateUser(userId, updates) {
    const response = await fetch(`${this.baseURL}/users/${userId}`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(updates)
    });

    const data = await response.json();
    return data.user;
  }

  async deleteUser(userId) {
    const response = await fetch(`${this.baseURL}/users/${userId}`, {
      method: 'DELETE'
    });

    return response.ok;
  }
}
```

## 重构计划

```bash
# 使用 Planner Agent
Agent(planner, """
规划 UserAPI 重构为 TypeScript：

重构步骤：
1. 定义类型接口
2. 转换函数签名
3. 添加类型注解
4. 改进错误处理
5. 更新测试

保持向后兼容：
- 不改变公共 API
- 不改变请求/响应格式
- 确保现有调用代码仍能工作
""")
```

**Planner 输出：**

```markdown
# UserAPI TypeScript 重构计划

## 类型定义

### 用户类型
```typescript
interface User {
  id: number;
  name: string;
  email: string;
  role: 'admin' | 'user' | 'guest';
  status: 'active' | 'inactive';
  createdAt: string;
  updatedAt: string;
}
```

### 请求/响应类型
```typescript
interface GetUsersFilters {
  status?: User['status'];
  role?: User['role'];
}

interface CreateUserRequest {
  name: string;
  email: string;
  password: string;
  role?: User['role'];
}

interface UpdateUserRequest {
  name?: string;
  email?: string;
  role?: User['role'];
  status?: User['status'];
}

interface APIResponse<T> {
  success: boolean;
  data: T;
  error?: string;
}

interface Error {
  code: string;
  message: string;
  details?: any;
}
```

### 错误类型
```typescript
class APIError extends Error {
  constructor(
    public code: string,
    message: string,
    public details?: any
  ) {
    super(message);
    this.name = 'APIError';
  }
}

class NetworkError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'NetworkError';
  }
}

class ValidationError extends Error {
  constructor(
    public field: string,
    message: string
  ) {
    super(message);
    this.name = 'ValidationError';
  }
}
```

## 重构步骤

### 阶段 1: 类型定义
- [ ] 创建 types.ts
- [ ] 定义所有接口
- [ ] 定义错误类

### 阶段 2: 函数签名转换
- [ ] 转换 getUsers 函数
- [ ] 转换 createUser 函数
- [ ] 转换 updateUser 函数
- [ ] 转换 deleteUser 函数

### 阶段 3: 错误处理改进
- [ ] 统一错误处理
- [ ] 添加错误类
- [ ] 改进错误消息

### 阶段 4: 测试更新
- [ ] 更新单元测试
- [ ] 添加类型测试
- [ ] 验证向后兼容性

## 向后兼容性保证

### 配置编译器
```json
{
  "compilerOptions": {
    "allowJs": true,
        "declaration": true,
        "declarationMap": true,
        "outDir": "./dist"
  }
}
```

### 提供 .d.ts 文件
- 生成 TypeScript 声明文件
- 保留 JavaScript 导出
- 支持渐进式迁移
```

## 重构后（TypeScript）

```typescript
// file: src/api/user.ts

// 类型定义
interface User {
  id: number;
  name: string;
  email: string;
  role: 'admin' | 'user' | 'guest';
  status: 'active' | 'inactive';
  createdAt: string;
  updatedAt: string;
}

interface GetUsersFilters {
  status?: User['status'];
  role?: User['role'];
}

interface CreateUserRequest {
  name: string;
  email: string;
  password: string;
  role?: User['role'];
}

interface UpdateUserRequest {
  name?: string;
  email?: string;
  role?: User['role'];
  status?: User['status'];
}

// 错误类
class APIError extends Error {
  constructor(
    public code: string,
    message: string,
    public details?: any
  ) {
    super(message);
    this.name = 'APIError';
  }
}

class NetworkError extends Error {
  constructor(message: string) {
    super(message);
    this.name = 'NetworkError';
  }
}

// API 类
class UserAPI {
  constructor(private baseURL: string) {}

  async getUsers(filters?: GetUsersFilters): Promise<User[]> {
    const params = this.buildQueryParams(filters);
    const url = `${this.baseURL}/users?${params}`;

    try {
      const response = await this.fetchWithTimeout(url);
      const data = await response.json();

      if (!response.ok) {
        throw new APIError('FETCH_USERS_FAILED', '获取用户失败', data);
      }

      return data.users as User[];
    } catch (error) {
      if (error instanceof TypeError) {
        throw new NetworkError('网络错误');
      }
      throw error;
    }
  }

  async createUser(userData: CreateUserRequest): Promise<User> {
    this.validateCreateUser(userData);

    const url = `${this.baseURL}/users`;
    const options = {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(userData)
    };

    try {
      const response = await this.fetchWithTimeout(url, options);
      const data = await response.json();

      if (!response.ok) {
        throw new APIError('CREATE_USER_FAILED', '创建用户失败', data);
      }

      return data.user as User;
    } catch (error) {
      if (error instanceof TypeError) {
        throw new NetworkError('网络错误');
      }
      throw error;
    }
  }

  async updateUser(
    userId: number,
    updates: UpdateUserRequest
  ): Promise<User> {
    if (Object.keys(updates).length === 0) {
      throw new APIError('NO_UPDATES', '没有提供更新数据');
    }

    const url = `${this.baseURL}/users/${userId}`;
    const options = {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(updates)
    };

    try {
      const response = await this.fetchWithTimeout(url, options);
      const data = await response.json();

      if !response.ok) {
        throw new APIError('UPDATE_USER_FAILED', '更新用户失败', data);
      }

      return data.user as User;
    } catch (error) {
      if (error instanceof TypeError) {
        throw new NetworkError('网络错误');
      }
      throw error;
    }
  }

  async deleteUser(userId: number): Promise<boolean> {
    const url = `${this.baseURL}/users/${userId}`;
    const options = { method: 'DELETE' };

    try {
      const response = await this.fetchWithTimeout(url, options);

      if (!response.ok) {
        throw new APIError('DELETE_USER_FAILED', '删除用户失败');
      }

      return true;
    } catch (error) {
      if (error instanceof TypeError) {
        throw new NetworkError('网络错误');
      }
      throw error;
    }
  }

  // 辅助方法
  private buildQueryParams(filters?: GetUsersFilters): string {
    const params = new URLSearchParams();

    if (filters?.status) {
      params.append('status', filters.status);
    }
    if (filters?.role) {
      params.append('role', filters.role);
    }

    return params.toString();
  }

  private validateCreateUser(userData: CreateUserRequest): void {
    if (!userData.name.trim()) {
      throw new Error('用户名不能为空');
    }

    if (!userData.email.match(/^[^\s@]+@[^\s@]+\.[^\s@]+$/)) {
      throw new Error('邮箱格式无效');
    }

    if (!userData.password || userData.password.length < 8) {
      throw new Error('密码至少 8 个字符');
    }
  }

  private async fetchWithTimeout(
    url: string,
    options?: RequestInit,
    timeout = 30000
  ): Promise<Response> {
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), timeout);

    try {
      const response = await fetch(url, {
        ...options,
        signal: controller.signal
      });
      clearTimeout(timeoutId);
      return response;
    } catch (error) {
      clearTimeout(timeoutId);
      if (error.name === 'AbortError') {
        throw new Error('请求超时');
      }
      throw error;
    }
  }
}

// 导出
export { UserAPI, User, APIError, NetworkError };
export type {
  GetUsersFilters,
  CreateUserRequest,
  UpdateUserRequest
};
```

## 测试

```typescript
// file: test/api/user.test.ts

import { UserAPI, User, APIError, NetworkError } from '../../src/api/user';

describe('UserAPI', () => {
  let api: UserAPI;
  const mockBaseURL = 'http://localhost:3000/api';

  beforeEach(() => {
    api = new UserAPI(mockBaseURL);
    global.fetch = jest.fn();
  });

  describe('getUsers', () => {
    it('应该返回用户列表', async () => {
      const mockUsers: User[] = [
        { id: 1, name: 'John', email: 'john@example.com', role: 'user', status: 'active', createdAt: '2024-01-01', updatedAt: '2024-01-01' },
        { id: 2, name: 'Jane', email: 'jane@example.com', role: 'user', status: 'active', createdAt: '2024-01-01', updatedAt: '2024-01-01' }
      ];

      (global.fetch as jest.Mock).mockResolvedValueOnce({
        ok: true,
        json: async () => ({ users: mockUsers })
      });

      const result = await api.getUsers();

      expect(result).toEqual(mockUsers);
      expect(result.length).toBe(2);
    });

    it('应该使用过滤参数', async () => {
      const filters = { status: 'active', role: 'user' };

      (global.fetch as jest.Mock).mockResolvedValueOnce({
        ok: true,
        json: async () => ({ users: [] })
      });

      await api.getUsers(filters);

      expect(global.fetch).toHaveBeenCalledWith(
        'http://localhost:3000/api/users?status=active&role=user'
      );
    });

    it('应该处理网络错误', async () => {
      (global.fetch as jest.Mock).mockRejectedValueOnce(new TypeError('Network error'));

      await expect(api.getUsers()).rejects.toThrow(NetworkError);
    });

    it('应该处理 API 错误', async () => {
      (global.fetch as jest.Mock).mockResolvedValueOnce({
        ok: false,
        json: async () => ({ error: '获取用户失败' })
      });

      await expect(api.getUsers()).rejects.toThrow(APIError);
    });
  });

  describe('createUser', () => {
    it('应该创建用户', async () => {
      const mockUser: User = {
        id: 1, name: 'John', email: 'john@example.com', role: 'user', status: 'active', createdAt: '2024-01-01', updatedAt: '2024-01-01'
      };
      const userData = {
        name: 'John',
        email: 'john@example.com',
        password: 'SecurePass123'
      };

      (global.fetch as jest.Mock).mockResolvedValueOnce({
        ok: true,
        json: async () => ({ user: mockUser })
      });

      const result = await api.createUser(userData);

      expect(result).toEqual(mockUser);
    });

    it('应该验证用户名', async () => {
      const userData = {
        name: '   ', // 空用户名
        email: 'john@example.com',
        password: 'SecurePass123'
      };

      await expect(api.createUser(userData)).rejects.toThrow('用户名不能为空');
    });

    it('应该验证邮箱格式', async () => {
      const userData = {
        name: 'John',
        email: 'invalid-email', // 无效邮箱
        password: 'SecurePass123'
      };

      await expect(api.createUser(userData)).rejects.toThrow('邮箱格式无效');
    });
  });
});
```

## 关键学习点

### 1. TypeScript 重构策略

- **渐进式迁移** - 允许 JavaScript 和 TypeScript 混用
- **类型优先** - 先定义类型，再实现逻辑
- **保持兼容** - 不破坏现有 API
- **逐步改进** - 先添加类型注解，再改进设计

### 2. 类型设计原则

- **接口明确** - 清晰定义数据结构
- **枚举类型** - 使用字面量类型限制值
- **泛型使用** - 提高代码复用性
- **错误类型** - 自定义错误类提供更多信息

### 3. 向后兼容性

```json
// tsconfig.json
{
  "compilerOptions": {
    "allowJs": true,
        "module": "commonjs",
        "target": "es2015",
        "declaration": true,
        "outDir": "./dist",
        "moduleResolution": "node"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "**/*.test.ts"]
}
```

---

**提示：** TypeScript 重构应该分阶段进行，先建立类型系统，再逐步改进代码设计。

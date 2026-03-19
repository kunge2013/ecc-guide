# 示例：开发 REST API 端点

## 场景描述

需要添加一个 REST API 端点，用于获取用户的任务统计信息，包括：
- 总任务数
- 已完成任务数
- 进行中任务数
- 逾期任务数

## 完整工作流程

### 第 1 步：规划阶段

```bash
用户: /plan 我需要添加一个 REST API 端点，用于获取用户的任务统计信息

ECC: [planner 代理]
```

**ECC 响应**：

```
分析需求...

## 实现计划

### 功能需求
- 创建 GET /api/users/{userId}/stats 端点
- 返回用户的任务统计数据
- 需要认证：只有用户本人或管理员可以访问

### 数据结构
**响应格式**:
```json
{
  "success": true,
  "data": {
    "total": 150,
    "completed": 120,
    "inProgress": 25,
    "overdue": 5
  }
}
```

### 实现步骤
1. 创建路由处理器 `/api/users/{userId}/stats`
2. 实现用户认证和授权检查
3. 添加数据查询逻辑
4. 编写单元测试
5. 编写集成测试
6. 添加 API 文档

### 依赖
- 现有用户认证系统
- 任务数据库模型

### 风险
- 大量数据时可能影响性能，考虑添加缓存
- 需要处理用户不存在的情况

是否开始实现？
```

```bash
用户: yes  # 确认计划
```

### 第 2 步：TDD 实现

```bash
用户: /tdd 实现任务统计 API 端点

ECC: [tdd-guide 代理]
```

**ECC 响应 - RED 阶段（编写测试）**：

```
## 第 1 步：编写测试（RED）

### 单元测试 - 数据查询逻辑

创建文件: `tests/unit/user-stats.test.ts`

```typescript
import { getUserStats } from '../../src/services/userStats';

describe('getUserStats', () => {
  it('应该返回正确的任务统计', async () => {
    const stats = await getUserStats('user123');
    expect(stats).toEqual({
      total: 150,
      completed: 120,
      inProgress: 25,
      overdue: 5
    });
  });

  it('用户不存在时应返回空统计', async () => {
    const stats = await getUserStats('nonexistent');
    expect(stats).toEqual({
      total: 0,
      completed: 0,
      inProgress: 0,
      overdue: 0
    });
  });
});
```

### 集成测试 - API 端点

创建文件: `tests/integration/user-stats-api.test.ts`

```typescript
import request from 'supertest';
import app from '../../src/app';

describe('GET /api/users/:userId/stats', () => {
  it('应该返回用户任务统计', async () => {
    const response = await request(app)
      .get('/api/users/user123/stats')
      .set('Authorization', 'Bearer valid-token');

    expect(response.status).toBe(200);
    expect(response.body).toEqual({
      success: true,
      data: {
        total: 150,
        completed: 120,
        inProgress: 25,
        overdue: 5
      }
    });
  });

  it('未认证时应返回 401', async () => {
    const response = await request(app)
      .get('/api/users/user123/stats');

    expect(response.status).toBe(401);
  });

  it('无权限时应返回 403', async () => {
    const response = await request(app)
      .get('/api/users/otheruser/stats')
      .set('Authorization', 'Bearer user123-token');

    expect(response.status).toBe(403);
  });
});
```

### 运行测试（预期失败）
```bash
npm test -- getUserStats
```

预期结果: ✗ 测试失败（功能未实现）
```

**ECC 响应 - GREEN 阶段（实现功能）**：

```
## 第 2 步：实现功能（GREEN）

创建文件: `src/services/userStats.ts`

```typescript
export interface UserStats {
  total: number;
  completed: number;
  inProgress: number;
  overdue: number;
}

export async function getUserStats(userId: string): Promise<UserStats> {
  const tasks = await TaskModel.find({ userId });

  return {
    total: tasks.length,
    completed: tasks.filter(t => t.status === 'completed').length,
    inProgress: tasks.filter(t => t.status === 'in_progress').length,
    overdue: tasks.filter(t =>
      t.status !== 'completed' && t.dueDate < new Date()
    ).length
  };
}
```

创建文件: `src/routes/userStats.ts`

```typescript
import { Router } from 'express';
import { authenticate, authorize } from '../middleware/auth';
import { getUserStats } from '../services/userStats';

const router = Router();

router.get('/users/:userId/stats', authenticate, async (req, res) => {
  const { userId } = req.params;
  const currentUser = req.user;

  // 授权检查：只有用户本人或管理员可以访问
  if (currentUser.id !== userId && !currentUser.isAdmin) {
    return res.status(403).json({ success: false, error: 'Forbidden' });
  }

  const stats = await getUserStats(userId);

  res.json({
    success: true,
    data: stats
  });
});

export default router;
```

### 运行测试（预期通过）
```bash
npm test -- getUserStats
```

预期结果: ✓ 所有测试通过
```

**ECC 响应 - IMPROVE 阶段（重构优化）**：

```
## 第 3 步：重构优化（IMPROVE）

### 优化建议

1. **添加缓存**：对于频繁访问的统计数据，添加 Redis 缓存
2. **批量查询**：使用聚合查询代替内存过滤
3. **添加日志**：记录 API 访问情况

### 重构后的数据查询逻辑

```typescript
export async function getUserStats(userId: string): Promise<UserStats> {
  // 检查缓存
  const cached = await cache.get(`stats:${userId}`);
  if (cached) return JSON.parse(cached);

  // 使用聚合查询优化性能
  const stats = await TaskModel.aggregate([
    { $match: { userId } },
    {
      $group: {
        _id: null,
        total: { $sum: 1 },
        completed: {
          $sum: { $cond: [{ $eq: ['$status', 'completed'] }, 1, 0] }
        },
        inProgress: {
          $sum: { $cond: [{ $eq: ['$status', 'in_progress'] }, 1, 0] }
        },
        overdue: {
          $sum: {
            $cond: [
              { $and: [
                { $ne: ['$status', 'completed'] },
                { $lt: ['$dueDate', new Date()] }
              ]},
              1, 0
            ]
          }
        }
      }
    }
  ]);

  const result = stats[0] || { total: 0, completed: 0, inProgress: 0, overdue: 0 };

  // 缓存结果（5 分钟过期）
  await cache.set(`stats:${userId}`, JSON.stringify(result), 300);

  return result;
}
```

### 运行测试确保重构没有破坏功能
```bash
npm test -- getUserStats
```

预期结果: ✓ 所有测试通过
```

### 第 3 步：代码审查

```bash
用户: /code-review 审查任务统计 API 的代码

ECC: [code-reviewer 代理]
```

**ECC 响应**：

```
## 代码审查报告

### ✅ 通过项
- ✓ 代码结构清晰，职责分离
- ✓ 错误处理完善
- ✓ 测试覆盖完整
- ✓ 添加了性能优化（缓存、聚合查询）

### ⚠️ 建议改进

#### 1. 添加速率限制
**位置**: `src/routes/userStats.ts`

**建议**: 添加速率限制中间件，防止 API 滥用

```typescript
import rateLimit from 'express-rate-limit';

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 分钟
  max: 100 // 最多 100 次请求
});

router.get('/users/:userId/stats', limiter, authenticate, ...);
```

#### 2. 改进错误消息
**位置**: 数据查询逻辑

**建议**: 使用统一的错误消息格式

```typescript
if (!user) {
  throw new NotFoundError('User not found');
}
```

#### 3. 添加监控指标
**建议**: 记录 API 性能指标

```typescript
const startTime = Date.now();
const stats = await getUserStats(userId);
const duration = Date.now() - startTime;
metrics.recordApiCall('userStats', duration);
```

### 🔒 安全检查
- ✓ 认证验证完善
- ✓ 授权检查正确
- ✓ 无 SQL 注入风险
- ✓ 未暴露敏感信息

### 总体评分: 8.5/10
- 功能完整性: 9/10
- 代码质量: 8/10
- 测试覆盖: 9/10
- 安全性: 9/10
```

### 第 4 步：端到端测试

```bash
用户: /e2e 生成并运行 API 端点测试

ECC: [e2e-runner 代理]
```

**ECC 响应**：

```
## 端到端测试

### 生成测试脚本

创建文件: `e2e/tests/user-stats-flow.spec.ts`

```typescript
import { test, expect } from '@playwright/test';

test.describe('用户任务统计 API 端到端测试', () => {
  test('完整流程：登录 -> 获取统计 -> 验证数据', async ({ request }) => {
    // 1. 登录获取 token
    const loginResponse = await request.post('/api/auth/login', {
      data: {
        email: 'test@example.com',
        password: 'password123'
      }
    });
    const { token } = await loginResponse.json();

    // 2. 获取任务统计
    const statsResponse = await request.get('/api/users/user123/stats', {
      headers: {
        Authorization: `Bearer ${token}`
      }
    });

    expect(statsResponse.status()).toBe(200);
    const { success, data } = await statsResponse.json();
    expect(success).toBe(true);
    expect(data).toHaveProperty('total');
    expect(data).toHaveProperty('completed');
    expect(data).toHaveProperty('inProgress');
    expect(data).toHaveProperty('overdue');

    // 3. 验证数据一致性
    expect(data.total).toBe(
      data.completed + data.inProgress + data.overdue
    );
  });

  test('性能测试：响应时间应 < 200ms', async ({ request }) => {
    const startTime = Date.now();

    await request.get('/api/users/user123/stats', {
      headers: {
        Authorization: 'Bearer valid-token'
      }
    });

    const duration = Date.now() - startTime;
    expect(duration).toBeLessThan(200);
  });
});
```

### 运行测试
```bash
npm run e2e
```

预期结果:
✓ 完整流程测试通过
✓ 性能测试通过
```

### 第 5 步：提交代码

```bash
# 添加文件
git add src/services/userStats.ts
git add src/routes/userStats.ts
git add tests/unit/user-stats.test.ts
git add tests/integration/user-stats-api.test.ts
git add e2e/tests/user-stats-flow.spec.ts

# 提交
git commit -m "feat: 添加用户任务统计 API 端点

- 创建 GET /api/users/{userId}/stats 端点
- 实现用户认证和授权
- 添加 Redis 缓存优化性能
- 使用聚合查询优化数据库访问
- 完整的单元测试和集成测试
- 端到端测试验证功能

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```

## 关键要点

### ✅ 最佳实践

1. **先规划后实现**：使用 `/plan` 明确需求和实现步骤
2. **测试驱动开发**：使用 `/tdd` 遵循 RED-GREEN-IMPROVE 循环
3. **代码审查**：使用 `/code-review` 确保代码质量
4. **端到端测试**：使用 `/e2e` 验证完整功能
5. **性能优化**：添加缓存和聚合查询提高性能

### 📊 测试覆盖

- ✓ 单元测试：数据查询逻辑
- ✓ 集成测试：API 端点
- ✓ 端到端测试：完整用户流程
- ✓ 性能测试：响应时间验证

### 🔒 安全考虑

- ✓ 用户认证
- ✓ 权限验证
- ✓ 速率限制
- ✓ 无敏感信息泄露

## 扩展建议

1. **添加 Swagger 文档**：生成 API 文档
2. **监控和告警**：添加性能监控
3. **分页支持**：如果数据量大，添加分页
4. **WebSocket 实时更新**：任务更新时实时推送统计数据

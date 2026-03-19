# 示例：Bug 修复 - 数据库连接泄漏

## 场景描述

生产环境报告内存泄漏问题，经过初步排查，怀疑是数据库连接未正确关闭导致。

## 完整工作流程

### 第 1 步：重现问题

```bash
用户: 我发现生产环境有内存泄漏问题，数据库连接数持续增长，需要修复
```

**ECC 响应**：

```
## 问题分析

### 症状
- 生产环境内存使用率持续上升
- 数据库连接数接近最大限制
- 应用偶尔抛出 "Too many connections" 错误

### 初步诊断
可能的原因：
1. 数据库连接未正确关闭
2. 连接池配置不当
3. 长时间运行的查询未释放连接
4. 异常情况下连接未释放

### 排查步骤
1. 检查数据库连接池配置
2. 检查代码中的数据库操作
3. 查看日志中的连接泄漏警告
4. 使用 pprof 进行内存分析

是否开始制定修复计划？
```

```bash
用户: yes
```

### 第 2 步：规划修复

```bash
用户: /plan 制定数据库连接泄漏修复计划

ECC: [planner 代理]
```

**ECC 响应**：

```
## 修复计划

### 问题根因分析
1. 检查数据库连接池配置
2. 查找未正确关闭连接的代码
3. 识别异常处理不当的地方

### 修复策略
1. **修复未关闭的连接**：确保所有数据库操作正确关闭连接
2. **优化连接池配置**：调整连接池大小和超时设置
3. **添加监控**：记录连接创建和释放
4. **编写测试**：添加连接泄漏检测测试

### 实现步骤
1. 审查所有数据库操作代码
2. 修复连接泄漏问题
3. 添加连接池配置优化
4. 添加监控和日志
5. 编写单元测试验证修复
6. 编写集成测试验证连接池行为

### 验证方法
1. 运行压力测试
2. 监控连接池状态
3. 检查内存使用情况
4. 验证异常情况下的连接释放

### 风险
- 修复可能影响现有功能
- 需要充分的测试覆盖
- 需要在测试环境充分验证

是否开始实现？
```

```bash
用户: yes
```

### 第 3 步：TDD 修复

```bash
用户: /tdd 修复数据库连接泄漏问题

ECC: [tdd-guide 代理]
```

**ECC 响应 - RED 阶段（编写测试）**：

```
## 第 1 步：编写测试（RED）

### 单元测试 - 连接泄漏检测

创建文件: `tests/unit/connection-pool.test.ts`

```typescript
import { getConnection, releaseConnection, getPoolStats } from '../../src/db/connection';

describe('数据库连接池管理', () => {
  afterEach(async () => {
    // 每次测试后清理连接池
    await cleanupConnections();
  });

  it('应该正确释放连接', async () => {
    const initialStats = getPoolStats();
    const connection = await getConnection();

    await releaseConnection(connection);

    const finalStats = getPoolStats();
    expect(finalStats.activeConnections).toBe(initialStats.activeConnections);
  });

  it('应该处理异常时的连接释放', async () => {
    const initialStats = getPoolStats();

    try {
      const connection = await getConnection();
      // 模拟数据库操作失败
      throw new Error('Database error');
    } catch (error) {
      // 异常应该被捕获，连接应该被释放
    }

    const finalStats = getPoolStats();
    await new Promise(resolve => setTimeout(resolve, 100)); // 等待清理
    expect(finalStats.activeConnections).toBe(initialStats.activeConnections);
  });

  it('长时间未释放连接应该被自动回收', async () => {
    const initialStats = getPoolStats();
    await getConnection();
    // 故意不释放连接

    // 等待连接超时自动回收
    await new Promise(resolve => setTimeout(resolve, 60000));

    const finalStats = getPoolStats();
    expect(finalStats.activeConnections).toBeLessThanOrEqual(initialStats.activeConnections + 1);
  });
});
```

### 集成测试 - 压力测试

创建文件: `tests/integration/connection-stress.test.ts`

```typescript
describe('连接池压力测试', () => {
  it('大量并发请求后连接数应该正常', async () => {
    const initialStats = getPoolStats();

    // 模拟 100 个并发请求
    const requests = Array(100).fill(null).map(() =>
      fetch('/api/data').then(res => res.json())
    );

    await Promise.all(requests);

    const finalStats = getPoolStats();
    expect(finalStats.activeConnections).toBe(initialStats.activeConnections);
  });

  it('异常情况下连接池应该保持稳定', async () => {
    const initialStats = getPoolStats();

    // 模拟会抛出异常的请求
    const requests = Array(50).fill(null).map((_, i) => {
      if (i % 10 === 0) {
        // 每 10 个请求抛出异常
        return fetch('/api/error-endpoint').catch(() => {});
      }
      return fetch('/api/data').then(res => res.json());
    });

    await Promise.all(requests);

    // 等待连接清理
    await new Promise(resolve => setTimeout(resolve, 1000));

    const finalStats = getPoolStats();
    expect(finalStats.activeConnections).toBeLessThanOrEqual(initialStats.activeConnections + 5);
  });
});
```

### 运行测试（预期失败）
```bash
npm test -- connection-pool
```

预期结果: ✗ 连接泄漏测试失败
```

**ECC 响应 - GREEN 阶段（修复问题）**：

```
## 第 2 步：修复问题（GREEN）

### 问题定位

通过代码审查发现以下问题：

1. **文件**: `src/services/userService.ts`
```typescript
// ❌ 问题代码：未关闭连接
export async function getUser(userId: string) {
  const connection = await getConnection();
  const user = await connection.query('SELECT * FROM users WHERE id = ?', [userId]);
  // 忘记释放连接！
  return user[0];
}
```

2. **文件**: `src/services/orderService.ts`
```typescript
// ❌ 问题代码：异常时未释放连接
export async function createOrder(orderData: Order) {
  const connection = await getConnection();
  try {
    const result = await connection.query('INSERT INTO orders SET ?', orderData);
    return result.insertId;
  } catch (error) {
    throw error; // 异常时连接未释放
  }
}
```

### 修复方案

**修复 1**: `src/services/userService.ts`

```typescript
// ✅ 修复后代码
export async function getUser(userId: string) {
  const connection = await getConnection();
  try {
    const user = await connection.query('SELECT * FROM users WHERE id = ?', [userId]);
    return user[0];
  } finally {
    await releaseConnection(connection);
  }
}
```

**修复 2**: `src/services/orderService.ts`

```typescript
// ✅ 修复后代码
export async function createOrder(orderData: Order) {
  const connection = await getConnection();
  try {
    const result = await connection.query('INSERT INTO orders SET ?', orderData);
    return result.insertId;
  } finally {
    await releaseConnection(connection);
  }
}
```

### 优化连接池配置

**文件**: `src/db/connection.ts`

```typescript
import { createPool, Pool } from 'mysql2/promise';

const pool: Pool = createPool({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  connectionLimit: 20,           // 最大连接数
  waitForConnections: true,       // 等待可用连接
  connectionTimeout: 60000,       // 连接超时 60 秒
  acquireTimeout: 10000,          // 获取连接超时 10 秒
  idleTimeout: 300000,            // 空闲连接超时 5 分钟
  enableKeepAlive: true,          // 保持连接活跃
  keepAliveInitialDelay: 10000,   // 保活延迟 10 秒
});

export async function getConnection() {
  const connection = await pool.getConnection();
  logger.info('Connection acquired', {
    activeConnections: pool.pool.activeConnections(),
    totalConnections: pool.pool.totalConnections()
  });
  return connection;
}

export async function releaseConnection(connection: any) {
  if (connection) {
    await connection.release();
    logger.info('Connection released', {
      activeConnections: pool.pool.activeConnections(),
      totalConnections: pool.pool.totalConnections()
    });
  }
}

export function getPoolStats() {
  return {
    activeConnections: pool.pool.activeConnections(),
    totalConnections: pool.pool.totalConnections(),
    freeConnections: pool.pool.freeConnections()
  }
}
```

### 添加连接泄漏检测

**文件**: `src/middleware/connection-monitor.ts`

```typescript
import { getPoolStats } from '../db/connection';
import logger from '../utils/logger';

setInterval(() => {
  const stats = getPoolStats();

  if (stats.activeConnections > 15) {
    logger.warn('High number of active connections detected', stats);
  }

  if (stats.totalConnections > 18) {
    logger.error('Connection pool nearly exhausted!', stats);
    // 可以在此触发告警
  }
}, 30000); // 每 30 秒检查一次
```

### 运行测试（预期通过）
```bash
npm test -- connection-pool
```

预期结果: ✓ 所有测试通过
```

**ECC 响应 - IMPROVE 阶段（优化）**：

```
## 第 3 步：优化（IMPROVE）

### 创建通用的数据库操作包装器

创建文件: `src/db/query-wrapper.ts`

```typescript
import { getConnection, releaseConnection } from './connection';

export async function executeQuery<T>(
  query: string,
  params: any[] = []
): Promise<T> {
  const connection = await getConnection();
  try {
    const result = await connection.query(query, params);
    return result[0]; // mysql2 返回 [rows, fields]
  } finally {
    await releaseConnection(connection);
  }
}

export async function executeTransaction<T>(
  callback: (connection: any) => Promise<T>
): Promise<T> {
  const connection = await getConnection();
  try {
    await connection.beginTransaction();
    const result = await callback(connection);
    await connection.commit();
    return result;
  } catch (error) {
    await connection.rollback();
    throw error;
  } finally {
    await releaseConnection(connection);
  }
}
```

### 使用包装器服务重构服务

**重构后**: `src/services/userService.ts`

```typescript
import { executeQuery } from '../db/query-wrapper';

export async function getUser(userId: string) {
  // 不需要手动管理连接
  const users = await executeQuery<any[]>(
    'SELECT * FROM users WHERE id = ?',
    [userId]
  );
  return users[0];
}
```

### 运行测试确保重构没有破坏功能
```bash
npm test
```

预期结果: ✓ 所有测试通过
```

### 第 4 步：代码审查

```bash
用户: /code-review 审查连接泄漏修复的代码

ECC: [code-reviewer 代理]
```

**ECC 响应**：

```
## 代码审查报告

### ✅ 通过项
- ✓ 所有数据库操作正确释放连接
- ✓ 添加了 try-finally 确保连接释放
- ✓ 连接池配置合理
- ✓ 添加了监控和日志
- ✓ 创建了通用的查询包装器

### ⚠️ 建议改进

#### 1. 添加连接重用检测
**建议**: 检查是否有效重用了连接，避免频繁创建

```typescript
// 在包装器中添加连接重用统计
const connectionReuseStats = {
  reused: 0,
  created: 0
};
```

#### 2. 改进错误消息
**建议**: 提供更详细的错误信息

```typescript
if (stats.totalConnections > 18) {
  logger.error('Connection pool nearly exhausted!', {
    ...stats,
    timestamp: new Date().toISOString(),
    stackTrace: new Error().stack
  });
}
```

#### 3. 添加熔断机制
**建议**: 当连接池压力大时，拒绝新请求

```typescript
if (stats.activeConnections >= 15) {
  throw new Error('Service temporarily unavailable');
}
```

### 🔒 安全检查
- ✓ 无 SQL 注入风险（使用参数化查询）
- ✓ 异常处理完善
- ✓ 资源正确释放

### 总体评分: 9/10
- 修复完整性: 10/10
- 代码质量: 9/10
- 测试覆盖: 9/10
- 安全性: 9/10
```

### 第 5 步：验证修复

```bash
# 运行压力测试
npm run stress-test

# 监控连接池状态
# 在另一个终端运行
npm run monitor-connections

# 检查内存使用
# 检查日志中的连接泄漏警告
```

**预期结果**：
- ✓ 连接池稳定，无连接泄漏
- ✓ 内存使用稳定
- ✓ 压力测试通过
- ✓ 异常情况下连接正确释放

### 第 6 步：提交代码

```bash
# 添加所有修改的文件
git add src/services/userService.ts
git add src/services/orderService.ts
git add src/db/connection.ts
git add src/db/query-wrapper.ts
git add src/middleware/connection-monitor.ts
git add tests/unit/connection-pool.test.ts
git add tests/integration/connection-stress.test.ts

# 提交
git commit -m "fix: 修复数据库连接泄漏问题

- 确保所有数据库操作正确释放连接
- 添加 try-finally 保证异常时连接释放
- 优化连接池配置（超时、保活等）
- 添加连接池监控和日志
- 创建通用的查询包装器
- 添加连接泄漏检测测试
- 添加压力测试验证修复

修复问题：
- 内存泄漏
- 连接数持续增长
- \"Too many connections\" 错误

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>"
```

## 关键要点

### ✅ 最佳实践

1. **问题重现**：先理解问题的表现和原因
2. **规划修复**：制定系统性的修复方案
3. **测试先行**：先编写测试验证问题存在
4. **系统性修复**：不要只修复一个地方，要全面审查
5. **充分验证**：压力测试、监控、日志验证

### 🐛 问题排查技巧

1. **查看日志**：寻找异常和错误模式
2. **监控资源**：CPU、内存、连接数等
3. **压力测试**：重现生产环境问题
4. **代码审查**：系统性地检查类似问题

### 📊 测试策略

- ✓ 单元测试：连接正确释放
- ✓ 集成测试：连接池行为
- ✓ 压力测试：高并发场景
- ✓ 异常测试：异常情况下的资源释放

### 🔍 预防措施

1. **代码审查**：检查所有资源管理代码
2. **静态分析**：使用工具检测资源泄漏
3. **监控告警**：实时监控资源使用
4. **定期审计**：定期审查连接池配置

## 经验教训

### 💡 根本原因

连接泄漏的根本原因：
1. **异常处理不当**：异常时未释放资源
2. **代码路径遗漏**：某些代码路径未释放资源
3. **缺乏包装器**：每个操作都手动管理连接

### 🎯 防止复发的策略

1. **使用包装器**：创建通用的资源管理包装器
2. **强制测试**：添加资源泄漏检测测试
3. **代码规范**：制定资源管理规范
4. **持续监控**：添加实时监控和告警

### 📈 改进建议

1. **添加指标**：记录连接创建、使用、释放的指标
2. **自动化检测**：CI/CD 中添加资源泄漏检测
3. **文档化**：记录常见问题和解决方案
4. **团队培训**：分享修复经验给团队成员

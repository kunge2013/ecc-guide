# 场景 1: 用户注册登录流程

**类型：** 用户认证流程 E2E 测试
**工具：** Playwright
**优先级：** CRITICAL (关键用户流程)

## 测试目标

验证完整的用户注册和登录流程，包括：
1. 访问注册页面
2. 填写注册表单
3. 提交注册
4. 自动登录
5. 验证跳转到仪表盘

## 测试代码

```typescript
// file: e2e/auth/registration-flow.spec.ts

import { test, expect } from '@playwright/test';

test.describe('用户注册登录流程', () => {
  test('完整注册流程 - 从注册到仪表盘', async ({ page }) => {
    // 1. 访问注册页面
    await page.goto('/register');
    await expect(page).toHaveTitle('注册 - 我的App');

    // 2. 填写注册表单
    const timestamp = Date.now();
    const email = `test+${timestamp}@example.com`;

    await page.fill('[name="username"]', `testuser${timestamp}`);
    await page.fill('[name="email"]', email);
    await page.fill('[name="password"]', 'SecurePass123!');
    await page.fill('[name="confirmPassword"]', 'SecurePass123!');

    // 3. 提交注册
    await page.click('[type="submit"]');

    // 4. 验证重定向到仪表盘（自动登录）
    await expect(page).toHaveURL(/\/dashboard/);
    await expect(page.locator('h1')).toContainText('仪表盘');

    // 5. 验证用户信息显示
    await expect(page.locator('[data-testid="user-email"]')).toContainText(email);
  });

    test('表单验证 - 必填字段', async ({ page }) => {
    await page.goto('/register');

    // 不填写任何字段直接提交
    await page.click('[type="submit"]');

    // 验证错误消息
    await expect(page.locator('[data-testid="error-username"]')).toContainText('用户名不能为空');
    await expect(page.locator('[data-testid="error-email"]')).toContainText('邮箱不能为空');
    await expect(page.locator('[data-testid="error-password"]')).toContainText('密码不能为空');
  });

    test('表单验证 - 密码强度', async ({ page }) => {
    await page.goto('/register');

    await page.fill('[name="username"]', 'testuser');
    await page.fill('[name="email"]', 'test@example.com');
    await page.fill('[name="password"]', 'weak'); // 弱密码
    await page.fill('[name="confirmPassword"]', 'weak');

    await page.click('[type="submit"]');

    // 验证密码强度错误
    await expect(page.locator('[data-testid="error-password"]')).toContainText('密码强度不足');
  });
});
```

## 关键要点

### 测试策略

1. **完整流程验证** - 测试从注册到成功的完整路径
2. **表单验证** - 确保客户端验证正常工作
3. **自动登录** - 验证注册后自动登录功能

### 最佳实践

- ✅ 使用唯一邮箱避免冲突
- ✅ 验证重定向行为
- ✅ 检查用户信息显示
- ✅ 测试表单验证
- ✅ 使用 data-testid 属性选择器

### 性能考虑

```typescript
test('注册流程性能验证', async ({ page }) => {
  const startTime = Date.now();

  await page.goto('/register');
  await page.fill('[name="username"]', 'testuser');
  await page.fill('[name="email"]', 'test@example.com');
  await page.fill('[name="password"]', 'SecurePass123!');
  await page.fill('[name="confirmPassword"]', 'SecurePass123!');
  await page.click('[type="submit"]');
  await page.waitForURL(/\/dashboard/);

  const endTime = Date.now();
  const duration = endTime - startTime;

  // ✅ 注册流程应在 3 秒内完成
  expect(duration).toBeLessThan(3000);
});
```

## 数据清理

```typescript
// 测试后清理测试用户
test.afterEach(async ({ request }) => {
  // 通过 API 清理测试数据
  await request.post('/api/test/cleanup', {
    data: { username: 'testuser' }
  });
});
```

---

**场景价值：** 这是所有应用的核心流程，必须通过 E2E 测试验证。确保用户可以成功注册并正常使用应用。

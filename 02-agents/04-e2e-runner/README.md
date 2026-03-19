# e2e-runner 代理

## 概述

e2e-runner 代理是 ECC 的核心代理之一，专门负责端到端（E2E）测试的生成和执行。E2E 测试模拟真实用户操作，验证完整的应用流程。

## 代理职责

e2e-runner 代理的主要职责包括：

1. **生成 E2E 测试** - 根据用户流程自动生成测试用例
2. **执行 E2E 测试** - 使用浏览器自动化工具运行测试
3. **验证用户流程** - 确保关键用户路径正常工作
4. **性能测试** - 测量用户操作的响应时间
5. **跨浏览器测试** - 在多个浏览器上验证兼容性
6. **测试报告** - 生成详细的测试报告和截图
7. **回归测试** - 在代码变更后自动运行 E2E 测试

## 触发条件

e2e-runner 代理在以下情况下被触发：

1. **用户执行 `/e2e` 命令** - 手动触发 E2E 测试
2. **代码提交后** - 代码提交后自动运行 E2E 测试
3. **关键功能完成** - 关键功能完成后自动生成并运行测试
4. **PR 创建** - 在 PR 中运行 E2E 测试验证变更

## 支持的工具

e2e-runner 代理支持多种浏览器自动化工具：

| 工具 | 优先级 | 特点 |
|------|--------|------|
| **Vercel Agent Browser** | 1 | 最先进，AI 驱动，无需维护 |
| **Playwright** | 2 | 现代化，多浏览器支持，测试隔离好 |
| **Cypress** | 3 | 开发者友好，实时重载，时间旅行调试 |

## 工作流程

### 标准 E2E 测试流程

```
1. 分析用户流程
   ↓
2. 识别关键路径
   ↓
3. 生成测试用例
   ↓
4. 设置测试环境
   ↓
5. 执行测试
   ↓
6. 收集结果
   ↓
7. 生成报告
   ↓
8. 分析失败原因
```

### 详细步骤

#### 步骤 1: 分析用户流程

e2e-runner 分析应用的典型用户流程：

- 用户注册流程
- 用户登录流程
- 创建资源流程
- 编辑资源流程
- 删除资源流程
- 搜索和筛选流程
- 支付流程（如果适用）

**示例输出**:
```markdown
## 识别的用户流程

1. 用户注册 → 邮箱验证 → 登录
2. 创建任务 → 编辑任务 → 删除任务
3. 创建项目 → 添加成员 → 分配任务
4. 查看任务列表 → 筛选任务 → 导出任务
```

#### 步骤 2: 识别关键路径

确定需要测试的关键用户路径：

```markdown
## 关键路径

### 高优先级（必须测试）
- 完整的注册到登录流程
- 创建和查看任务
- 用户退出登录

### 中优先级
- 任务编辑和删除
- 项目管理
- 搜索和筛选

### 低优先级
- 用户设置
- 通知管理
- 导出功能
```

#### 步骤 3: 生成测试用例

自动生成测试用例代码：

**Playwright 示例**:
```typescript
// e2e/user-registration.spec.ts
import { test, expect } from '@playwright/test';

test.describe('用户注册流程', () => {
  test('应该成功注册新用户', async ({ page }) => {
    // 访问注册页面
    await page.goto('/register');

    // 填写注册表单
    await page.fill('[name="username"]', 'testuser');
    await page.fill('[name="email"]', 'test@example.com');
    await page.fill('[name="password"]', 'SecurePass123!');
    await page.fill('[name="confirmPassword"]', 'SecurePass123!');

    // 提交表单
    await page.click('button[type="submit"]');

    // 验证重定向到登录页面
    await expect(page).toHaveURL('/login');
    await expect(page.locator('.success-message')).toContainText('注册成功');
  });

  test('应该拒绝重复的用户名', async ({ page }) => {
    await page.goto('/register');

    await page.fill('[name="username"]', 'existinguser');
    await page.fill('[name="email"]', 'new@example.com');
    await page.fill('[name="password"]', 'SecurePass123!');
    await page.fill('[name="confirmPassword"]', 'SecurePass123!');

    await page.click('button[type="submit"]');

    // 验证错误消息
    await expect(page.locator('.error-message')).toContainText('用户名已被使用');
  });

  test('应该验证密码强度', async ({ page }) => {
    await page.goto('/register');

    await page.fill('[name="username"]', 'testuser');
    await page.fill('[name="email"]', 'test@example.com');
    await page.fill('[name="password"]', 'weak');  // 弱密码
    await page.fill('[name="confirmPassword"]', 'weak');

    await page.click('button[type="submit"]');

    // 验证密码强度错误
    await expect(page.locator('.error-message')).toContainText('密码强度不足');
  });
});

test.describe('用户登录流程', () => {
  test('应该成功登录', async ({ page }) => {
    await page.goto('/login');

    await page.fill('[name="username"]', 'testuser');
    await page.fill('[name="password"]', 'SecurePass123!');

    await page.click('button[type="submit"]');

    // 验证重定向到仪表板
    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('.user-greeting')).toContainText('你好，testuser');
  });

  test('应该拒绝无效凭证', async ({ page }) => {
    await page.goto('/login');

    await page.fill('[name="username"]', 'nonexistent');
    await page:fill('[name="password"]', 'wrongpassword');

    await page.click('button[type="submit"]');

    // 验证错误消息
    await expect(page.locator('.error-message')).toContainText('用户名或密码错误');
  });
});

test.describe('任务管理流程', () => {
  test.beforeEach(async ({ page }) => {
    // 登录
    await page.goto('/login');
    await page.fill('[name="username"]', 'testuser');
    await page.fill('[name="password"]', 'SecurePass123!');
    await page.click('button[type="submit"]');
    await page.waitForURL('/dashboard');
  });

  test('应该创建并查看任务', async ({ page }) => {
    // 导航到任务创建页面
    await page.click('text=创建任务');
    await expect(page).toHaveURL('/tasks/create');

    // 填写任务信息
    await page.fill('[name="title"]', '测试任务');
    await page.fill('[name="description"]', '这是一个测试任务');
    await page.selectOption('[name="priority"]', 'high');

    // 提交任务
    await page.click('button[type="submit"]');

    // 验证任务创建成功
    await expect(page).toHaveURL('/tasks');
    await expect(page.locator('.task-item')).toContainText('测试任务');
    await expect(page.locator('.task-item .priority')).toContainText('高');
  });

  test('应该编辑任务', async ({ page }) => {
    // 导航到任务列表
    await page.goto('/tasks');

    // 点击编辑按钮
    await page.click('.task-item:first-child button.edit');
    await expect(page).toHaveURL(/\/tasks\/\d+\/edit/);

    // 编辑任务
    await page.fill('[name="title"]', '更新后的任务');
    await page.click('button[type="submit"]');

    // 验证更新成功
    await expect(page).toHaveURL('/tasks');
    await expect(page.locator('.task-item:first-child')).toContainText('更新后的任务');
  });

  test('应该删除任务', async ({ page }) => {
    await page.goto('/tasks');

    // 获取任务数量
    const taskCountBefore = await page.locator('.task-item').count();

    // 点击删除按钮
    await page.click('.task-item:first-child button.delete');

    // 确认删除
    await page.click('button.confirm-delete');

    // 验证任务已删除
    const taskCountAfter = await page.locator('.task-item').count();
    expect(taskCountAfter).toBe(taskCountBefore - 1);
  });
});
```

#### 步骤 4: 设置测试环境

配置测试环境：

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,

  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },

  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
    {
      name: 'Mobile Safari',
      use: { ...devices['iPhone 12'] },
    },
  ],

  webServer: {
    command: 'npm run start',
    port: 3000,
    reuseExistingServer: !process.env.CI,
  },
});
```

#### 步骤 5: 执行测试

运行 E2E 测试：

```bash
# 运行所有 E2E 测试
npm run test:e2e

# 运行特定测试
npx playwright test user-registration.spec.ts

# 运行特定浏览器
npx playwright test --project=chromium

# 以可视化模式运行（开发时）
npx playwright test --ui

# 生成 HTML 报告
npx playwright show-report
```

#### 步骤 6: 收集结果

收集测试结果和指标：

```markdown
## 测试结果摘要

### 总体统计
- 总测试数: 15
- 通过: 13
- 失败: 2
- 跳过: 0
- 覆盖率: 86.7%

### 浏览器兼容性
| 浏览器 | 通过 | 失败 | 状态 |
|--------|------|------|------|
| Chrome | 15 | 0 | ✓ |
| Firefox | 15 | 0 | ✓ |
| Safari | 14 | 1 | ⚠ |
| Mobile Chrome | 15 | 0 | ✓ |
| Mobile Safari | 13 | 2 | ❌ |

### 性能指标
| 测试用例 | 平均响应时间 | 最长响应时间 | 状态 |
|----------|--------------|--------------|------|
| 用户注册 | 450ms | 620ms | ✓ |
| 用户登录 | 280ms | 350ms | ✓ |
| 创建任务 | 520ms | 780ms | ⚠ |
| 查看任务列表 | 180ms | 240ms | ✓ |
| 编辑任务 | 320ms | 410ms | ✓ |
```

#### 步骤 7: 生成报告

生成详细的测试报告：

```markdown
# E2E 测试报告

## 执行信息
- 开始时间: 2024-03-19 10:00:00
- 结束时间: 2024-03-19 10:05:23
- 总耗时: 5分23秒
- 浏览器: Chrome, Firefox, Safari, Mobile Chrome, Mobile Safari

## 失败的测试

### 1. 应该在 Mobile Safari 上正确显示任务列表
**文件**: `e2e/task-management.spec.ts:45`
**浏览器**: Mobile Safari
**错误**: 超时 - 等待元素 `.task-item` 超时

**截图**: [失败截图](reports/screenshots/mobile-safari-failure.png)

**视频**: [失败视频](reports/videos/mobile-safari-failure.webm)

**日志**:
```
[error] Timeout 30000ms exceeded.
[info] Expected: found .task-item
[info] Received: not found
```

**建议**: 检查 Mobile Safari 的响应式布局，可能存在 CSS 兼容性问题

---

### 2. 应该在 Mobile Safari 上正确提交表单
**文件**: `e2e/user-registration.spec.ts:78`
**浏览器**: Mobile Safari
**错误**: 元素点击被拦截

**截图**: [失败截图](reports/screenshots/mobile-safari-failure-2.png)

**建议**: 可能是软键盘弹出导致元素位置变化，需要添加等待

---

## 性能警告

### 慢速测试（响应时间 > 500ms）

1. **创建任务** - 平均响应时间: 520ms
   - 最长响应时间: 780ms
   - 建议: 优化表单提交逻辑，考虑使用 loading 状态

---

## 测试覆盖率

| 用户流程 | 测试用例 | 覆盖率 |
|----------|----------|--------|
| 用户注册 | 3 | 100% |
| 用户登录 | 2 | 100% |
| 任务管理 | 7 | 100% |
| 项目管理 | 2 | 67% |
| 搜索筛选 | 1 | 50% |

**总覆盖率**: 83.3%
```

#### 步骤 8: 分析失败原因

分析测试失败的根本原因：

```markdown
## 失败原因分析

### 根本原因

1. **Mobile Safari CSS 兼容性问题**
   - 问题: 某些 CSS 属性在 Safari 上支持不完整
   - 影响: 任务列表在 Mobile Safari 上无法正确渲染
   - 解决方案: 添加 Safari 特定的 CSS 前缀或降级方案

2. **软键盘交互问题**
   - 问题: Mobile Safari 软键盘弹出时改变视口高度
   - 影响: 表单提交按钮位置改变，点击被拦截
   - 解决方案: 添加 `visualViewport` 监听和适当的等待逻辑

### 建议修复

1. 添加 Mobile Safari 特定的测试配置
2. 优化响应式布局的兼容性
3.为移动设备添加额外的等待逻辑
```

## 使用场景

### 场景 1: 新功能验证

**用户输入**:
```
我刚完成了任务搜索功能筛选，请生成 E2E 测试
```

**e2e-runner 响应**:
```markdown
好的！让我为任务搜索筛选功能生成 E2E 测试。

## 分析用户流程

识别的关键路径：
1. 搜索任务（按标题）
2. 筛选任务（按状态）
3. 组合搜索和筛选
4. 清空搜索条件

## 生成测试用例

正在生成测试代码...

[测试代码]

## 执行测试

正在运行测试...

[测试结果]
```

### 场景 2: 回归测试

**用户输入**:
```
运行完整的 E2E 回归测试套件
```

**e2e-runner 响应**:
```markdown
好的！运行完整的 E2E 回归测试套件。

## 测试配置

- 测试范围: 全部
- 浏览器: Chrome, Firefox, Safari
- 并发数: 4

## 执行测试

正在运行测试...

[测试结果]
```

### 场景 3: 跨浏览器测试

**用户输入**:
```
在所有支持的浏览器上测试用户注册流程
```

**e2e-runner 响应**:
```markdown
好的！在所有支持的浏览器上测试用户注册流程。

## 测试浏览器

- Desktop Chrome
- Desktop Firefox
- Desktop Safari
- Mobile Chrome (Pixel 5)
- Mobile Safari (iPhone 12)

## 执行测试

[跨浏览器测试结果]
```

## 代理特性

### 智能测试生成

e2e-runner 代理会：

- **分析路由结构** - 自动识别页面和路由
- **检测表单** - 自动识别表单字段和验证规则
- **识别交互元素** - 自动检测按钮、链接、下拉菜单等
- **生成完整流程** - 自动连接相关页面形成完整流程

### 智能等待

e2e-runner 代理会：

- **等待元素可见** - 确保元素出现在页面上
- **等待元素可点击** - 确保元素可以交互
- **等待网络请求** - 等待 API 请求完成
- **等待动画完成** - 等待页面动画和过渡完成

### 失败处理

e2e-runner 代理会：

- **自动截图** - 测试失败时自动截图
- **录制视频** - 记录失败的测试过程
- **生成日志** - 记录详细的错误日志
- **提供建议** - 为每次失败提供修复建议

## 与其他代理的协作

### 与 tdd-guide 协作

```
tdd-guide 指导单元测试和集成测试
    ↓
e2e-runner 生成和运行 E2E 测试
    ↓
完整的测试覆盖
```

### 与 code-reviewer 协作

```
e2e-runner 发现测试失败
    ↓
code-reviewer 审查相关代码
    ↓
修复问题
```

## 最佳实践

### 对于开发者

1. **保持测试稳定** - 避免使用硬编码的等待时间
2. **使用 Page Objects** - 使用页面对象模式提高可维护性
3. **测试真实场景** - 测试真实的用户行为，不是实现细节
4. **保持测试独立** - 每个测试应该独立运行，不依赖其他测试
5. **定期运行** - 在 CI/CD 中定期运行 E2E 测试

### 测试金字塔

```
        /\
       /E2E\      少量 (10%) - 关键用户流程
      /------\
     / 集成  \     适量 (30%) - 组件交互
    /----------\
   /   单元    \   大量 (60%) - 函数和方法
  /--------------\
```

### 测试维护建议

- **定期更新测试** - 随着 UI 变化更新测试
- **移过时测试** - 删除不再适用的测试
- **监控测试时长** - 保持测试快速运行
- **分析失败模式** - 识别常见的失败原因

## 学习资源

- [/e2e 命令说明](../../01-commands/04-e2e/README.md)
- [Playwright 文档](https://playwright.dev)
- [示例：电商网站 E2E 测试](./examples/ecommerce-e2e.md)
- [示例：移动应用 E2E 测试](./examples/mobile-e2e.md)

## 下一步

- 查看 [电商网站 E2E 测试示例](./examples/ecommerce-e2e.md)
- 或返回 [代理总览](../README.md)

# E2E Runner Agent 示例集合

这个目录包含了 3 个经典的 E2E（端到端）测试场景示例。

## 📚 示例列表

### 1. [用户注册登录流程](./01-user-registration-flow.md)
**类型：** 用户认证流程 E2E 测试
**工具：** Playwright
**优先级：** CRITICAL (关键用户流程)

**测试内容：**
- 完整注册流程（注册→自动登录→仪表盘）
- 表单验证（必填字段、密码强度）
- 邮箱唯一性验证

**关键要点：**
- 使用唯一邮箱避免冲突
- 验证重定向行为
- 检查用户信息显示
- 数据清理

---

### 2. [电商购物流程](./02-ecommerce-shopping-flow.md)
**类型：** 复杂业务流程 E2E 测试
**工具：** Playwright
**优先级：** HIGH (核心业务流程)

**测试内容：**
- 完整购物流程（浏览→添加→结账→支付）
- 购物车操作（添加、更新、删除）
- 订单确认和查看

**关键要点：**
- 模拟真实用户行为
- 验证每个状态变化
- 使用测试数据（测试卡号）
- 网络错误处理

---

### 3. [响应式设计测试](./03-responsive-design.md)
**类型：** 跨设备兼容性 E2E 测试
**工具：** Playwright
**优先级：** MEDIUM (用户体验)

**测试内容：**
- 桌面端布局和交互
- 移动端适配和触摸
- 断点切换测试

**关键要点：**
- 多设备测试
- 验证交互方式正确
- 视觉回归测试
- 性能验证

---

## 🎯 使用建议

### 学习路径

1. **核心流程优先** - 从示例 1 开始
   - 这是最基础的用户流程
   - 展示 E2E 测试的基本模式

2. **业务流程** - 学习示例 2
   - 复杂状态管理
   - 多步骤流程验证

3. **兼容性测试** - 学习示例 3
   - 跨设备测试
   - 响应式设计验证

### E2E 测试原则

| 原则 | 说明 | 示例 |
|------|------|------|
| 真实用户行为 | 模拟真实用户操作 | 点击、输入、等待 |
| 完整流程 | 测试端到端流程 | 从开始到完成 |
| 状态验证 | 验证每个状态变化 | 数量更新、重定向 |
| 数据隔离 | 每次测试独立 | 清理测试数据 |
| 可靠性 | 处理异步和时序 | 使用 waitFor |

### 测试金字塔

```
       E2E 测试 (10%)
          ↑
     集成测试 (20%)
          ↑
    单元测试 (70%)
```

E2E 测试应该：
- 关注关键用户流程
- 验证系统集成
- 不要覆盖所有边界条件（单元测试负责）

---

## 📊 E2E 测试最佳实践

### 1. 测试数据管理

```typescript
// 使用测试工厂模式
class TestDataFactory {
  static createTestUser() {
    return {
      username: `testuser_${Date.now()}`,
      email: `test_${Date.now()}@example.com`,
      password: 'SecurePass123!'
    };
  }

  static cleanupTestData(user) {
    // 清理测试数据
  }
}
```

### 2. 页面对象模式

```typescript
// 封装页面交互
class LoginPage {
  constructor(page) {
    this.page = page;
  }

  async login(email, password) {
    await this.page.fill('[name="email"]', email);
    await this.page.fill('[name="password"]', password);
    await this.page.click('[type="submit"]');
    await this.page.waitForURL(/\/dashboard/);
  }
}
```

### 3. 测试复用

```typescript
// 测试辅助函数
async fillRegistrationForm(page, data) {
  await page.fill('[name="username"]', data.username);
  await page.fill('[name="email"]', data.email);
  await page.fill('[name="password"]', data.password);
  await page.fill('[name="confirmPassword"]', data.confirmPassword);
}

// 在多个测试中使用
test('注册流程', async ({ page }) => {
  const userData = TestDataFactory.createTestUser();
  await page.goto('/register');
  await fillRegistrationForm(page, userData);
  await page.click('[type="submit"]');
  // ...
});
```

### 4. 测试组织

```
e2e/
├── auth/                 # 认证相关测试
│   ├── login.spec.ts
│   └── registration.spec.ts
├── ecommerce/            # 电商功能测试
│   ├── shopping.spec.ts
│   └── checkout.spec.ts
├── responsive/            # 响应式测试
│   └── design.spec.ts
└── helpers/               # 测试辅助
    ├── testDataFactory.ts
    └── pages/
        ├── LoginPage.ts
        └── DashboardPage.ts
```

---

## 🔗 相关资源

- [E2E Runner Agent](../README.md) - E2E Runner Agent 指南
- [Playwright 文档](https://playwright.dev) - Playwright 官方文档
- [测试最佳实践](../../../rules/common/testing.md) - 项目测试规范

---

**提示：** E2E 测试运行较慢，应该专注于关键用户流程。其他测试类型应该由单元测试和集成测试覆盖。

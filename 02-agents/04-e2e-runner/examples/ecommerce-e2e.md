# 示例：电商网站 E2E 测试

这个示例展示 e2e-runner 如何为电商网站生成和运行完整的端到端测试。

## 项目概览

**应用**: 在线商城
**技术栈**: Next.js + TypeScript
**关键功能**:
- 用户注册和登录
- 商品浏览和搜索
- 购物车管理
- 订单创建和支付
- 订单查看

## 生成的 E2E 测试

### 1. 用户流程测试

```typescript
// e2e/user-flow.spec.ts
import { test, expect } from '@playwright/test';

test.describe('用户注册和登录流程', () => {
  test('应该完成完整的注册流程', async ({ page }) => {
    // 访问注册页面
    await page.goto('/register');

    // 填写注册表单
    await page.fill('[name="username"]', 'shopper123');
    await page.fill('[name="email"]', 'shopper@example.com');
    await page.fill('[name="password"]', 'SecurePass123!');
    await page.fill('[name="confirmPassword"]', 'SecurePass123!');
    await page.fill('[name="fullName"]', 'Jane Shopper');
    await page.fill('[name="phone"]', '1234567890');

    // 提交表单
    await page.click('button[type="submit"]');

    // 等待成功消息
    await expect(page.locator('.success-message')).toContainText('注册成功！');

    // 验证重定向到登录页面
    await expect(page).toHaveURL(/\/login/);
  });

  test('应该成功登录', async ({ page }) => {
    // 访问登录页面
    await page.goto('/login');

    // 填写登录表单
    await page.fill('[name="email"]', 'shopper@example.com');
    await page.fill('[name="password"]', 'SecurePass123!');

    // 提交表单
    await page.click('button[type="submit"]');

    // 等待重定向到首页
    await expect(page).toHaveURL('/');

    // 验证用户已登录
    await expect(page.locator('.user-greeting')).toContainText('你好，Jane Shopper');
    await expect(page.locator('button:has-text("登录")')).not.toBeVisible();
  });

  test('应该成功退出登录', async ({ page }) => {
    // 先登录
    await page.goto('/login');
    await page.fill('[name="email"]', 'shopper@example.com');
    await page.fill('[name="password"]', 'SecurePass123!');
    await page.click('button[type="submit"]');
    await page.waitForURL('/');

    // 点击退出按钮
    await page.click('button:has-text("退出")');

    // 验证退出成功
    await expect(page.locator('button:has-text("登录")')).toBeVisible();
    await expect(page.locator('.user-greeting')).not.toBeVisible();
  });
});
```

### 2. 商品浏览和搜索测试

```typescript
// e2e/product-browsing.spec.ts
import { test, expect } from '@playwright/test';

test.describe('商品浏览和搜索', () => {
  test.beforeEach(async ({ page }) => {
    // 登录
    await page.goto('/login');
    await page.fill('[name="email"]', 'shopper@example.com');
    await page.fill('[name="password"]', 'SecurePass123!');
    await page.click('button[type="submit"]');
    await page.waitForURL('/');
  });

  test('应该显示商品列表', async ({ page }) => {
    // 访问商品列表页面
    await page.click('a:has-text("商品")');
    await expect(page).toHaveURL('/products');

    // 验证商品列表显示
    const productCards = page.locator('.product-card');
    await expect(productCards).toHaveCount(20);  // 假设每页显示 20 个商品

    // 验证第一个商品的信息
    await expect(productCards.first().locator('.product-name')).toBeVisible();
    await expect(productCards.first().locator('.product-price')).toBeVisible();
    await expect(productCards.first().locator('.product-image')).toBeVisible();
  });

  test('应该正确搜索商品', async ({ page }) => {
    // 访问商品列表页面
    await page.goto('/products');

    // 输入搜索关键词
    await page.fill('[name="search"]', 'iPhone');
    await page.click('button:has-text("搜索")');

    // 等待搜索结果加载
    await page.waitForSelector('.product-card');

    // 验证搜索结果
    const productNames = page.locator('.product-name');
    const count = await productNames.count();

    for (let i = 0; i < count; i++) {
      const name = await productNames.nth(i).textContent();
      expect(name?.toLowerCase()).toContain('iphone');
    }
  });

  test('应该按类别筛选商品', async ({ page }) => {
    // 访问商品列表页面
    await page.goto('/products');

    // 选择类别
    await page.click('button:has-text("类别")');
    await page.click('button:has-text("手机")');

    // 验证筛选结果
    await page.waitForSelector('.product-card');

    // 验证面包屑导航
    await expect(page.locator('.breadcrumb')).toContainText('手机');
  });

  test('应该查看商品详情', async ({ page }) => {
    // 访问商品列表页面
    await page.goto('/products');

    // 点击第一个商品
    await page.click('.product-card:first-child');

    // 验证商品详情页面
    await expect(page.locator('.product-detail')).toBeVisible();
    await expect(page.locator('.product-title')).toBeVisible();
    await expect(page.locator('.product-description')).toBeVisible();
    await expect(page.locator('.product-price')).toBeVisible();
    await expect(page.locator('button:has-text("加入购物车")')).toBeVisible();
  });

  test('应该按价格排序商品', async ({ page }) => {
    // 访问商品列表页面
    await page.goto('/products');

    // 选择价格排序（从低到高）
    await page.selectOption('select[name="sort"]', 'price-asc');

    // 等待排序完成
    await page.waitForSelector('.product-card');

    // 验证价格顺序
    const prices = await page.locator('.product-price').allTextContents();
    const numericPrices = prices.map(p => parseFloat(p.replace(/[^0-9.]/g, '')));

    for (let i = 1; i < numericPrices.length; i++) {
      expect(numericPrices[i]).toBeGreaterThanOrEqual(numericPrices[i - 1]);
    }
  });
});
```

### 3. 购物车测试

```typescript
// e2e/shopping-cart.spec.ts
import { test, expect } from '@playwright/test';

test.describe('购物车管理', () => {
  test.beforeEach(async ({ page }) => {
    // 登录
    await page.goto('/login');
    await page.fill('[name="email"]', 'shopper@example.com');
    await page.fill('[name="password"]', 'SecurePass123!');
    await page.click('button[type="submit"]');
    await page.waitForURL('/');
  });

  test('应该添加商品到购物车', async ({ page }) => {
    // 访问商品详情页面
    await page.goto('/products/1');

    // 点击"加入购物车"按钮
    await page.click('button:has-text("加入购物车")');

    // 验证成功消息
    await expect(page.locator('.toast').filter({ hasText: '已添加到购物车' })).toBeVisible();

    // 验证购物车数量更新
    await expect(page.locator('.cart-count')).toContainText('1');
  });

  test('应该查看购物车内容', async ({ page }) => {
    // 添加商品到购物车
    await page.goto('/products/1');
    await page.click('button:has-text("加入购物车")');

    // 访问购物车页面
    await page.click('a:has-text("购物车")');
    await expect(page).toHaveURL('/cart');

    // 验证购物车内容
    await expect(page.locator('.cart-item')).toHaveCount(1);

    const cartItem = page.locator('.cart-item').first();
    await expect(cartItem.locator('.product-name')).toBeVisible();
    await expect(cartItem.locator('.product-price')).toBeVisible();
    await expect(cartItem.locator('.quantity')).toContainText('1');
  });

  test('应该更新商品数量', async ({ page }) => {
    // 添加商品到购物车
    await page.goto('/products/1');
    await page.click('button:has-text("加入购物车")');

    // 访问购物车页面
    await page.goto('/cart');

    // 增加数量
    await page.click('.cart-item:first-child button:has-text("+")');

    // 验证数量更新
    await expect(page.locator('.cart-item:first-child .quantity')).toContainText('2');

    // 验证总价更新
    const totalPrice = await page.locator('.total-price').textContent();
    expect(totalPrice).toBeTruthy();
  });

  test('应该从购物车删除商品', async ({ page }) => {
    // 添加商品到购物车
    await page.goto('/products/1');
    await page.click('button:has-text("加入购物车")');

    // 访问购物车页面
    await page.goto('/cart');

    // 删除商品
    await page.click('.cart-item:first-child button:has-text("删除")');

    // 确认删除
    await page.click('button:has-text("确认")');

    // 验证购物车为空
    await expect(page.locator('.cart-item')).toHaveCount(0);
    await expect(page.locator('.empty-cart')).toBeVisible();
  });

  test('应该计算购物车总价', async ({ page }) => {
    // 添加多个商品到购物车
    await page.goto('/products/1');
    await page.click('button:has-text("加入购物车")');

    await page.goto('/products/2');
    await page.click('button:has-text("加入购物车")');

    // 访问购物车页面
    await page.goto('/cart');

    // 获取商品价格
    const price1 = parseFloat(
      (await page.locator('.cart-item').nth(0).locator('.product-price').textContent()) || '0'
    );
    const price2 = parseFloat(
      (await page.locator('.cart-item').nth(1).locator('.product-price').textContent()) || '0'
    );

    // 验证总价
    const expectedTotal = price1 + price2;
    const actualTotal = parseFloat(
      (await page.locator('.total-price').textContent()) || '0'
    );

    expect(actualTotal).toBeCloseTo(expectedTotal, 2);
  });
});
```

### 4. 订单创建和支付测试

```typescript
// e2e/order-payment.spec.ts
import { test, expect } from '@playwright/test';

test.describe('订单创建和支付', () => {
  test.beforeEach(async ({ page }) => {
    // 登录并添加商品到购物车
    await page.goto('/login');
    await page.fill('[name="email"]', 'shopper@example.com');
    await page.fill('[name="password"]', 'SecurePass123!');
    await page.click('button[type="submit"]');
    await page.waitForURL('/');

    await page.goto('/products/1');
    await page.click('button:has-text("加入购物车")');
  });

  test('应该创建订单', async ({ page }) => {
    // 进入结账流程
    await page.click('a:has-text("购物车")');
    await page.click('button:has-text("结账")');

    // 填写收货地址
    await page.fill('[name="fullName"]', 'Jane Shopper');
    await page.fill('[name="address"]', '123 Main St');
    await page.fill('[name="city"]', 'New York');
    await page.fill('[name="state"]', 'NY');
    await page.fill('[name="zipCode"]', '10001');
    await page.fill('[name="phone"]', '1234567890');

    // 提交订单
    await page.click('button:has-text("创建订单")');

    // 验证订单创建成功
    await expect(page).toHaveURL(/\/orders\/\d+/);
    await expect(page.locator('.order-success')).toBeVisible();
  });

  test('应该使用信用卡支付', async ({ page }) => {
    // 进入结账流程
    await page.goto('/cart');
    await page.click('button:has-text("结账")');

    // 填写收货地址
    await page.fill('[name="fullName"]', 'Jane Shopper');
    await page.fill('[name="address"]', '123 Main St');
    await page.fill('[name="city"]', 'New York');
    await page.fill('[name="state"]', 'NY');
    await page.fill('[name="zipCode"]', '10001');
    await page.fill('[name="phone"]', '1234567890');

    // 选择支付方式
    await page.click('input[value="credit-card"]');

    // 填写信用卡信息（测试卡号）
    await page.fill('[name="cardNumber"]', '4242424242424242');
    await page.selectOption('[name="expiryMonth"]', '12');
    await page.selectOption('[name="expiryYear"]', '2025');
    await page.fill('[name="cvv"]', '123');
    await page.fill('[name="cardholderName"]', 'JANE SHOPPER');

    // 提交支付
    await page.click('button:has-text("支付")');

    // 验证支付成功
    await expect(page.locator('.payment-success')).toBeVisible();
    await expect(page.locator('.order-status')).toContainText('已支付');
  });

  test('应该查看订单历史', async ({ page }) => {
    // 创建一个订单
    await page.goto('/cart');
    await page.click('button:has-text("结账")');
    await page.fill('[name="fullName"]', 'Jane Shopper');
    await page.fill('[name="address"]', '123 Main St');
    await page.fill('[name="city"]', 'New York');
    await page.fill('[name="state"]', 'NY');
    await page.fill('[name="zipCode"]', '10001');
    await page.fill('[name="phone"]', '1234567890');
    await page.click('button:has-text("创建订单")');

    // 访问订单历史页面
    await page.click('a:has-text("我的订单")');
    await expect(page).toHaveURL('/orders');

    // 验证订单列表
    await expect(page.locator('.order-item')).toHaveCountGreaterThan(0);

    // 点击第一个订单
    await page.click('.order-item:first-child');

    // 验证订单详情
    await expect(page.locator('.order-detail')).toBeVisible();
    await expect(page.locator('.order-number')).toBeVisible();
    await expect(page.locator('.order-date')).toBeVisible();
    await expect(page.locator('.order-items')).toBeVisible();
    await expect(page.locator('.order-total')).toBeVisible();
  });
});
```

## 配置文件

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
    video: 'retain-on-failure',
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
    command: 'npm run dev',
    port: 3000,
    reuseExistingServer: !process.env.CI,
    timeout: 120 * 1000,
  },
});
```

## 运行测试

```bash
# 运行所有 E2E 测试
npm run test:e2e

# 运行特定测试套件
npx playwright test user-flow.spec.ts

# 以可视化模式运行（开发时）
npx playwright test --ui

# 生成 HTML 报告
npx playwright show-report

# 在特定浏览器上运行
npx playwright test --project=chromium
```

## 测试报告示例

```markdown
# E2E 测试报告

## 执行摘要
- 开始时间: 2024-03-19 14:00:00
- 结束时间: 2024-03-19 14:08:45
- 总耗时: 8分45秒
- 总测试数: 15
- 通过: 14
- 失败: 1
- 跳过: 0
- 成功率: 93.3%

## 浏览器兼容性

| 浏览器 | 通过 | 失败 | 状态 |
|--------|------|------|------|
| Chrome | 15 | 0 | ✓ |
| Firefox | 15 | 0 | ✓ |
| Safari | 15 | 0 | ✓ |
| Mobile Chrome | 15 | 0 | ✓ |
| Mobile Safari | 14 | 1 | ⚠ |

## 失败的测试

### 1. 应该使用信用卡支付（Mobile Safari）
**文件**: `e2e/order-payment.spec.ts:87`
**浏览器**: Mobile Safari
**错误**: 元素不可点击，被软键盘遮挡

**截图**: [失败截图](reports/screenshots/mobile-safari-failure.png)

**建议**: 添加等待软键盘消失的逻辑，或调整页面布局

## 性能指标

| 测试用例 | 平均响应时间 | 最长响应时间 | 状态 |
|----------|--------------|--------------|------|
| 用户注册 | 650ms | 890ms | ✓ |
| 用户登录 | 320ms | 450 | ✓ |
| 商品浏览 | 280ms | 380ms | ✓ |
| 商品搜索 | 450ms | 620ms | ✓ |
| 加入购物车 | 210ms | 310ms | ✓ |
| 创建订单 | 890ms | 1200ms | ⚠ |
| 支付处理 | 1500ms | 2100ms | ⚠ |

## 建议

1. 优化 Mobile Safari 的支付表单布局
2. 考虑优化创建订单和支付处理性能
3. 添加更多边界情况测试
```

## 关键要点

1. **完整的用户流程测试** - 覆盖从注册到支付的完整流程
2. **跨浏览器兼容性** - 在多个浏览器上测试
3. **响应式设计验证** - 在移动设备上测试
4. **性能监控** - 测量关键操作的响应时间
5. **详细的报告** - 提供截图、视频和修复建议

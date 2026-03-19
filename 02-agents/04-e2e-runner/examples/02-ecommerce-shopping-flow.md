# 场景 2: 电商购物流程

**类型：** 复杂业务流程 E2E 测试
**工具：** Playwright
**优先级：** HIGH (核心业务流程)

## 测试目标

验证完整的电商购物流程：
1. 浏览商品列表
2. 搜索商品
3. 添加商品到购物车
4. 查看购物车
5. 更新数量
6. 结账
7. 支付模拟

## 测试代码

```typescript
// file: e2e/ecommerce/shopping-flow.spec.ts

import { test, expect } from '@playwright/test';

test.describe('电商购物流程', () => {
  test('完整购物流程 - 浏览到结账', async ({ page }) => {
    // 1. 浏览商品列表
    await page.goto('/products');
    await expect(page.locator('.product-card')).toHaveCount(12);

    // 2. 搜索特定商品
    await page.fill('[name="search"]', 'laptop');
    await page.click('[data-testid="search-button"]');
    await expect(page.locator('.product-card')).toHaveCount(3);

    // 3. 添加第一个商品到购物车
    await page.click('.product-card:first-child [data-testid="add-to-cart"]');
    await expect(page.locator('[data-testid="cart-count"]')).toHaveText('1');

    // 4. 查看购物车
    await page.click('[data-testid="cart-icon"]');
    await expect(page).toHaveURL(/\/cart/);

    // 5. 验证购物车商品
    await expect(page.locator('[data-testid="cart-item"]')).toHaveCount(1);
    const itemName = await page.locator('[data-testid="cart-item-name"]').textContent();
    expect(itemName).toContain('Laptop');

    // 6. 更新数量为 2
    await page.click('[data-testid="increase-quantity"]');
    await expect(page.locator('[data-testid="cart-total"]')).toContainText('$'); // 更新总价

    // 7. 结账
    await page.click('[data-testid="checkout-button"]');
    await expect(page).toHaveURL(/\/checkout/);

    // 8. 填写收货信息
    await page.fill('[name="fullName"]', 'John Doe');
    await page.fill('[name="address"]', '123 Main St');
    await page.fill('[name="city"]', 'NewYork');
    await page.fill('[name="zipCode"]', '10001');

    // 9. 选择支付方式
    await page.click('[value="credit-card"]');
    await page.fill('[name="cardNumber"]', '4242424242424242'); // 测试卡号
    await page.fill('[name="expiry"]', '12/25');
    await page.fill('[name="cvv"]', '123');

    // 10. 提交订单
    await page.click('[data-testid="place-order"]');
    await expect(page.locator('[data-testid="order-success"]')).toBeVisible();

    // 11. 验证订单确认信息
    const orderNumber = await page.locator('[data-testid="order-number"]').textContent();
    expect(orderNumber).toMatch(/ORD-\d+/);

    // 12. 查看订单详情
    await page.click('[data-testid="view-order"]');
    await expect(page.locator('[data-testid="order-total"]')).toBeVisible();
  });

  test('购物车空状态处理', async ({ page }) => {
    await page.goto('/cart');

    // 验证空购物车提示
    await expect(page.locator('[data-testid="empty-cart"]')).toContainText('购物车为空');
    await expect(page.locator('[data-testid="checkout-button"]')).toBeDisabled();
  });

  test('从商品详情页添加到购物车', async ({ page }) => {
    await page.goto('/products');
    await page.click('.product-card:first-child');

    // 验证商品详情页加载
    await expect(page.locator('[data-testid="product-detail"]')).toBeVisible();

    // 选择规格
    await page.click('[data-testid="size-M"]');

    // 添加到购物车
    await page.click('[data-testid="add-to-cart"]');

    // 验证成功提示
    await expect(page.locator('[data-testid="add-to-cart-success"]')).toBeVisible();
    await expect(page.locator('[data-testid="cart-count"]')).toHaveText('1');
  });
});
```

## 关键要点

### 测试策略

1. **用户路径完整验证** - 从浏览到下单的完整流程
2. **状态转换验证** - 购物车数量、总价更新
3. **表单填写** - 收货信息和支付表单
4. **订单确认** - 验证订单号和详情

### 最佳实践

- ✅ 模拟真实用户行为
- ✅ 验证每个状态变化
- ✅ 使用测试数据（测试卡号）
- ✅ 检查空状态和边界条件
- ✅ 验证成功后的重定向

### 数据驱动测试

```typescript
const testProducts = [
  { id: 1, name: 'Laptop', price: 999.99 },
  { id: 2, name: 'Mouse', price: 29.99 },
  { id: 3, name: 'Keyboard', price: 79.99 }
];

test.describe('多商品购物测试', () => {
  testProducts.forEach(product => {
    test(`添加 ${product.name} 到购物车`, async ({ page }) => {
      await page.goto('/products');
      await page.click(`[data-testid="product-${product.id}"]`);
      await page.click('[data-testid="add-to-cart"]');

      await expect(page.locator('[data-testid="cart-count"]')).toHaveText('1');
    });
  });
});
```

### 网络模拟

```typescript
test('网络错误处理', async ({ page }) => {
  // 模拟结账 API 失败
  await page.route('/api/checkout', route => {
    route.fulfill({
      status: 500,
      body: JSON.stringify({ error: 'Payment gateway error' })
    });
  });

  await page.goto('/cart');
  await page.click('[data-testid="checkout-button"]');
  await page.click('[data-testid="place-order"]');

  // 验证错误消息显示
  await expect(page.locator('[data-testid="error-message"]')).toContainText('结账失败');
});
```

### 性能验证

```typescript
test('购物流程性能', async ({ page }) => {
  // 启用性能追踪
  await page.goto('/products');
  const metrics = await page.evaluate(() => performance.getEntries());

  // 检查关键指标
  const loadTime = metrics.filter(entry => entry.entryType === 'navigation')
    .reduce((sum, entry) => sum + entry.duration, 0);

  // ✅ 页面加载应在 2 秒内完成
  expect(loadTime).toBeLessThan(2000);
});
```

---

**场景价值：** 电商应用的核心盈利流程，必须确保每个环节都正常工作，包括异常处理和网络错误。

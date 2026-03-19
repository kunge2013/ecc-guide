# 场景 3: 响应式设计测试

**类型：** 跨设备兼容性 E2E 测试
**工具：** Playwright
**优先级：** MEDIUM (用户体验)

## 测试目标

验证应用在不同设备和屏幕尺寸上的响应式设计：
1. 桌面端布局
2. 平板端布局
3. 移动端布局
4. 断点切换
5. 触摸交互

## 测试代码

```typescript
// file: e2e/responsive/responsive-design.spec.ts

import { test, expect, devices } from '@playwright/test';

test.describe('响应式设计测试', () => {
  // 桌面端测试
  test.describe('桌面端布局', () => {
    test.use({ viewport: { width: 1920, height: 1080 } });

    test('完整布局 - 桌面端', async ({ page }) => {
      await page.goto('/');

      // ✅ 验证完整导航栏
      await expect(page.locator('[data-testid="main-nav"]')).toBeVisible();
      await expect(page.locator('[data-testid="nav-items"]')).toHaveCount(5);

      // ✅ 验证侧边栏显示
      await expect(page.locator('[data-testid="sidebar"]')).toBeVisible();

      // ✅ 验证网格布局
      const grid = page.locator('[data-testid="content-grid"]');
      await expect(grid).toBeVisible();
    });

    test('悬停交互 - 桌面端', async ({ page }) => {
      await page.goto('/');

      // ✅ 验证导航项悬停效果
      await page.hover('[data-testid="nav-item-1"]');
      await expect(page.locator('[data-testid="dropdown-menu"]')).toBeVisible();
    });
  });

  // 移动端测试
  test.describe('移动端布局', () => {
    test.use(devices['iPhone 12']);

    test('移动端适配', async ({ page }) => {
      await page.goto('/');

      // ✅ 汉堡菜单
      await expect(page.locator('[data-testid="hamburger-menu"]')).toBeVisible();
      await expect(page.locator('[data-testid="main-nav"]')).not.toBeVisible();

      // ✅ 点击打开菜单
      await page.click('[data-testid="hamburger-menu"]');
      await expect(page.locator('[data-testid="mobile-nav"]')).toBeVisible();

      // ✅ 单列内容
      const grid = page.locator('[data-testid="content-grid"]');
      const gridStyle = await grid.evaluate(el => {
        return window.getComputedStyle(el).gridTemplateColumns;
      });
      expect(gridStyle).toContain('repeat(1'); // 1 列
    });

    test('触摸交互 - 移动端', async ({ page }) => {
      await page.goto('/');

      // ✅ 点击打开移动端菜单
      await page.click('[data-testid="hamburger-menu"]');
      await expect(page.locator('[data-testid="mobile-nav"]')).toBeVisible();

      // ✅ 点击菜单项导航
      await page.click('[data-testid="nav-item-2"]');
      await expect(page.locator('[data-testid="mobile-nav"]')).not.toBeVisible();
      await expect(page).toHaveURL(/\/page-2/);
    });
  });

  // 断点测试
  test.describe('断点切换测试', () => {
    const breakpoints = [
      { width: 320, expectedColumns: 1 },
      { width: 768, expectedColumns: 2 },
      { width: 1024, expectedColumns: 3 },
      { width: 1920, expectedColumns: 4 }
    ];

    breakpoints.forEach(breakpoint => {
      test(`断点 ${breakpoint.width}px - ${breakpoint.expectedColumns} 列`, async ({ page }) => {
        await page.setViewportSize({ width: breakpoint.width, height: 800 });
        await page.goto('/');

        const grid = page.locator('[data-testid="content-grid"]');
        const gridStyle = await grid.evaluate(el => {
          return window.getComputedStyle(el).gridTemplateColumns;
        });

        expect(gridStyle).toContain(`repeat(${breakpoint.expectedColumns}`);
      });
    });
  });
});
```

## 关键要点

### 测试策略
- 多设备测试（桌面、平板、移动）
- 断点验证（所有主要断点）
- 交互方式（桌面悬停，移动端触摸）
- 布局变化（元素显隐）

### 最佳实践
- ✅ 测试所有主要断点
- ✅ 验证交互方式正确
- ✅ 检查元素显隐状态
- ✅ 测试触摸交互

---

**场景价值：** 确保响应式设计在不同设备上都能提供良好的用户体验。

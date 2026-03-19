# 使用场景 1: 用户认证功能

## 需求描述

实现完整的用户认证系统，包括：
- 用户注册（邮箱验证）
- 用户登录（JWT Token）
- 密码重置（邮件链接）
- 第三方登录集成（GitHub, Google）

## 技术要求

- 密码使用 bcrypt 哈希
- JWT 有效期 7 天
- 支持刷新 token
- 防止暴力破解
- 邮箱验证流程

## 完整实现流程

### 步骤 1: 需求分析和架构设计

```bash
# 使用 Planner Agent
Agent(planner, """
开发用户认证功能，设计完整的系统架构：

需求：
- 用户注册（邮箱验证）
- 用户登录（JWT Token）
- 密码重置（邮件链接）
- 第三方登录集成（GitHub, Google）

技术栈：Node.js + Express + MongoDB + JWT
安全要求：
- 密码 bcrypt 哈希
- JWT 有效期 7 天
- 支持刷新 token
- 防止暴力破解

请设计：
1. 数据模型
2. API 接口
3. 认证流程
4. 安全措施
""")
```

**Planner 输出：**

```markdown
# 用户认证系统架构设计

## 数据模型

### User Schema
```javascript
{
  _id: ObjectId,
  email: String (unique, required),
  password: String (bcrypt hash),
  username: String (unique),
  isVerified: Boolean (default: false),
  verificationToken: String,
  resetPasswordToken: String,
  resetPasswordExpires: Date,
  providers: Array [{
    name: String, // 'local', 'github', 'google'
    providerId: String,
    accessToken: String,
    refreshToken: String
  }],
  createdAt: Date,
  lastLoginAt: Date
}
```

## API 接口

### POST /api/auth/register
- 请求体: `{ email, password, username }`
- 响应: `{ user, token }`
- 发送验证邮件

### POST /api/auth/login
- 请求体: `{ email, password }`
- 响应: `{ user, token, refreshToken }`

### POST /api/auth/verify-email
- 请求体: `{ token }`
- 响应: `{ success }`

### POST /api/auth/reset-password
- 请求体: `{ email }`
- 响应: `{ success }`（发送重置邮件）

### POST /api/auth/new-password
- 请求体: `{ token, password }`
- 响应: `{ success }`

### POST /api/auth/refresh-token
- 请求体: `{ refreshToken }`
- 响应: `{ token, refreshToken }`

### GET /api/auth/github
- 重定向到 GitHub OAuth

### GET /api/auth/github/callback
- GitHub OAuth 回调

## 安全措施

1. **密码安全**
   - bcrypt 哈希（cost: 12）
   - 密码强度验证
   - 防止常见密码

2. **JWT 安全**
   - 使用 RS256 算法
   - 有效期 7 天
   - 刷新 token 有效期 30 天
   - Token 黑名单机制

3. **暴力破解防护**
   - 登录录失败限制（5 次/15 分钟）
   - IP 限制（10 次/分钟）
   - CAPTCHA 验证

4. **会话管理**
   - Redis 存储 refresh tokens
   - 自动清理过期 tokens
   - 支持多设备登录

## 实施计划

### 阶段 1: 核心认证（2 天）
- [ ] 实现 User 模型
- [ ] 实现注册功能
- [ ] 实现登录功能
- [ ] 实现 JWT 生成和验证

### 阶段 2: 邮件验证（1 天）
- [ ] 实现验证 token 生成
- [ ] 邮件发送服务
- [ ] 邮箱验证流程

### 阶段 3: 密码重置（1 天）
- [ ] 实现重置 token 生成
- [ ] 密码重置流程
- [ ] Token 过期验证

### 阶段 4: 第三方登录（2 天）
- [ ] GitHub OAuth 集成
- [ ] Google OAuth 集成
- [ ] Provider 账户绑定

### 阶段 5: 安全增强（1 天）
- [ ] 登录录失败限制
- [ ] IP 限制
- [ ] Token 黑名单

### 阶段 6: 测试和文档（1 天）
- [ ] 单元测试
- [ ] 集成测试
- [ ] E2E 测试
- [ ] API 文档
```

### 步骤 2: TDD 开发核心功能

```bash
# 使用 TDD Guide Agent
Agent(tdd-guide, """
按照 TDD 流程实现用户注册功能：

需求：
- 验证邮箱格式
- 验证密码强度（至少 8 个字符，包含大小写字母和数字）
- 验证用户名唯一性
- 密码使用 bcrypt 哈希
- 生成验证 token
- 发送验证邮件
- 返回用户信息（不包含密码）

使用 TDD：
1. 先写测试（RED）
2. 实现代码（GREEN）
3. 重构代码（REFACTOR）

确保测试覆盖率达到 80% 以上。
""")
```

**TDD 开发过程：**

```javascript
// file: test/auth/user-service.test.js

describe('UserService - 用户注册', () => {
  describe('validateEmail', () => {
    it('应该接受有效的邮箱', () => {
      const result = UserService.validateEmail('test@example.com');
      expect(result.valid).toBe(true);
    });

    it('应该拒绝无效的邮箱', () => {
      const result = UserService.validateEmail('invalid-email');
      expect(result.valid).toBe(false);
    });
  });

  describe('validatePassword', () => {
    it('应该接受强密码', () => {
      const result = UserService.validatePassword('SecurePass123');
      expect(result.valid).toBe(true);
    });

    it('应该拒绝弱密码', () => {
      const result = UserService.validatePassword('weak');
      expect(result.valid).toBe(false);
    });
  });

  describe('register', () => {
    it('应该成功注册新用户', async () => {
      const userData = {
        email: 'test@example.com',
        password: 'SecurePass123',
        username: 'testuser'
      };

      const result = await userService.register(userData);

      expect(result.user.email).toBe(userData.email);
      expect(result.user.username).toBe(userData.username);
      expect(result.user.password).toBeUndefined(); // 不返回密码
      expect(result.user.isVerified).toBe(false);
      expect(result.verificationToken).toBeDefined();
    });

    it('应该哈希密码', async () => {
      const userData = {
        email: 'test@example.com',
        password: 'SecurePass123',
        username: 'testuser'
      };

      await userService.register(userData);
      const user = await User.findOne({ email: userData.email });

      // 验证密码已哈希
      const isMatch = await bcrypt.compare(userData.password, user.password);
      expect(isMatch).toBe(true);
    });

    it('应该拒绝重复邮箱', async () => {
      const userData = {
        email: 'test@example.com',
        password: 'SecurePass123',
        username: 'testuser'
      };

      await userService.register(userData);

      await expect(userService.register(userData))
        .rejects.toThrow('邮箱已存在');
    });
  });
});
```

```javascript
// file: src/auth/user-service.js (已实现）

class UserService {
  static validateEmail(email) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(email)) {
      return { valid: false, message: '邮箱格式无效' };
    }
    return { valid: true };
  }

  static validatePassword(password) {
    if (!password || password.length < 8) {
      return { valid: false, message: '密码至少 8 个字符' };
    }
    if (!/[A-Z]/.test(password)) {
      return { valid: false, message: '密码必须包含大写字母' };
    }
    if (!/[a-z]/.test(password)) {
      return { valid: false, message: '密码必须包含小写字母' };
    }
    if (!/[0-9]/.test(password)) {
      return { valid: false, message: '密码必须包含数字' };
    }
    return { valid: true };
  }

  async register({ email, password, username }) {
    // 验证邮箱
    const emailValidation = UserService.validateEmail(email);
    if (!emailValidation.valid) {
      throw new Error(emailValidation.message);
    }

    // 验证密码
    const passwordValidation = UserService.validatePassword(password);
    if (!passwordValidation.valid) {
      throw new Error(passwordValidation.message);
    }

    // 检查邮箱唯一性
    const existingUser = await User.findOne({ email });
    if (existingUser) {
      throw new Error('邮箱已存在');
    }

    // 哈希密码
    const hashedPassword = await bcrypt.hash(password, 12);

    // 生成验证 token
    const verificationToken = crypto.randomBytes(32).toString('hex');

    // 创建用户
    const user = await User.create({
      email,
      password: hashedPassword,
      username,
      verificationToken,
      isVerified: false
    });

    // 发送验证邮件
    await this.sendVerificationEmail(user.email, verificationToken);

    return {
      user: user.toObject(),
      verificationToken
    };
  }

  async sendVerificationEmail(email, token) {
    // 实现邮件发送
    const verificationUrl = `${process.env.APP_URL}/verify-email?token=${token}`;

    await emailService.send({
      to: email,
      subject: '验证您的邮箱',
      html: `
        <p>请点击以下链接验证您的邮箱：</p>
        <a href="${verificationUrl}">验证邮箱</a>
      `
    });
  }
}
```

### 步骤 3: 安全审查

```bash
# 使用 Code Reviewer Agent
Agent(code-reviewer, """
审查认证模块代码，重点检查：

1. 安全问题
   - SQL 注入风险
   - XSS 漏洞
   - 密码处理安全
   - JWT 安全

2. 最佳实践
   - 错误处理
   - 输入验证
   - 密码强度要求
   - Token 管理

3. 性能考虑
   - 密码哈希性能
   - 数据库查询优化
   - 缓存策略

4. 代码质量
   - 代码可读性
   - 函数复杂度
   - 测试覆盖
""")
```

**安全审查发现：**

```markdown
## 发现的问题

### 🔴 CRITICAL

#### 1. 密码可能被记录
**位置：** `src/auth/user-service.js:45`
**问题：** 错误日志可能包含密码
**修复：**
```javascript
// ❌ 错误：可能记录密码
console.log('注册失败', { email, password });

// ✅ 修复：不记录密码
console.log('注册失败', { email });
```

### 🟠 HIGH

#### 2. 验证 Token 没有过期时间
**位置：** `src/auth/user-service.js:85`
**问题：** 验证 token 没有设置过期时间
**修复：**
```javascript
// ✅ 添加过期时间
user.verificationToken = crypto.randomBytes(32).toString('hex');
user.verificationTokenExpires = Date.now() + 24 * 60 * 60 * 1000; // 24 小时
```

### 🟡 MEDIUM

#### 3. 缺少登录录失败限制
**位置：** `src/auth/auth-controller.js`
**问题：** 没有限制登录录失败次数
**修复：**
```javascript
// ✅ 添加登录录失败限制
const MAX_ATTEMPTS = 5;
const LOCKOUT_TIME = 15 * 60 * 1000; // 15 分钟

async login(req, res) {
  const { email, password } = req.body;

  // 检查是否被锁定
  const attempts = await this.getLoginAttempts(email);
  if (attempts.count >= MAX_ATTEMPTS &&
      Date.now() - attempts.lastAttempt < LOCKOUT_TIME) {
    return res.status(429).json({
      error: '账户已锁定，请 15 分钟后再试'
    });
  }

  // ... 登录逻辑 ...
}
```

## 修复后的代码

所有 CRITICAL 和 HIGH 问题已修复。MEDIUM 问题已记录并在下个迭代处理。
```

### 步骤 4: E2E 测试

```bash
# 使用 E2E Runner Agent
Agent(e2eater, """
测试完整的认证流程：

1. 注册流程
   - 填写注册表单
   - 验证邮箱格式
   - 验证密码强度
   - 提交注册
   - 接收验证邮件
   - 验证邮箱
   - 自动登录

2. 登录流程
   - 输入有效凭据
   - 成功登录
   - 重定向到仪表盘
   - JWT Token 有效
   - 可以访问受保护页面

3. 密码重置流程
   - 请求密码重置
   - 接收重置邮件
   - 设置新密码
   - 使用新密码登录

4. 错误场景
   - 无效邮箱
   - 弱密码
   - 重复注册
   - 错误凭据
   - 过期的验证 token

使用 Playwright 进行测试，确保流程完整性和用户友好性。
""")
```

**E2E 测试实现：**

```typescript
// file: e2e/auth/complete-auth-flow.spec.ts

import { test, expect } from '@playwright/test';

test.describe('完整认证流程', () => {
  test('注册流程 - 从注册到验证', async ({ page }) => {
    const timestamp = Date.now();
    const email = `test+${timestamp}@example.com`;

    // 1. 访问注册页面
    await page.goto('/register');
    await expect(page).toHaveTitle('注册 - 我的App');

    // 2. 填写注册表单
    await page.fill('[name="email"]', email);
    await page.fill('[name="password"]', 'SecurePass123');
    await page.fill('[name="confirmPassword"]', 'SecurePass123');

    // 3. 提交注册
    await page.click('[type="submit"]');
    await expect(page.locator('[data-testid="success-message"]'))
      .toContainText('注册成功，请验证您的邮箱');

    // 4. 模拟验证邮箱（实际场景会从邮件获取 token）
    // 这里假设有测试接口获取验证 token
    const response = await page.request.get(`/api/test/get-verification-token?email=${email}`);
    const { verificationToken } = await response.json();

    // 5. 验证邮箱
    await page.goto(`/verify-email?token=${verificationToken}`);
    await expect(page.locator('[data-testid="success-message"]'))
      .toContainText('邮箱验证成功');

    // 6. 自动登录并重定向到仪表盘
    await expect(page).toHaveURL(/\/dashboard/);
  });

  test('登录流程', async ({ page }) => {
    // 1. 访问登录页面
    await page.goto('/login');

    // 2. 填写凭据
    await page.fill('[name="email"]', 'test@example.com');
    await page.fill('[name="password"]', 'SecurePass123');

    // 3. 提交登录
    await page.click('[type="submit"]');

    // 4. 验证重定向到仪表盘
    await expect(page).toHaveURL(/\/dashboard/);
    await expect(page.locator('h1')).toContainText('仪表盘');

    // 5. 验证 JWT Token
    const cookies = await page.context().cookies();
    const token = cookies.find(c => c.name === 'token');
    expect(token).toBeDefined();

    // 6. 访问受保护页面
    await page.goto('/profile');
    await expect(page.locator('[data-testid="user-profile"]')).toBeVisible();
  });

  test('密码重置流程', async ({ page }) => {
    // 1. 请求密码重置
    await page.goto('/forgot-password');
    await page.fill('[name="email"]', 'test@example.com');
    await page.click('[type="submit"]');

    await expect(page.locator('[data-testid="success-message"]'))
      .toContainText('重置链接已发送到您的邮箱');

    // 2. 获取重置 token
    const response = await page.request.get('/api/test/get-reset-token?email=test@example.com');
    const { resetToken } = await response.json();

    // 3. 访问重置页面
    await page.goto(`/reset-password?token=${resetToken}`);

    // 4. 设置新密码
    await page.fill('[name="newPassword"]', 'NewSecurePass123');
    await page.fill('[name="confirmPassword"]', 'NewSecurePass123');
    await page.click('[type="submit"]');

    // 5. 验证成功
    await expect(page.locator('[data-testid="success-message"]'))
      .toContainText('密码重置成功');

    // 6. 使用新密码登录
    await page.goto('/login');
    await page.fill('[name="email"]', 'test@example.com');
    await page.fill('[name="password"]', 'NewSecurePass123');
    await page.click('[type="submit"]');

    await expect(page).toHaveURL(/\/dashboard/);
  });

  test('错误场景 - 无效凭据', async ({ page }) => {
    await page.goto('/login');
    await page.fill('[name="email"]', 'invalid@example.com');
    await page.fill('[name="password"]', 'WrongPassword');
    await page.click('[type="submit"]');

    await expect(page.locator('[data-testid="error-message"]'))
      .toContainText('邮箱或密码错误');
  });

  test('错误场景 - 过期的验证 token', async ({ page }) => {
    await page.goto('/verify-email?token=expired-token');

    await expect(page.locator('[data-testid="error-message"]'))
      .toContainText('验证链接已过期');
  });
});
```

## 关键学习点

### 1. 架构设计优先

使用 Planner Agent 在编码前设计完整的架构：
- 数据模型设计
- API 接口设计
- 安全措施规划
- 实施计划制定

### 2. TDD 保证质量

使用 TDD Guide Agent 确保代码质量：
- 测试驱动开发
- 高测试覆盖率
- 安全验证
- 边界条件测试

### 3. 安全审查

使用 Code Reviewer Agent 发现安全问题：
- 密码处理安全
- JWT 安全配置
- 输入验证
- 错误处理

### 4. A 端到端验证

使用 E2E Runner Agent 验证完整流程：
- 用户流程完整性
- 错误场景处理
- 用户体验验证
- 集成测试

## 成功指标

### 功能完整性

✅ 所有认证功能实现
✅ 第三方登录集成
✅ 密码重置流程
✅ 邮箱验证流程

### 安全性

✅ 密码正确哈希
✅ JWT 安全配置
✅ 暴力破解防护
✅ 无安全漏洞

### 测试覆盖

✅ 单元测试覆盖率 > 80%
✅ 集成测试通过
✅ E2E 测试通过
✅ 性能测试通过

### 文档完整性

✅ API 文档完整
✅ 用户文档更新
✅ 安全文档更新
✅ 开发者指南更新

---

**提示：** 这是一个完整的新功能开发场景，展示了如何使用多个 Agent 协作完成从需求到实现的全流程。

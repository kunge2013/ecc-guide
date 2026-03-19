# 场景 3: 代码质量分析

**类型：** 代码质量审查
**工具：** SonarQube, ESLint, Pylint
**优先级：** MEDIUM (可维护性)

## 场景目标

分析代码质量并识别改进机会：
1. 代码重复检测
2. 复杂度分析
3. 代码覆盖率
4. 技术债评估

## 代码重复检测

### 问题代码

```javascript
// file: src/user/user-validator.js

class UserValidator {
  validateUsername(username) {
    if (!username || username.trim().length === 0) {
      return { valid: false, message: '用户名不能为空' };
    }
    if (username.length < 3) {
      return { valid: false, message: '用户名至少3个字符' };
    }
    if (username.length > 30) {
      return { valid: false, message: '用户名最多30个字符' };
    }
    if (!/^[a-zA-Z0-9_]+$/.test(username)) {
      return { valid: false, message: '用户名只能包含字母、数字和下划线' };
    }
    return { valid: true };
  }

  validateEmail(email) {
    // ❌ 重复的验证逻辑
    if (!email || email.trim().length === 0) {
      return { valid: false, message: '邮箱不能为空' };
    }
    if (email.length < 5) {
      return { valid: false, message: '邮箱至少5个字符' };
    }
    if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) {
      return { valid: false, message: '邮箱格式无效' };
    }
    return { valid: true };
  }

  validatePassword(password) {
    // ❌ 重复的验证逻辑
    if (!password || password.trim().length === 0) {
      return { valid: false, message: '密码不能为空' };
    }
    if (password.length < 8) {
      return { valid: false, message: '密码至少8个字符' };
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
}
```

### SonarQube 重复检测结果

```json
{
  "duplications": [
    {
      "blocks": [
        {
          "fromLine": 5,
          "toLine": 12,
          "file": "src/user/user-validator.js"
        },
        {
          "fromLine": 18,
          "toLine": 25,
          "file": "src/user/user-validator.js"
        },
        {
          "fromLine": 32,
          "toLine": 40,
          "file": "src/user/user-validator.js"
        }
      ],
      "lines": 30,
      "tokens": 180,
      "percentage": 25.6
    }
  ]
}
```

### 修复方案

```javascript
// file: src/user/user-validator.js (已优化)

class UserValidator {
  // ✅ 提取通用验证逻辑
  validateNotEmpty(value, fieldName) {
    if (!value || value.trim().length === 0) {
      return { valid: false, message: `${fieldName}不能为空` };
    }
    return { valid: true };
  }

  validateLength(value, min, max, fieldName) {
    if (min && value.length < min) {
      return { valid: false, message: `${fieldName}至少${min}个字符` };
    }
    if (max && value.length > max) {
      return { valid: false, message: `${fieldName}最多${max}个字符` };
    }
    return { valid: true };
  }

  validatePattern(value, pattern, errorMessage) {
    if (!pattern.test(value)) {
      return { valid: false, message: errorMessage };
    }
    return { valid: true };
  }

  // ✅ 使用通用方法
  validateUsername(username) {
    const checks = [
      this.validateNotEmpty(username, '用户名'),
      this.validateLength(username, 3, 30, '用户名'),
      this.validatePattern(username, /^[a-zA-Z0-9_]+$/, '用户名只能包含字母、数字和下划线')
    ];

    return checks.find(result => !result.valid) || { valid: true };
  }

  validateEmail(email) {
    const checks = [
      this.validateNotEmpty(email, '邮箱'),
      this.validateLength(email, 5, null, '邮箱'),
      this.validatePattern(email, /^[^\s@]+@[^\s@]+\.[^\s@]+$/, '邮箱格式无效')
    ];

    return checks.find(result => !result.valid) || { valid: true };
  }

  validatePassword(password) {
    const checks = [
      this.validateNotEmpty(password, '密码'),
      this.validateLength(password, 8, null, '密码'),
      this.validatePattern(password, /[A-Z]/, '密码必须包含大写字母'),
      this.validatePattern(password, /[a-z]/, '密码必须包含小写字母'),
      this.validatePattern(password, /[0-9]/, '密码必须包含数字')
    ];

    return checks.find(result => !result.valid) || { valid: true };
  }
}
```

## 复杂度分析

### 高复杂度代码

```javascript
// file: src/order/order-processor.js

class OrderProcessor {
  // ❌ 圈复杂度：15（太复杂）
  processOrder(order) {
    let result = { success: false, message: '' };

    // 验证订单
    if (!order) {
      result.message = '订单不存在';
      return result;
    }

    if (!order.items || order.items.length === 0) {
      result.message = '订单必须包含商品';
      return result;
    }

    if (!order.customer) {
      result.message = '订单必须包含客户信息';
      return result;
    }

    if (order.total <= 0) {
      result.message = '订单金额必须大于零';
      return result;
    }

    // 检查库存
    for (const item of order.items) {
      if (!this.checkStock(item.productId, item.quantity)) {
        result.message = `商品 ${item.productId} 库存不足`;
        return result;
      }
    }

    // 应用折扣
    if (order.couponCode) {
      const discount = this.getDiscount(order.couponCode);
      if (discount && discount.isValid()) {
        order.total = order.total * (1 - discount.percentage);
      }
    }

    // 计算运费
    if (order.shippingMethod === 'express') {
      order.shippingCost = order.total > 100 ? 0 : 15;
    } else if (order.shippingMethod === 'standard') {
      order.shippingCost = order.total > 50 ? 5 : 10;
    } else {
      order.shippingCost = 8;
    }

    // 检查支付
    if (order.paymentMethod === 'credit') {
      if (!this.validateCreditCard(order.paymentInfo)) {
        result.message = '信用卡信息无效';
        return result;
      }
    } else if (order.paymentMethod === 'paypal') {
      if (!this.validatePayPal(order.paymentInfo)) {
        result.message = 'PayPal 信息无效';
        return result;
      }
    }

    // 保存订单
    try {
      const savedOrder = this.saveOrder(order);
      result.success = true;
      result.order = savedOrder;
    } catch (error) {
      result.message = '保存订单失败';
      result.error = error;
    }

    return result;
  }
}
```

### SonarQube 复杂度报告

```json
{
  "components": [
    {
      "name": "OrderProcessor.processOrder",
      "complexity": 15,
      "violations": [
        {
          "rule": "cyclomatic-complexity",
          "severity": "MAJOR",
          "message": "圈复杂度为15（允许：10）"
        }
      ]
    }
  ]
}
```

### 修复方案

```javascript
// file: src/order/order-processor.js (已优化)

class OrderProcessor {
  // ✅ 拆分为多个小方法
  processOrder(order) {
    const validation = this.validateOrder(order);
    if (!validation.valid) {
      return { success: false, message: validation.message };
    }

    try {
      const processed = this.applyDiscounts(order);
      this.calculateShipping(processed);
      const saved = this.saveOrder(processed);
      return { success: true, order: saved };
    } catch (error) {
      return { success: false, message: '保存订单失败', error };
    }
  }

  // ✅ 验证逻辑
  validateOrder(order) {
    if (!order) return { valid: false, message: '订单不存在' };
    if (!order.items?.length) return { valid: false, message: '订单必须包含商品' };
    if (!order.customer) return { valid: false, message: '订单必须包含客户信息' };
    if (order.total <= 0) return { valid: false, message: '订单金额必须大于零' };

    const stockCheck = this.checkStock(order.items);
    if (!stockCheck.valid) {
      return stockCheck;
    }

    const paymentCheck = this.validatePayment(order);
    if (!paymentCheck.valid) {
      return paymentCheck;
    }

    return { valid: true };
  }

  // ✅ 库存检查
  checkStock(items) {
    for (const item of items) {
      if (!this.checkItemStock(item.productId, item.quantity)) {
        return { valid: false, message: `商品 ${item.productId} 库存不足` };
      }
    }
    return { valid: true };
  }

  // ✅ 折扣应用
  applyDiscounts(order) {
    const processed = { ...order };

    if (order.couponCode) {
      const discount = this.getDiscount(order.couponCode);
      if (discount?.isValid()) {
        processed.total = order.total * (1 - discount.percentage);
      }
    }

    return processed;
  }

  // ✅ 运费计算
  calculateShipping(order) {
    const shippingCalculator = {
      express: (total) => total > 100 ? 0 : 15,
      standard: (total) => total > 50 ? 5 : 10,
      default: () => 8
    };

    const calculator = shippingCalculator[order.shippingMethod] || shippingCalculator.default;
    order.shippingCost = calculator(order.total);
  }

  // ✅ 支付验证
  validatePayment(order) {
    const validators = {
      credit: (info) => this.validateCreditCard(info),
      paypal: (info) => this.validatePayPal(info)
    };

    const validator = validators[order.paymentMethod];
    if (validator && !validator(order.paymentInfo)) {
      return { valid: false, message: `${order.paymentMethod} 信息无效` };
    }

    return { valid: true };
  }
}
```

## 技术债评估

### SonarQube 报告

```json
{
  "project": {
    "name": "my-project",
    "metrics": {
      "complexity": 450,
      "duplicated_lines_density": 12.5,
      "coverage": 65.4,
      "sqale_rating": "C",
      "reliability_rating": "B",
      "security_rating": "A",
      "maintainability_rating": "C",
      "code_smells": 45,
      "bugs": 8,
      "vulnerabilities": 2,
      "security_hotspots": 5
    }
  }
}
```

### 技术债改善计划

```markdown
# 技术债改善计划

## 当前状态
- 质量等级: C
- 代码重复率: 12.5%
- 测试覆盖率: 65.4%
- 代码异味: 45 个
- Bug: 8 个
- 漏洞: 2 个

## 改善目标（3 个月）
- [ ] 质量等级提升到 B
- [ ] 代码重复率降低到 5%
- [ ] 测试覆盖率达到 80%
- [ ] 修复所有 CRITICAL 问题
- [ ] 减少代码异味到 20 个以下

## 行动计划
1. 第一个月
   - 修复所有漏洞和安全热点
   - 提升测试覆盖到 75%
   - 减少代码重复到 10%

2. 第二个月
   - 修复所有 Bug
   - 提升测试覆盖到 80%
   - 减少代码异味到 30 个

3. 第三个月
   - 减少代码重复到 5%
   - 减少代码异味到 20 个
   - 建立持续质量监控
```

## CI/CD 集成

```yaml
# .github/workflows/quality-gate.yml

name: Quality Gate

on:
  pull_request:
    branches: [main]

jobs:
  sonarqube:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Setup Node.js
        uses: actions/setup-node@v3

      - name: Install dependencies
        run: npm install

      - name: Run tests with coverage
        run: npm test -- --coverage

      - name: SonarQube Scan
        uses: SonarSource/sonarqube-scan-action@master
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
        with:
          args: >
            -Dsonar.projectKey=my-project
            -Dsonar.sources=src/
            -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info

  eslint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3

      - name: Install dependencies
        run: npm install

      - name: Run ESLint
        run: npm run lint

      - name: Check complexity
        run: |
          # 检查圈复杂度
          npm run complexity-check
```

## 关键要点

### 质量度量

| 指标 | 好的阈值 | 需要改进 | 差 |
|------|----------|----------|-----|
| 圈复杂度 | < 10 | 10-20 | > 20 |
| 代码重复率 | < 3% | 3-10% | > 10% |
| 测试覆盖率 | > 80% | 50-80% | < 50% |
| 代码行数/函数 | < 50 | 50-100 | > 100 |
| 文件行数 | < 300 | 300-500 | > 500 |

### 改善策略

1. **减少复杂度**
   - 拆分大函数
   - 提取重复逻辑
   - 使用策略模式

2. **消除重复**
   - 提取公共方法
   - 创建工具函数
   - 使用继承或组合

3. **提高覆盖率**
   - 添加单元测试
   - 测试边界条件
   - 测试错误路径

---

**场景价值：** 代码质量直接影响可维护性和开发效率。定期质量分析可以预防技术债积累。

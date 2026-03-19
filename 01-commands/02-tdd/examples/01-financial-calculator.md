# 场景 1: 金融计算 - 利息计算器

这个场景展示了 TDD 如何处理复杂业务逻辑和边界条件。

## 需求

实现一个简单的银行存款利息计算器：
- 存款金额
- 年利率（百分比）
- 存款期限（月）
- 复利计算（可选）
- 计算最终本息

## 测试驱动开发过程

### Step 1: 写测试（RED）- simple_interest.test.js

```javascript
describe('利息计算器', function() {
  describe('简单利息', function() {
    it('应该计算正确的简单利息', function() {
      // 本金 10000，年利率 5%，存 12 个月
      // 简单利息 = 本金 * 利率 * 时间
      // 利息 = 10000 * 0.05 * 1 = 500
      // 本息合计 = 10500
      const result = calculateInterest(10000, 5, 12, false);
      expect(result).to.equal(10500);
    });

    it('应该处理零利率', function() {
      const result = calculateInterest(10000, 0, 12, false);
      expect(result).to.equal(10000);
    });

    it('应该处理零本金', function() {
      const result = calculateInterest(0, 5, 12, false);
      expect(result).to.equal(0);
    });

    it('应该处理部分月份', function() {
      // 本金 10000，年利率 12%，存 6 个月
      // 简单利息 = 10000 * 0.12 * 0.5 = 600
      const result = calculateInterest(10000, 12, 6, false);
      expect(result).to.equal(10600);
    });
  });

  describe('复利计算', function() {
    it('应该计算复利（按月复利）', function() {
      // 本金 10000，年利率 12%，存 12 个月，按月复利
      // 月利率 = 1% = 0.01
      // 本息 = 10000 * (1.01)^12 ≈ 11268.25
      const result = calculateInterest(10000, 12, 12, true);
      // 允许小数误差
      expect(Math.abs(result - 11268.25)).to.be.lessThan(0.01);
'});

    it('复利应该比简单利息高', function() {
      const simple = calculateInterest(10000, 10, 24, false);
      const compound = calculateInterest(10000, 10, 24, true);
      expect(compound).to.be.above(simple);
    });
  });

  describe('边界条件', function() {
    it('应该拒绝负本金', function() {
      expect(() => calculateInterest(-1000, 5, 12, false))
        .to.throw('本金不能为负数');
    });

    it('应该拒绝负利率', function() {
      expect(() => calculateInterest(1000, -5, 12, false))
        .to.throw('利率不能为负数');
    });

    it('应该拒绝负期限', function() {
      expect(() => calculateInterest(1000, 5, -12, false))
        .to.throw('存款期限不能为负数');
    });

    it('应该处理小数金额', function() {
      const result = calculateInterest(9999.99, 5, 12, false);
      expect(result).to.equal(10499.9895);
    });
  });
});
```

### Step 2: 实现代码（GREEN）- interest_calculator.js

```javascript
/**
 * 计算存款利息
 * @param {number} principal - 本金
 * @param {number} annualRate - 年利率（百分比）
 * @param {number} months - 存款期限（月）
 * @param {boolean} isCompound - 是否复利
 * @returns {number} 最终本息
 */
function calculateInterest(principal, annualRate, months, isCompound = false) {
  // 验证输入
  if (principal < 0) {
    throw new Error('本金不能为负数');
  }
  if (annualRate < 0) {
    throw new Error('利率不能为负数');
  }
  if (months < 0) {
    throw new Error('存款期限不能为负数');
  }

  // 转换年利率为小数
  const rate = annualRate / 100;
  const years = months / 12;

  if (isCompound) {
    // 复利计算：按月复利
    const monthlyRate = rate / 12;
    const amount = principal * Math.pow(1 + monthlyRate, months);
    // 保留两位小数
    return Math.round(amount * 100) / 100;
  } else {
    // 简单利息计算
    const interest = principal * rate * years;
    const amount = principal + interest;
    return Math.round(amount * 100) / 100;
  }
}
```

### Step 3: 重构（REFACTOR）

提取验证逻辑：

```javascript
function validateInterestInputs(principal, annualRate, months) {
  if (principal < 0) {
    throw new Error('本金不能为负数');
  }
  if (annualRate < 0) {
    throw new Error('利率不能'为负数');
  }
  if (months < 0) {
    throw new Error('存款期限不能为负数');
  }
}

function calculateInterest(principal, annualRate, months, isCompound = false) {
  validateInterestInputs(principal, annualRate, months);

  const rate = annualRate / 100;
  const years = months / 12;

  if (isCompound) {
    return calculateCompoundInterest(principal, rate, months);
  } else {
    return calculateSimpleInterest(principal, rate, years);
  }
}
```

## 关键学习点

1. **边界条件测试优先** - 在业务逻辑之前先测试零值、负值等边界条件
2. **业务规则形式化** - 将需求转化为数学公式，便于测试
3. **小数精度处理** - 金融计算必须注意浮点数精度问题
4. **输入验证** - 测试驱动开发确保所有验证逻辑都被覆盖

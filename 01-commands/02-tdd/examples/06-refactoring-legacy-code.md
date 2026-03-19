# 场景 6: 重构遗留代码 - 字符串工具类

这个场景展示了如何使用 TDD 安全地重构高风险的遗留代码。

## 场景背景

发现一个广泛使用的遗留工具类，存在以下问题：
- 没有测试覆盖
- 性能问题（大量字符串拷贝）
- 代码难以理解
- 潜在的边界条件 bug

## 测试驱动重构过程

### Step 1: 创建测试（RED）- Characterization Tests

首先编写特征测试，捕捉当前行为：

```javascript
// legacy_string_utils.test.js
describe('Legacy StringUtils - 特征测试', function() {
  describe('camelCase 转换', function() {
    it('当前行为：空字符串', function() {
      const result = toCamelCase('');
      // 记录当前行为，即使看起来不对
      expect(result).to.equal('');
    });

    it('当前行为：已经是 camelCase', function() {
      const result = toCamelCase('camelCase');
      expect(result).to.equal('camelCase');
    });

    it('当前行为：PascalCase 转 camelCase', function() {
      const result = toCamelCase('PascalCase');
      expect(result).to.equal('pascalCase');
    });

    it('当前行为：kebab-case 转 camelCase', function() {
      const result = toCamelCase('kebab-case-string');
      expect(result).to.equal('kebabCaseString');
    });

    it('当前行为：snake_case 转 camelCase', function() {
      const result = toCamelCase('snake_case_string');
      expect(result).to.equal('snakeCaseString');
    });

    it('当前行为：处理连续分隔符', function() {
      const result = toCamelCase('multiple---separators');
      expect(result)to.equal('multipleSeparators');
    });

    it('当前行为：处理空格', function() {
      const result = toCamelCase('space separated string');
      expect(result).to.equal('spaceSeparatedString');
    });

    it('当前行为：处理混合分隔符', function() {
      const result = toCamelCase('mixed-case_and_separators');
      expect(result).to.equal('mixedCaseAndSeparators');
    });

    it('当前行为：处理数字', function() {
      const result = toCamelCase('test-123-string');
      expect(result).to.equal('test123String');
    });

    it('当前行为：全大写', function() {
      const result = toCamelCase('UPPER-CASE-STRING');
      expect(result).to.equal('upperCaseString');
    });
  });

  describe('truncate 字符串截断', function() {
    it('当前行为：短于限制', function() {
      const result = truncate('short', 10);
      expect(result).to.equal('short');
    });

    it('当前行为：等于限制', function() {
      const result = truncate('exactly10', 10);
      expect(result).to.equal('exactly10');
    });

    it('当前行为：长于限制，默认省略号', function() {
      const result = truncate('very long string', 10);
      expect(result).to.equal('very long...');
    });

    it('当前行为：自定义省略号', function() {
      const result = truncate('very long string', 10, '>>');
      expect(result).to.equal('very long>>');
    });

    it('当前行为：空字符串', function() {
      const result = truncate('', 10);
      expect(result).to.equal('');
    });

    it('当前行为：负长度限制', function() {
      const result = truncate('test', -1);
      // 可能会抛出异常或返回空字符串
      expect(result).to.equal('');
    });
  });
});
```

### Step 2: 验证所有测试通过（GREEN）

运行测试，确保所有特征测试都通过：
```bash
npm test
# 所有测试应该通过
```

### Step 3: 重构代码（REFACTOR）

现在可以安全地重构，因为测试会捕捉任何行为变化：

#### 重构 1: toCamelCase 优化

```javascript
/**
 * 将各种命名格式转换为 camelCase
 * 支持格式：kebab-case, snake_case, PascalCase, space separated
 */
function toCamelCase(str) {
  if (!str) return '';

  // 统一处理：将所有分隔符（-、_、空格）替换为空格
  let normalized = str
    .replace(/[-_]/g, ' ')  // 替换 - 和 _ 为空格
    .replace(/\s+/g, ' ')   // 合并多个空格
    .trim();                 // 去除首尾空格

  if (!normalized) return '';

  // 分割单词
  const words = normalized.split(' ');

  // 第一个单词小写，后续单词首字母大写
  return words.map((word, index) => {
    if (index === 0) {
      return word.toLowerCase();
    }
    return word.charAt(0).toUpperCase() + word.slice(1).toLowerCase();
  }).join('');
}
```

#### 重构 2: truncate 优化

```javascript
/**
 * 截断字符串到指定长度，添加省略号
 * @param {string} str - 原字符串
 * @param {number} maxLength - 最大长度
 * @param {string} ellipsis - 省略号，默认 '...'
 * @returns {string} 截断后的字符串
 */
function truncate(str, maxLength, ellipsis = '...') {
  if (!str || maxLength <= 0) return '';
  if (str.length <= maxLength) return str;

  const availableLength = maxLength - ellipsis.length;
  if (availableLength <= 0) {
    // 即使只显示省略号也超出限制，截断省略号
    return ellipsis.slice(0, maxLength);
  }

  return str.slice(0, availableLength) + ellipsis;
}
```

### Step 4: 验证重构正确性

运行测试，确保行为没有改变：
```bash
npm test
# 所有特征测试应该仍然通过
```

### Step 5: 添加新功能和优化

现在可以安全地添加新功能：

```javascript
describe('StringUtils - 新功能', function() {
  describe('toCamelCase 增强版', function() {
    it('应该处理 null/undefined', function() {
      expect(toCamelCase(null)).to.equal('');
      expect(toCamelCase(undefined)).to.equal('');
    });

    it('应该保留单词内部的大写字母', function() {
      expect(toCamelCase('XML-Parser')).to.equal('xmlParser');
    });

    it('应该处理 Unicode 字符', function() {
      expect(toCamelCase('中文-测试-string')).to.equal('中文测试String');
    });
  });

  describe('truncate 增强版', function() {
    it('应该智能截断单词边界', function() {
      const result = truncateSmart('this is a long sentence', 15);
      expect(result).to.equal('this is...');
    });

    it('应该在单词中间截断时避免分割', function() {
      const result = truncateSmart('word', 4);
      expect(result).to.equal('w...');
    });
  });
});

/**
 * 智能截断，优先在单词边界截断
 */
function truncateSmart(str, maxLength, ellipsis = '...') {
  if (!str || maxLength <= 0) return '';
  if (str.length <= maxLength) return str;

  const availableLength = maxLength - ellipsis.length;
  if (availableLength <= 0) return ellipsis.slice(0, maxLength);

  // 查找最后一个空格位置
  let lastSpace = str.slice(0, availableLength + 1).lastIndexOf(' ');

  if (lastSpace > availableLength * 0.5) {
    // 如果空格位置合理（超过可用长度的一半），在空格处截断
    return str.slice(0, lastSpace) + ellipsis;
  }

  // 否则，在指定长度处截断
  return str.slice(0, availableLength) + ellipsis;
}
```

### Step 6: 性能基准测试

```javascript
describe('StringUtils - 性能测试', function() {
  it('toCamelCase 大量字符串处理', function() {
    const testString = 'a'.repeat(10000);
    const iterations = 1000;

    const start = performance.now();
    for (let i = 0; i < iterations; i++) {
      toCamelCase(testString);
    }
    const end = performance.now();

    const avgTime = (end - start) / iterations;
    console.log(`Average time per call: ${avgTime.toFixed(3)}ms`);

    // 确保性能在合理范围内
    expect(avgTime).to.be.lessThan(1); // 每次调用小于 1ms
  });

  it('compare old vs new implementation', function() {
    const testCases = Array(100).fill(0).map((_, i) =>
      `test-string-${i}-with-many-words-and-separators`
    );

    // 旧实现（如果需要保留对比）
    function toCamelCaseOld(str) {
      // 旧实现的代码
    }

    // 对比性能
    const startOld = performance.now();
    testCases.forEach(s => toCamelCaseOld(s));
    const endOld = performance.now();

    const startNew = performance.now();
    testCases.forEach(s => toCamelCase(s));
    const endNew = performance.now();

    const oldTime = endOld - startOld;
    const newTime = endNew - startNew;

    console.log(`Old: ${oldTime.toFixed(2)}ms, New: ${newTime.toFixed(2)}ms`);
    console.log(`Speedup: ${(oldTime / newTime).toFixed(2)}x`);

    // 新实现应该更快
    expect(newTime).to.be.lessThan(oldTime);
  });
});
```

## 关键学习点

1. **特征测试（Characterization Tests）** - 在重构前捕捉现有行为
2. **安全重构** - 测试作为安全网，确保行为不变
3. **增量优化** - 小步重构，每次运行测试验证
4. **性能基准** - 对比重构前后的性能
5. **逐步改进** - 先让代码能工作，再优化性能和可读性

## 重构检查清单

- [ ] 所有特征测试都通过
- [ ] 新功能有自己的测试
- [ ] 性能测试通过并显示改进
- [ ] 代码可读性提升
- [ ] 复杂度降低
- [ ] 边界条件处理正确
- [ ] 向后兼容（不破坏现有调用）

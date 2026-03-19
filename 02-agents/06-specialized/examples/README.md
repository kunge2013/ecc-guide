# Specialized Agents 示例集合

这个目录包含了 3 个专业化的 Agent 场景示例。

## 📚 示例列表

### 1. [安全漏洞扫描](./01-security-vulnerability-scan.md)
**类型：** 安全审查
**工具：** Snyk, OWASP ZAP
**优先级：** CRITICAL (安全关键)

**扫描内容：**
- 依赖漏洞检查
- 静态安全分析
- OWASP Top 10 检查
- 敏感数据泄露检测

**关键要点：**
- 定期扫描依赖
- 及时更新到安全版本
- 使用安全编码实践
- 实施防御深度

---

### 2. [性能分析和优化](./02-performance-analysis.md)
**类型：** 性能审查
**工具：** Lighthouse, Web Vitals
**优先级：** HIGH (用户体验)

**分析内容：**
- 加载性能（FCP, LCP）
- 运行时性能（TTI, FID）
- 资源优化（压缩, CDN）
- 渲染性能（CLS, 实例）

**关键要点：**
- 监控核心 Web Vitals
- 优化关键渲染路径
- 使用代码分割和懒加载
- 建立性能基线

---

### 3. [代码质量分析](./03-code-quality-analysis.md)
**类型：** 代码质量审查
**工具：** SonarQube, ESLint
**优先级：** MEDIUM (可维护性)

**分析内容：**
- 代码重复检测
- 复杂度分析
- 测试覆盖率
- 技术债评估

**关键要点：**
- 减少圈复杂度
- 消除代码重复
- 提高测试覆盖率
- 建立持续质量监控

---

## 🎯 使用建议

### 学习路径

1. **安全优先** → 示例 1
   - 安全是首要任务
   - 学习漏洞扫描
   - 了解 OWASP Top 10

2. **性能优化** → 示例 2
   - 用户体验关键
   - 学习性能监控
   - 掌握优化技巧

3. **质量提升** → 示例 3
   - 长期可维护性
   - 学习质量度量
   - 建立改善计划

### 专业领域

| 领域 | 推荐示例 | 工具集 |
|------|----------|--------|
| 安全审计 | 1 | Snyk, OWASP ZAP |
| 性能优化 | 2 | Lighthouse, Web Vitals |
| 代码质量 | 3 | SonarQube, ESLint |

### CI/CD 集成

```yaml
# .github/workflows/comprehensive-scan.yml

name: Comprehensive Scan

on:
  schedule:
    - cron: '0 2 * * *'  # 每天凌晨 2 点运行
  push:
    branches: [main]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Security Scan
        run: |
          npm install -g snyk
          snyk test --severity-threshold=high

  performance:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Performance Check
        run: npm run test:performance

  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Quality Gate
        run: npm run test:quality

  comprehensive-report:
    needs: [security, performance, quality]
    runs-on: ubuntu-latest
    steps:
      - name: Generate Report
        run: node scripts/generate-report.js
```

---

## 📊 优先级对比

| Agent | 优先级 | 响应时间 | 影响范围 |
|-------|--------|----------|----------|
| 安全扫描 | 🔴 CRITICAL | 立即 | 全系统 |
| 性能分析 | 🟠 HIGH | 24小时内 | 用户体验 |
| 代码质量 | 🟡 MEDIUM | 1周内 | 可维护性 |

---

## 🔗 相关资源

- [Specialized Agents](../README.md) - Agent 使用指南
- [安全指南](../../../rules/common/security.md) - 安全最佳实践
- [性能指南](../../../rules/common/performance.md) - 性能优化
- [编码规范](../../../rules/common/coding-style.md) - 代码质量

---

**提示：** 这些 Agent 专注于特定领域，提供深度的分析和建议。根据项目需求选择合适的 Agent。

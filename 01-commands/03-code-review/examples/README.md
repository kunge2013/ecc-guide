# Code Review 场景示例集合

这个目录包含了 4 个实际的 Code Review 场景示例，展示了如何通过代码审查发现和解决各种问题。

## 📚 示例列表

### 1. [SQL 注入漏洞修复](./01-sql-injection-fix.md)
**场景：** 安全漏洞发现和修复
**语言：** JavaScript

**涵盖要点：**
- SQL 注入漏洞识别
- 参数化查询最佳实践
- 安全测试用例
- 使用 ORM 库防注入

**审查重点：**
- 🔴 CRITICAL: SQL 注入漏洞
- ✅ 使用参数化查询
- ✅ 输入验证
- ✅ 安全编码实践

---

### 2. [异步错误处理改进](./02-async-error-handling.md)
**场景：** 异步代码错误处理完善
**语言：** JavaScript

**涵盖要点：**
- Promise 错误处理
- 异常传播策略
- 自定义错误类
- 资源清理和事务管理

**审查重点：**
- 🔴 CRITICAL: 未处理的 Promise rejection
- 🟠 HIGH: 错误信息丢失
- 🟡 MEDIUM: 资源泄漏风险
- ✅ finally 块使用
- ✅ 错误日志记录

---

### 3. [API 设计一致性改进](./03-api-design-consistency.md)
**场景：** REST API 设计规范化和一致性
**语言：** JavaScript/Node.js

**涵盖要点：**
- 统一 API 响应格式
- HTTP 状态码正确使用
- 输入验证和清理
- 控制器基类设计

**审查重点：**
- 🔴 CRITICAL: API 响应格式不一致
- 🔴 CRITICAL: HTTP 状态码使用不当
- 🟡 MEDIUM: 缺少输入验证
- 🟢 LOW: 内容协商功能
- ✅ RESTful 设计原则

---

### 4. [性能问题识别和优化](./04-performance-optimization.md)
**场景：** 代码性能问题发现和解决
**语言：** JavaScript/Node.js

**涵盖要点：**
- N+1 查询问题解决
- 算法复杂度优化
- 缓存策略应用
- 异步操作优化
- 内存泄漏预防

**审查重点：**
- 🔴 CRITICAL: N+1 查询性能问题
- 🟠 HIGH: 同步阻塞操作
- 🟡 MEDIUM: 重复计算
- 🟡 MEDIUM: 缺少缓存
- ✅ 批量查询
- ✅ 单次遍历优化

---

## 🎯 使用建议

### 学习路径

1. **安全审查优先** - 从示例 1 开始
   - 安全问题最紧急
   - 展示 CRITICAL 问题处理

2. **错误处理** - 学习示例 2
   - 异步代码是常见场景
   - 错误处理模式通用

3. **API 设计** - 学习示例 3
   - 后端开发者必备
   - API 一致性重要

4. **性能优化** - 学习示例 4
   - 性能问题影响用户体验
   - 需要性能测试验证

### 选择指南

| 你的需求 | 推荐示例 |
|---------|---------|
| 学习安全漏洞识别 | 1 |
| 处理异步代码 | 2 |
| 设计 REST API | 3 |
| 优化代码性能 | 4 |
| 学习错误处理模式 | 2 |
| 建立 API 规范 | 3 |
| 解决数据库性能问题 | 4 |

## 📊 问题严重程度说明

### 🔴 CRITICAL（必须修复）

阻止合并，必须立即修复：
- 安全漏洞（SQL 注入、XSS 等）
- 数据损坏风险
- 未处理的异常
- 资源泄漏

### 🟠 HIGH（强烈建议）

尽快修复：
- 性能问题（N+1 查询）
- 错误处理不当
- 安全最佳实践违背
- 兼容性问题

### 🟡 MEDIUM（建议修复）

在下一个版本修复：
- 代码可读性
- 命名规范
- 注释完善
- 小的性能改进

### 🟢 LOW（可选改进）

有时间时处理：
- 代码风格
- 注释改进
- 可选功能增强

## 🔍 Code Review 检查清单

### 安全检查（CRITICAL）

- [ ] 没有 SQL 注入风险
- [ ] 没有 XSS 漏洞
- [ ] 没有 CSRF 漏洞
- [ ] 认证授权正确
- [ ] 敏感数据不泄露
- [ ] 密码正确哈希
- [ ] 输入验证完整

### 功能检查（HIGH）

- [ ] 业务逻辑正确
- [ ] 边界条件处理
- [ ] 错误处理完善
- [ ] 异步操作正确
- [ ] 资源清理正确
- [ ] 事务管理正确

### 性能检查（HIGH）

- [ ] 没有 N+1 查询
- [ ] 没有不必要的重复计算
- [ ] 使用缓存（如适用）
- [ ] 没有同步阻塞操作
- [ ] 没有内存泄漏风险
- [ ] 数据库查询优化

### 质量检查（MEDIUM）

- [ ] 代码可读性好
- [ ] 命名清晰准确
- [ ] 函数职责单一
- [ ] 没有重复代码
- [ ] 复杂度合理
- [ ] 注释准确必要

### 测试检查（MEDIUM）

- [ ] 测试覆盖充分
- [ ] 测试质量高
- [ ] 测试命名清晰
- [ ] 边界条件已测试
- [ ] 错误情况已测试

## 💡 Code Review 最佳实践

### 作为审查者

1. **及时响应**
   ```bash
   # 不要让 PR 等待太久
   # 目标：24 小时内首次审查
   ```

2. **具体反馈**
   ```markdown
   ❌ "这段代码不太对"
   ✅ "在 file.js:42，应该使用参数化查询防止 SQL 注入"
   ```

3. **区分优先级**
   ```markdown
   🔴 CRITICAL: 必须修复
   🟠 HIGH: 强烈建议修复
   🟡 MEDIUM: 建议修复
   🟢 LOW: 可选优化
   ```

4. **建设性沟通**
   ```markdown
   ❌ "你的代码写得不好"
   ✅ "我建议使用函数式风格，这样更容易测试"
   ```

### 作为被审查者

1. **自检提交**
   ```bash
   # 提PR 前运行
   npm test
   npm run lint
   npm run type-check
   ```

2. **保持 PR 小**
   ```bash
   # PR 不应该太大
   # 目标：<400 行变更
   # 分解为多个小 PR
   ```

3. **清晰描述**
   ```markdown
   ## 变更类型
   - [ ] 新功能
   - [ ] Bug 修复

   ## 描述
   详细说明这次变更

   ## 测试
   - [ ] 单元测试
   - [ ] 集成测试
   - [ ] 手动测试
   ```

4. **回应反馈**
   ```markdown
   # 及时回应所有评论
   # 解释为什么采用某种方式
   # 修复 CRITICAL 和 HIGH 问题
   ```

## 🛠️ 工具支持

### 静态分析工具

```bash
# JavaScript/TypeScript
npm install -D eslint prettier
npm install -D @typescript-eslint/eslint-plugin

# 安全扫描
npm install -D eslint-plugin-security

# Python
pip install pylint black mypy

# Go
go fmt ./...
go vet ./...
golangci-lint run

# Java
./gradlew checkstyleMain
./gradlew spotbugsMain
```

### Code Review 工具

- **GitHub PR Review** - 内置审查功能
- **GitLab Merge Request** - 内置审查功能
- **SonarQube** - 自动代码质量分析
- **Phabricator** - 高级审查工具
- **Reviewable** - 专业审查工具

### 自动化工具

```yaml
# .github/workflows/code-review.yml
name: Code Review

on: [pull_request]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Lint
        run: npm run lint

      - name: Type Check
        run: npm run type-check

      - name: Test
        run: npm test

      - name: Security Scan
        run: npm audit

      - name: Code Coverage
        run: npm run test:coverage
```

## 📈 性能指标

### Code Review 效率

| 指标 | 目标值 | 说明 |
|------|--------|------|
| PR 大小 | <400 行 | 保持 PR 小而聚焦 |
| 首次审查时间 | <24 小时 | 及时响应 |
| 修复时间 | <2 天 | 快速修复反馈 |
| 通过率 | >70% | 一次通过的 PR 比例 |

### 代码质量

| 指标 | 目标值 | 说明 |
|------|--------|------|
| 测试覆盖率 | >80% | 代码测试覆盖 |
| 代码复杂度 | <10 | 圈复杂度限制 |
| 重复代码 | <3% | 代码重复率 |
| 安全扫描 | 0 漏洞 | 严重安全漏洞 |

## 🔗 相关资源

- [Code Review Guide](../code-review-guide.md) - Code Review 指南
- [Code Review Scenarios](./code-review-scenarios.md) - 场景分析
- [Security Guidelines](../../../rules/common/security.md) - 安全指南
- [Coding Standards](../../../rules/common/coding-style.md) - 编码规范

---

**提示：** 每个示例都包含完整的问题分析、修复方案和测试用例。建议按照示例中的步骤进行实践，并尝试在实际项目中应用这些审查技巧。

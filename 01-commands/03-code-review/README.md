# Code Review 完整指南

Code Review（代码审查）是确保代码质量、发现缺陷、分享知识和维护代码库健康的核心实践。

## 📚 文档结构

```
01-commands/03-code-review/
├── code-review-guide.md          # Code Review 指南
└── examples/                     # 实战示例集合
    ├── README.md                  # 示例索引
    ├── code-review-scenarios.md   # 场景分析
    ├── 01-sql-injection-fix.md         # SQL 注入修复
    ├── 02-async-error-handling.md      # 异步错误处理
    ├── 03-api-design-consistency.md    # API 设计一致性
    └── 04-performance-optimization.md  # 性能优化

02-agents/03-code-reviewer/
└── README.md                     # Code Reviewer Agent 指南
```

## 🚀 快速开始

### 1. 学习基础概念

阅读 [Code Review Guide](./code-review-guide.md) 了解：
- 什么是 Code Review
- 何时进行 Code Review
- 审查最佳实践
- 检查清单和模板

### 2. 学习实战场景

查看 [示例集合](./examples/)：
- 安全漏洞修复
- 错误处理改进
- API 设计优化
- 性能问题解决

### 3. 使用自动化工具

学习 [Code Reviewer Agent](../../02-agents/03-code-reviewer/)：
- 自动化代码审查
- 安全检查
- 性能分析
- CI/CD 集成

## 🎯 核心概念

### Code Review 的价值

1. **质量保证**
   - 在代码运行前发现缺陷
   - 确保代码符合项目标准
   - 防止技术债积累

2. **知识共享**
   - 团队成员互相学习
   - 最佳实践传播
   - 隐式知识显性化

3. **架构一致性**
   - 维持设计模式一致性
   - 防止架构腐化
   - 确保接口契约稳定

4. **安全防护**
   - 发现安全漏洞
   - 验证权限控制
   - 检查敏感数据处理

### 审查优先级

| 优先级 | 类型 | 处理时间 | 示例 |
|--------|------|----------|------|
| 🔴 CRITICAL | 阻止合并 | 立即 | 安全漏洞、数据损坏风险 |
| 🟠 HIGH | 强烈建议 | 24小时内 | 性能问题、架构不一致 |
| 🟡 MEDIUM | 建议 | 3天内 | 代码风格、命名改进 |
| 🟢 LOW | 可选 | 1周内 | 注释完善、微优化 |

## 📖 学习路径

### 路径 1: 安全优先（适合所有人）

1. 阅读 [SQL 注入修复](./examples/01-sql-injection-fix.md)
2. 学习安全检查清单
3. 在项目中实施安全 Code Review

### 路径 2: 错误处理（适合后端开发者）

1. 阅读 [异步错误处理改进](./examples/02-async-error-handling.md)
2. 学习错误处理模式
3. 完善项目的错误处理

### 路径 3: API 设计（适合 API 开发者）

1. 阅读 [API 设计一致性](./examples/03-api-design-consistency.md)
2. 建立 API 设计规范
3. 统一项目的 API 接口

### 路径 4: 性能优化（适合高级开发者）

1. 阅读 [性能问题识别和优化](./examples/04-performance-optimization.md)
2. 学习性能分析工具
3. 优化项目的性能瓶颈

### 路径 5: 自动化审查（适合 DevOps）

1. 阅读 [Code Reviewer Agent](../../02-agents/03-code-reviewer/)
2. 配置自动化审查工具
3. 集成到 CI/CD 流程

## 🔧 实践指南

### 在团队中建立 Code Review 流程

#### 步骤 1: 定义规范

创建团队 Code Review 规范：

```markdown
# Team Code Review 规范

## 审查范围
- 所有代码变更必须经过审查
- 配置文件变更需要技术负责人审查
- 安全相关代码需要安全专家审查

## 审查者
- 至少一人审查
- 关键变更需要两人审查
- 作者不能审查自己的代码

## 时间要求
- 首次审查响应：24 小时内
- 反馈修复：48 小时内
- PR 合并：所有 CRITICAL 问题修复
```

#### 步骤 2: 配置工具

```yaml
# .github/workflows/code-review.yml
name: Code Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  automated-review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Run Code Reviewer
        run: |
          Agent(code-reviewer, "自动代码审查")

      - name: Security Scan
        run: npm audit
```

#### 步骤 3: 培训团队

- 分享 Code Review 最佳实践
- 演示如何使用工具
- 建立审查示例库
- 定期回顾审查质量

### 个人 Code Review 实践

#### 作为审查者

```bash
# 检查清单
1. 安全性
   - [ ] 没有安全漏洞
   - [ ] 认证授权正确
   - [ ] 敏感数据保护

2. 功能性
   - [ ] 业务逻辑正确
   - [ ] 边界条件处理
   - [ ] 错误处理完善

3. 性能
   - [ ] 没有 N+1 查询
   - [ ] 算法复杂度合理
   - [ ] 使用缓存（如适用）

4. 可维护性
   - [ ] 代码可读性好
   - [ ] 命名清晰准确
   - [ ] 没有重复代码
```

#### 作为被审查者

```bash
# 提交前检查
1. 格式化代码
   npm run format

2. 运行 linter
   npm run lint

3. 运行测试
   npm test

4. 自我审查
   Agent(code-reviewer, "自我审查")

5. 提交 PR
```

## 📊 效果评估

### Code Review 指标

| 指标 | 目标 | 测量方法 |
|------|------|----------|
| PR 大小 | <400 行 | 统计每次 PR 的变更行数 |
| 响应时间 | <24 小时 | 从 PR 创建到首次审查的时间 |
| 修复时间 | <48 小时 | 从评论到修复的时间 |
| 通过率 | >70% | 一次通过审查的 PR 比例 |
| 覆盖率 | >90% | 经过审查的代码比例 |

### 质量指标

| 指标 | 目标 | 说明 |
|------|------|------|
| Bug 率 | <5% | Code Review 后发现的 bug 比例 |
| 安全漏洞 | 0 | Code Review 后发现的安全漏洞 |
| 测试覆盖率 | >80% | 测试覆盖的代码比例 |
| 代码复杂度 | <10 | 圈复杂度平均值 |

## 🔗 相关资源

### 内部资源

- [Code Review Guide](./code-review-guide.md) - 详细指南
- [实战示例](./examples/) - 4 个完整场景
- [场景分析](./examples/code-review-scenarios.md) - 应用场景
- [Code Reviewer Agent](../../02-agents/03-code-reviewer/) - 自动化工具

### 规则和标准

- [安全指南](../../rules/common/security.md) - 安全检查清单
- [编码规范](../../rules/common/coding-style.md) - 代码风格
- [测试要求](../../rules/common/testing.md) - 测试标准
- [Git 工作流](../../rules/common/git-workflow.md) - 提交规范

### 外部资源

- [Google Code Review Guide](https://google.github.io/eng-practices/review/)
- [Effective Code Review](https://martinfowler.com/articles/effective-code-review.html)
- [Code Review Best Practices](https://github.com/blog/2014-10-30-code-review-is-gold)

## 💡 最佳实践总结

### 团队层面

1. **建立规范**
   - 明确审查范围和标准
   - 定义响应时间要求
   - 制定优先级规则

2. **自动化工具**
   - 集成静态分析工具
   - 配置自动化测试
   - 使用 Code Reviewer Agent

3. **持续改进**
   - 定期回顾审查流程
   - 收集团队反馈
   - 优化审查效率

### 个人层面

1. **主动审查**
   - 及时响应 PR
   - 提供具体反馈
   - 保持建设性态度

2. **质量提交**
   - 保持 PR 小而聚焦
   - 包含充分测试
   - 提供清晰的描述

3. **持续学习**
   - 从反馈中学习
   - 阅读他人的代码
   - 分享最佳实践

---

**记住：** Code Review 的目标是改进代码和团队，而不是批评作者。保持建设性和尊重的沟通，共同提高代码质量。

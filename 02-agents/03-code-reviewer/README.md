# Code Reviewer Agent

Code Reviewer Agent 是专门用于代码审查的智能代理，能够自动检测代码中的问题并提供改进建议。

## 🎯 功能特性

### 自动化审查

- **安全检查** - SQL 注入、XSS、认证授权问题
- **性能分析** - N+1 查询、重复计算、内存泄漏
- **代码质量** - 命名规范、复杂度、重复代码
- **错误处理** - 异步错误、资源管理、异常传播
- **最佳实践** - API 设计、设计模式、编码标准

### 多语言支持

- JavaScript / TypeScript
- Python
- Go
- Java
- Rust

### 智能分析

- 上下文感知分析
- 优先级分类（CRITICAL、HIGH、MEDIUM、LOW）
- 可操作的改进建议
- 代码示例对比

## 🚀 使用方法

### 基本用法

```bash
# 方式 1: 直接调用 agent
Agent(code-reviewer, "审查用户认证模块的代码")

# 方式 2: 指定文件范围
Agent(code-reviewer, "审查 src/auth/ 目录下的所有文件")

# 方式 3: 聚焦特定类型
Agent(code-reviewer, "只检查安全问题")
```

### 高级用法

```bash
# 对比两个版本的差异
Agent(code-reviewer, """
审查以下代码变更：
- 新增文件：src/api/user-controller.js
- 修改文件：src/services/user-service.js
- 删除文件：src/legacy/auth.js
""")

# 结合 Git diff
git diff main...feature/user-auth | Agent(code-reviewer, "审查这些变更")

# 审查特定功能
Agent(code-reviewer, """
审查支付功能的代码，特别关注：
1. 金额计算的准确性
2. 幂等性保证
3. 错误处理
4. 事务管理
""")
```

## 📋 审查检查清单

### 安全检查（CRITICAL）

```javascript
// ✅ Code Reviewer 会自动检查
class UserService {
  async createUser(userData) {
    // ❌ SQL 注入风险
    const query = `INSERT INTO users VALUES ('${userData.email}')`;
    // Code Reviewer: 检测到 SQL 注入风险
    // 建议：使用参数化查询
  }
}
```

### 性能检查（HIGH）

```javascript
// ✅ Code Reviewer 会自动检测
class DataLoader {
  async loadUsers(userId) {
    const posts = await this.getPosts(userId);

    // ❌ N+1 查询
    const results = [];
    for (const post of posts) {
      const author = await this.getUser(post.authorId);
      results.push({ post, author });
    }
    // Code Reviewer: 检测到 N+1 查询模式
    // 建议：使用批量查询
  }
}
```

### 错误处理（HIGH）

```javascript
// ✅ Code Reviewer 会检查异步错误处理
class Processor {
  async processData(data) {
    // ❌ 没有错误处理
    const result = await this.externalAPI.call(data);
    return result;

    // Code Reviewer: 未处理的 Promise rejection
    // 建议：添加 try-catch 或使用 .catch()
  }
}
```

## 🔍 输出格式

### 问题报告

```markdown
## Code Review 报告

### 🔴 CRITICAL 问题 (2)

#### 1. SQL 注入漏洞
文件: `src/user/user-repository.js:8`
类型: 安全

问题：直接拼接用户输入到 SQL 查询中

攻击示例：
```javascript
email = "admin' OR '1'='1"
```

建议：使用参数化查询
```javascript
const query = 'SELECT * FROM users WHERE email = ?';
return await this.db.execute(query, [email]);
```

---

#### 2. 未处理的 Promise rejection
文件: `src/api/controller.js:15`
类型: 错误处理

问题：异步操作没有错误处理

建议：
```javascript
try {
  const result = await this.service.process(data);
  return res.json(result);
} catch (error) {
  return res.status(500).json({ error: error.message });
}
```

### 🟠 HIGH 问题 (3)

#### 1. N+1 查询
文件: `src/data/loader.js:20`
类型: 性能

问题：在循环中执行数据库查询

建议：使用批量查询
```javascript
const authorIds = posts.map(p => p.authorId);
const authors = await this.getUsersByIds(authorIds);
```

---

### 🟡 MEDIUM 问题 (4)

#### 1. 函数复杂度过高
文件: `src/utils/formatter.js:45`
类型: 代码质量

问题：函数有 15 条分支，建议拆分

建议：提取子函数或简化逻辑

---

## 总结

- 🔴 CRITICAL: 2 个（必须修复）
- 🟠 HIGH: 3 个（强烈建议修复）
- 🟡 MEDIUM: 4 个（建议修复）
- 🟢 LOW: 1 个（可选改进）

总体评分: 6.5/10

优先修复项：
1. 修复 SQL 注入漏洞（安全关键）
2. 添加错误处理（防止应用崩溃）
3. 优化 N+1 查询（性能问题）
```

## 🎓 最佳实践

### 1. 审查前准备

```bash
# 确保代码格式化
npm run format

# 运行 linter
npm run lint

# 运行测试
npm test

# 然后运行 Code Reviewer
Agent(code-reviewer, "审查最新的代码变更")
```

### 2. 分阶段审查

```bash
# 阶段 1: 安全审查（CRITICAL）
Agent(code-reviewer, "只检查安全问题")

# 修复 CRITICAL 问题后...

# 阶段 2: 性能审查（HIGH）
Agent(code-reviewer, "只检查性能问题")

# 修复 HIGH 问题后...

# 阶段 3: 全面审查
Agent(code-reviewer, "完整代码审查")
```

### 3. 集成到工作流

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

      - name: Setup Claude Code
        uses: anthropics/claude-code-action@v1

      - name: Run Code Reviewer
        run: |
          Agent(code-reviewer, "审查 PR 中的代码变更")

      - name: Post Review Comments
        uses: actions/github-script@v6
        with:
          script: |
            const reviewOutput = process.env.REVIEW_OUTPUT;
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: reviewOutput
            });
```

## 📊 指标和统计

### 审查覆盖率

```bash
# 查看审查统计
Agent(code-reviewer, "显示审查统计信息")

# 输出示例：
审查统计：
- 总文件数: 25
- 已审查: 25
- 覆盖率: 100%

问题统计：
- CRITICAL: 5
- HIGH: 12
- MEDIUM: 28
- LOW: 15

代码质量评分: 7.2/10
```

### 趋势分析

```bash
# 对比历史审查
Agent(code-reviewer, "对比上周的代码质量趋势")

# 输出示例：
代码质量趋势：
- 上周评分: 6.8/10
- 本周评分: 7.2/10
- 提升: +0.4

问题趋势：
- CRITICAL: -2 (改善)
- HIGH: -5 (改善)
- MEDIUM: +3 (需要注意)
```

## 🔧 配置选项

### 自定义规则

```javascript
// .claude-code-reviewer.config.js
module.exports = {
  rules: {
    security: {
      enabled: true,
      severity: 'CRITICAL'
    },
    performance: {
      enabled: true,
      severity: 'HIGH',
      nPlus1Query: {
        maxDepth: 3,
        ignorePatterns: ['test/**/*']
      }
    },
    complexity: {
      enabled: true,
      maxCyclomaticComplexity: 10
    },
    naming: {
      enabled: true,
      patterns: {
        camelCase: ['variables', 'functions'],
        PascalCase: ['classes', 'components']
      }
    }
  },
  ignore: [
    'node_modules/**',
    'dist/**',
    'coverage/**'
  ],
  outputFormat: 'markdown'
};
```

### 语言特定配置

```javascript
// JavaScript/TypeScript 配置
{
  javascript: {
    rules: {
      'no-sql-injection': 'error',
      'no-eval': 'error',
      'no-implied-eval': 'error',
      'prefer-const': 'warn',
      'no-var': 'warn'
    }
  }
}

// Python 配置
{
  python: {
    rules: {
      'no-sql-injection': 'error',
      'no-eval': 'error',
      'use-f-strings': 'warn',
      'no-print-statement': 'warn'
    }
  }
}
```

## 🎯 使用场景

### 场景 1: PR 自动审查

```bash
# 在 PR 创建时自动触发
.github/workflows/pr-review.yml
→ 拉取代码
→ 运行 Code Reviewer
→ 发布评论到 PR
```

### 场景 2: 本地开发审查

```bash
# 开发过程中实时审查
npm run dev -- --watch
# 同时运行
Agent(code-reviewer, "监控 src/ 目录的变更并实时审查")
```

### 场景 3: 合并前最终审查

```bash
# 在合并到 main 分支前
git checkout main
git pull origin main
git checkout feature-branch
git rebase main

# 运行完整审查
Agent(code-reviewer, "完整代码审查，包括安全、性能和质量检查")
```

### 场景 4: 重构前评估

```bash
# 重构前先审查
Agent(code-reviewer, "审查即将重构的模块，标记所有依赖和问题")

# 执行重构

# 重构后验证
Agent(code-reviewer, "验证重构后的代码，确保没有引入新问题")
```

## 📈 与其他 Agent 配合

### 与 Planner Agent 配合

```bash
# 1. 先规划
Agent(planner, "规划用户认证系统的重构")

# 2. 根据 planner 的建议执行开发

# 3. 完成后审查
Agent(code-reviewer, "审查重构后的代码，确保符合安全要求")
```

### 与 TDD-Guide Agent 配合

```bash
# 1. 使用 TDD 开发新功能
Agent(tdd-guide, "实现新的支付功能")

# 2. 完成后审查
Agent(code-reviewer, "审查支付功能代码，特别关注准确性和安全性")
"```

### 与 Security-Reviewer Agent 配合

```bash
# 1. 代码审查
Agent(code-reviewer, "完整代码审查")

# 2. 深度安全审查
Agent(security-reviewer, "深度安全审查和漏洞扫描")
```

## 🔗 相关资源

- [Code Review Examples](../01-commands/03-code-review/examples/) - 实战案例
- [Security Guidelines](../../rules/common/security.md) - 安全指南
- [Coding Standards](../../rules/common/coding-style.md) - 编码规范

---

**提示：** Code Reviewer Agent 是开发流程的重要补充，但不能完全替代人工审查。建议将其作为辅助工具，提高审查效率和质量。

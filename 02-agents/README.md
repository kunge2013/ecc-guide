# 代理 (Agents)

## 什么是 ECC 代理？

ECC 代理（Agents）是专用的子代理，可以处理特定的开发任务。代理由命令或其他工具自动调用，执行复杂的分析和决策任务，然后执行相应的操作。

## 代理 vs 命令 vs 技能

| 组件 | 触发方式 | 用途 | 示例 |
|-------|---------|------|------|
| **命令** | 用户输入 `/command` | 启动特定的开发工作流 | `/tdd`、`/plan`、`/code-review` |
| **代理** | 由命令或其他工具自动调用 | 执行特定的自动化任务 | `t`-guide`、`planner`、`code-review` |
| **技能** | 被 Claude 自动加载 | 提供领域知识和最佳实践指导 | `python-patterns`、`springboot-patterns` |

## 代理分类

### 1. 开发工作流代理（核心）
这些代理构成了 ECC 的核心开发工作流：

- **[planner](./01-planner/README.md)** - 实现规划代理
- **[tdd-guide](./02-tdd-guide/README.md)** - 测试驱动开发指导代理
- **[code-reviewer](./03-code-reviewer/README.md)** - 代码审查代理

### 2. 质量保证代理
- **[e2e-runner](./04-e2e-runner/README.md)** - 端到端测试执行代理

### 3. 语言特定审查代理
- **[java-reviewer](./05-language-reviewers/java-reviewer.md)** - Java 代码审查
- **[python-reviewer](./05-language-reviewers/python-reviewer.md)** - Python 代码审查
- **[go-reviewer](./05-language-reviewers/go-reviewer.md)** - Go 代码审查
- **[rust-reviewer](./05-language-reviewers/rust-reviewer.md)** - Rust 代码审查
- **[kotlin-reviewer](./05-language-reviewers/kotlin-reviewer.md)** - Kotlin 代码审查
- **[cpp-reviewer](./05-language-reviewers/cpp-reviewer.md)** - C++ 代码审查

### 4. 构建错误解决代理
- **[build-error-resolver](./06-specialized/build-error-resolver.md)** - 通用构建错误解决
- **[go-build-resolver](./06-specialized/go-build-resolver.md)** - Go 构建错误解决
- **[rust-build-resolver](./06-specialized/rust-build-resolver.md)** - Rust 构建错误解决
- **[kotlin-build-resolver](./06-specialized/kotlin-build-resolver.md)** - Kotlin 构建错误解决
- **[cpp-build-resolver](./06-specialized/cpp-build-resolver.md)** - C++ 构建错误解决
- **[java-build-resolver](./06-specialized/java-build-resolver.md)** - Java 构建错误解决

### 5. 专业代理
- **[architect](./06-specialized/architect.md)** - 系统架构设计
- **[security-reviewer](./06-specialized/security-reviewer.md)** - 安全审查
- **[database-reviewer](./06-specialized/database-reviewer.md)** - 数据库审查
- **[refactor-cleaner](./06-specialized/refactor-cleaner.md)** - 重构和代码清理
- **[docs-lookup](./06-specialized/docs-lookup.md)** - 文档查找
- **[doc-updater](./06-specialized/doc-updater.md)** - 文档更新
- **[harness-optimizer](./06-specialized/harness-optimizer.md)** - Harness 优化
- **[loop-operator](./06-specialized/loop-operator.md)** - 循环操作
- **[chief-of-staff](./06-specialized/chief-of-staff.md)** - 个人通信管理

## 代理工作原理

### 自动调用机制

代理通常由以下方式触发：

1. **由命令调用** - 例如，`/tdd` 命令会调用 `tdd-guide` 代理
2. **由其他代理调用** - 例如，`planner` 代理完成后可能调用 `tdd-guide`
3. **由工作流自动调用** - 例如，代码提交后自动触发 `code-reviewer`
4. **由工具调用** - 例如，构建失败时自动调用 `build-error-resolver`

### 代理执行流程

```
用户执行命令
    ↓
命令分析需求
    ↓
调用相应代理
    ↓
代理执行任务
    ↓
返回结果或继续调用其他代理
    ↓
用户收到最终结果
```

## 核心代理详解

### 1. planner 代理

**用途**: 在编写代码前创建详细的实现计划

**何时被调用**:
- 用户执行 `/plan` 命令时
- 需要复杂规划的场景

**工作内容**:
- 分析需求并重述
- 分解实现步骤
- 识别依赖关系和风险
- 创建实现计划文档

### 2. tdd-guide 代理

**用途**: 指导测试驱动开发流程

**何时被调用**:
- 用户执行 `/tdd` 命令时
- 需要编写新功能时（ECC 会自动建议）

**工作内容**:
- 指导测试先行（RED）
- 指导最小实现（GREEN）
- 指导重构优化（IMPROVE）
- 确保测试覆盖率达标

### 3. code-reviewer 代理

**用途**: 审查代码质量、安全性和最佳实践

**何时被调用**:
- 用户执行 `/code-review` 命令时
- 代码提交后（如果配置了自动审查）

**工作内容**:
- 检查代码质量
- 识别安全问题
- 验证最佳实践
- 提供改进建议

### 4. e2e-runner 代理

**用途**: 生成和运行端到端测试

**何时被调用**:
- 用户执行 `/e2e` 命令时
- 需要验证完整用户流程时

**工作内容**:
- 生成 E2E 测试
- 使用浏览器或测试框架运行测试
- 验证关键用户流程
- 报告测试结果

## 语言特定代理

### Java 开发代理

**java-reviewer**: 专门审查 Java 代码
- 检查 Java 编码规范
- 验证 Spring Boot 最佳实践
- 检查 JPA/ Hibernate 使用
- 验证并发和安全性

**java-build-resolver**: 解决 Java 构建错误
- 修复 Maven/Gradle 配置问题
- 解决编译错误
- 修复依赖冲突

## 学习路径

1. **第一步**: 理解核心代理
   - [01. planner 代理](./01-planner/README.md)
   - [02. tdd-guide 代理](./02-tdd-guide/README.md)

2. **第二步**: 理解质量保证代理
   - [03. code-reviewer 代理](./03-code-reviewer/README.md)
   - [04. e2e-runner 代理](./04-e2e-runner/README.md)

3. **第三步**: 了解语言特定代理
   - [05. 语言特定审查代理](./05-language-reviewers/)

4. **第四步**: 了解专业代理
   - [06. 专业代理](./06-specialized/)

## 代理最佳实践

### 1. 理解代理职责
每个代理专注于特定的任务，了解每个代理的职责有助于更好地使用 ECC。

### 2. 信任代理建议
代理是基于最佳实践和经验设计的，通常它们的建议是值得采纳的。

### 3. 理解自动调用
了解代理何时被自动调用，可以更好地预期 ECC 的行为。

### 4. 结合命令使用
代理通常与命令配合使用，理解它们的配合关系很重要。

## 下一步

- 开始学习 [01. planner 代理](./01-planner/README.md)
- 或查看 [学习路线图](../学习路线图.md)

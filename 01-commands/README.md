# 命令 (Commands)

## 什么是 ECC 命令？

ECC 命令是用户可以直接调用的斜杠命令（格式为 `/command`），用于启动特定的开发工作流。命令是 ECC 与用户交互的主要方式，提供了一套标准化的开发流程。

## 命令 vs 代理 vs 技能

| 组件 | 触发方式 | 用途 | 示例 |
|-------|---------|------|------|
| **命令** | 用户输入 `/command` | 启动特定的开发工作流 | `/tdd`、`/plan`、`/code-review` |
| **代理** | 由命令或其他工具自动调用 | 执行特定的自动化任务 | `tdd-guide`、`planner`、`code-reviewer` |
| **技能** | 被 Claude 自动加载 | 提供领域知识和最佳实践指导 | `python-patterns`、`springboot-patterns` |

## 命令分类

### 1. 开发工作流命令（核心）
这些命令构成了 ECC 的核心开发工作流：

- **[/plan](./01-plan/README.md)** - 创建实现计划
- **[/tdd](./02-tdd/README.md)** - 测试驱动开发
- **[/code-review](./03-code-review/README.md)** - 代码审查
- **[/e2e](./04-e2e/README.md)** - 端到端测试

### 2. 构建和修复命令
- **[/build-fix](./05-build-fix/README.md)** - 修复构建错误

### 3. 学习和提取命令
- **[/learn](./06-learn/README.md)** - 提取可重用模式

### 4. 语言特定命令
- Java: `/kotlin-build`、`/java-review`
- Python: `/python-review`
- Go: `/go-build`、`/go-review`
- Rust: `/rust-build`、`/rust-review`
- Kotlin: `/kotlin-build`、`/kotlin-review`
- C++: `/cpp-build`、`/cpp-review`

### 5. 高级命令
- `/multi-plan` - 多模型协作规划
- `/multi-execute` - 多模型协作执行
- `/multi-workflow` - 多模型协作开发
- `/devfleet` - 并行 Claude Code 代理
- `/verify` - 验证命令
- `/quality-gate` - 质量门禁

### 6. 会话管理命令
- `/save-session` - 保存会话
- `/resume-session` - 恢复会话
- `/sessions` - 管理会话
- `/checkpoint` - 检查点

### 7. 文档和学习命令
- `/docs` - 文档查找
- `/update-docs` - 更新文档
- `/update-codemaps` - 更新代码图

### 8. Instinct 管理命令
- `/instinct-status` - 显示学习到的本能
- `/instinct-import` - 导入本能
- `/instinct-export` - 导出本能
- `/promote` - 提升项目作用域本能
- `/evolve` - 分析并建议演进结构

### 9. 测试和评估命令
- `/test-coverage` - 测试覆盖率
- `/eval` - 评估命令

### 10. 工具和配置命令
- `/harness-audit` - Harness 审计
- `/setup-pm` - 设置包管理器
- `/pm2` - PM2 初始化
- `/model-route` - 模型路由
- `/loop-start` - 循环开始
- `/loop-status` - 循环状态

## 核心工作流

典型的 ECC 开发工作流使用以下核心命令：

```
1. /plan          → 创建实现计划
2. /tdd           → 测试驱动实现
3. /code-review   → 代码审查
4. /e2e           → 端到端测试
5. git commit     → 提交代码
```

## 使用建议

### 何时使用命令

1. **开始新功能** → 使用 `/plan`
2. **修复 Bug** → 先 `/plan` 然后 `/tdd`
3. **重构代码** → 使用 `/plan` 制定重构计划
4. **代码审查** → 使用 `/code-review`
5. **构建失败** → 使用 `/build-fix`
6. **提取模式** → 使用 `/learn`

### 命令使用顺序

- 总是先使用 `/plan` 来规划工作
- 然后使用 `/tdd` 来实现
- 使用 `/code-review` 保证质量
- 使用 `/e2e` 验证完整流程

## 学习路径

1. **第一步**: 学习核心命令
   - [01. /plan 命令](./01-plan/README.md)
   - [02. /tdd 命令](./02-tdd/README.md)

2. **第二步**: 学习质量保证命令
   - [03. /code-review 命令](./03-code-review/README.md)
   - [04. /e2e 命令](./04-e2e/README.md)

3. **第三步**: 学习辅助命令
   - [05. /build-fix 命令](./05-build-fix/README.md)
   - [06. /learn 命令](./06-learn/README.md)

4. **第四步**: 学习语言特定命令
   - [语言特定命令](./07-language-commands/)

## 练习建议

1. 完成每个命令模块的练习
2. 在 Java 实践项目中应用所学命令
3. 记录每个命令的使用场景和效果
4. 逐步掌握完整的工作流

## 下一步

- 开始学习 [01. /plan 命令](./01-plan/README.md)
- 或查看 [学习路线图](../学习路线图.md)

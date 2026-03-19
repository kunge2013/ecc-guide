# ECC Guide - Everything Claude Code 实践指南

欢迎使用 Everything Claude Code (ECC) 学习工程！

## 项目简介

这是一个结构化的学习工程，旨在帮助开发者学习和掌握 Everything Claude Code (ECC) 的各种功能。ECC 是一个 Claude Code 插件，提供生产就绪的代理、技能、命令、规则和 MCP 配置，用于专业的软件开发工作流程。

## 学习目标

通过本学习工程，你将：

- ✅ 掌握 ECC 的核心概念和架构
- ✅ 学会使用核心命令（`/plan`, `/tdd`, `/code-review`, `/e2e`）
- ✅ 理解 ECC 代理的工作原理
- ✅ 掌握 Java Spring Boot 项目的 TDD 实践
- ✅ 完成从规划到部署的完整开发周期
- ✅ 将 ECC 融入到真实的工作流程中

## 项目结构

```
ecc-guide/
├── README.md                          # 本文件
├── ecc使用手册.md                      # ECC 完整使用手册
├── 学习路线图.md                       # 循序渐进学习路线
├── 01-commands/                       # 命令学习模块
│   ├── README.md                       # 命令总览
│   ├── 01-plan/                       # /plan 命令
│   ├── 02-tdd/                        # /tdd 命令
│   ├── 03-code-review/                # /code-review 命令
│   ├── 04-e2e/                        # /e2e 命令
│   ├── 05-build-fix/                  # /build-fix 命令
│   ├── 06-learn/                      # /learn 命令
│   └── 07-language-commands/          # 语言特定命令
├── 02-agents/                         # 代理学习模块
│   ├── README.md                       # 代理总览
│   ├── 01-planner/                    # planner 代理
│   ├── 02-tdd-guide/                  # tdd-guide 代理
│   ├── 03-code-reviewer/              # code-reviewer 代理
│   ├── 04-e2e-runner/                 # e2e-runner 代理
│   ├── 05-language-reviewers/          # 语言特定审查代理
│   └── 06-specialized/                # 其他专业代理
├── 03-skills/                         # 技能学习模块
│   ├── README.md                       # 技能总览
│   ├── 01-java/                       # Java 开发技能
│   ├── 02-python/                     # Python 开发技能
│   ├── 03-go/                         # Go 开发技能
│   ├── 04-rust/                       # Rust 开发技能
│   └── 05-common/                     # 通用技能
├── 04-practice/                       # 实践项目
│   ├── java-springboot-demo/           # Java Spring Boot 实践项目
│   ├── python-django-demo/             # Python Django 实践项目
│   ├── go-api-demo/                   # Go API 实践项目
│   └── rust-api-demo/                 # Rust API 实践项目
├── 05-scenarios/                      # 完整使用场景案例
│   ├── 01-new-feature-development/     # 新功能开发场景
│   ├── 02-bug-fix-memory-leak/        # Bug 修复场景
│   ├── 03-refactoring-api-client/      # 重构场景
│   ├── 04-multi-language-project/     # 多语言项目场景
│   ├── 05-technical-debt-cleanup/     # 技术债务清理场景
│   └── 06-team-code-review/           # 团队代码审查场景
├── 06-workflows/                      # 工作流程指南
│   ├── feature-development.md           # 功能开发流程
│   ├── bug-fix.md                     # Bug 修复流程
│   ├── refactoring.md                 # 重构流程
│   ├── decision-tree.md                # 决策树
│   └── best-practices.md              # 最佳实践
└── 07-exercises/                      # 练习题
    ├── beginner/                       # 初级练习
    ├── intermediate/                   # 中级练习
    └── advanced/                      # 高级练习
```

## 快速开始

### 第一步：了解 ECC

如果你还不了解 ECC，建议先阅读：

1. 📖 [ECC 使用手册](./ecc使用手册.md) - 了解 ECC 的完整功能
2. 📖 [学习路线：第一周](./学习路线图.md#第一周基础入门) - 开始学习之旅

### 第二步：选择你的学习路径

根据你的经验和目标选择合适的学习路径：

#### 路径 A：初学者路径（推荐）
适合刚开始使用 ECC 的开发者

1. 📖 阅读 [学习路线图](./学习路线图.md)
2. 🚀 完成初级练习（[01-exercises/beginner](./07-exercises/beginner/)）
3. 🛏️ 在 Java 实践项目中练习（[04-practice/java-springboot-demo](./04-practice/java-springboot-demo/)）

#### 路径 B：Java 开发者路径
适合主要使用 Java 的开发者

1. 📖 学习 [Java 技能](./03-skills/01-java/)
2. 🛏️ 在 [Java Spring Boot Demo](./04-practice/java-springboot-demo/) 中实践
3. 📝 完成 [中级练习](./07-exercises/intermediate/)

#### 路径 C：快速上手路径
适合有经验的开发者

1. 📖 阅读 [命令总览](./01-commands/README.md)
2. 📖 阅读 [完整场景示例](./05-scenarios/01-new-feature-development/)
3. 🛏️ 在你的项目中应用 ECC

### 第三步：开始实践

选择一个实践项目开始：

- **Java 开发者**: [Java Spring Boot Demo](./04-practice/java-springboot-demo/) ⭐
- **Python 开发者**: [Python Django Demo](./04-practice/python-django-demo/)
- **Go 开发者**: [Go API Demo](./04-practice/go-api-demo/)
- **Rust 开发者**: [Rust API Demo](./04-practice/rust-api-demo/)

## 核心概念

### 命令 (Commands)

ECC 命令是用户可以直接调用的斜杠命令（`/command`），用于启动特定的开发工作流。

**核心命令**:
- `/plan` - 创建实现计划
- `/tdd` - 测试驱动开发
- `/code-review` - 代码审查
- `/e2e` - 端到端测试

**了解更多**: [01-commands](./01-commands/)

### 代理 (Agents)

ECC 代理是专用的子代理，可以处理特定的开发任务。代理由命令或其他工具自动调用。

**核心代理**:
- `planner` - 实现规划代理
- `tdd-guide` - TDD 指导代理
- `code-reviewer` - 代码审查代理
- `e2e-runner` - E2E 测试执行代理

**了解更多**: [02-agents](./02-agents/)

### 技能 (Skills)

ECC 技能是包含领域知识和最佳实践的文档。技能会被 Claude 自动加载，提供特定技术栈或领域的指导。

**Java 技能**:
- `java-coding-standards` - Java 编码规范
- `springboot-patterns` - Spring Boot 架构模式
- `springboot-tdd` - Spring Boot TDD 实践

**了解更多**: [03-skills](./03-skills/)

## 典型工作流

ECC 的典型开发工作流：

```
1. /plan          → 创建实现计划
2. /tdd           → 测试驱动实现
3. /code-review   → 代码审查
4. /e2e           → 端到端测试
5. git commit     → 提交代码
```

## 学习资源

### 文档
- [ECC 使用手册](./ecc使用手册.md) - 完整的功能参考
- [学习路线图](./学习路线图.md) - 循序渐进学习指南
- [工作流程指南](./06-workflows/) - 不同场景的工作流程

### 实践
- [Java Spring Boot Demo](./04-practice/java-springboot-demo/) - 主要实践项目
- [练习题集](./07-exercises/) - 初级/中级/高级练习

### 案例
- [场景案例](./05-scenarios/) - 6 个完整的使用场景

## 学习支持

### 遇到问题？

1. 📖 先查阅相关文档
2. 🎬 查看场景案例
3. 📝 尝试练习题
4. 💡 在你的项目中实践

### 获取帮助

- 📖 查看 [学习路线图](./学习路线图.md)
- 🎬 查看 [完整场景示例](./05-scenarios/01-new-feature-development/)
- 📝 完成 [练习题](./07-exercises/)

## 贡献

欢迎为 ECC 学习工程贡献内容！

1. Fork 本项目
2. 创建你的学习笔记和示例
3. 提交 Pull Request

## 许可证

本项目采用 MIT 许可证。

## 开始学习

准备好开始学习了吗？选择一个起点：

- 🚀 [从学习路线图开始](./学习路线图.md) - 循序渐进学习
- 📖 [从命令学习开始](./01-commands/) - 学习核心命令
- 🛏️ [从实践项目开始](./04-practice/java-springboot-demo/) - 动手实践
- 📝 [从练习开始](./07-exercises/beginner/) - 完成练习题

祝你学习愉快！🎉

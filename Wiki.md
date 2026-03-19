# ECC Guide - Everything Claude Code 实践指南 Wiki

## 项目概述

**ECC Guide** 是一个结构化的学习工程，旨在帮助开发者学习和掌握 **Everything Claude Code (ECC)** 的各种功能。ECC 是一个 Claude Code 插件，提供生产就绪的代理、技能、命令、规则和和 MCP 配置，用于专业的软件开发工作流程。

### 项目特点

- 🎓 **系统性学习** - 从基础概念到高级应用的完整学习路径
- 🛠️ **实战导向** - 通过真实项目练习掌握 ECC 功能
- 📚 **丰富资源** - 包含详细的文档、示例和最佳实践
- 🔄 **持续更新** - 跟随 ECC 最新发展

## 核心概念

### 命令 (Commands)

命令是用户可以直接调用的斜杠命令（`/command`），用于启动特定的开发工作流。

**核心命令：**
- `/plan` - 创建实现计划
- `/tdd` - 测试驱动开发
- `/code-review` - 代码审查
- `/e2e` - 端到端测试
- `/build-fix`` - 修复构建错误
- `/learn` - 提取可重用模式

**详细文档：** [01-commands/README.md](./01-commands/README.md)

### 代理 (Agents)

代理是专用的子代理，可以处理特定的开发任务。代理由命令或其他工具自动调用，执行复杂的分析和决策任务。

**核心代理：**
- `planner` - 实现规划代理
- `tdd-guide` - TDD 指导代理
- `code-reviewer` - 代码审查代理
- `e2e-runner` - E2E 测试执行代理

**详细文档：** [02-agents/README.md](./02-agents/README.md)

### 技能 (Skills)

技能是包含领域知识和最佳实践的文档。技能会被 Claude 自动加载，提供特定技术栈或领域的指导。

**主要技能：**
- **Java 技能** - `java-coding-standards`, `springboot-patterns`, `springboot-tdd`
- **Python 技能** - `python-patterns`, `python-testing`
- **Go 技能** - `golang-patterns`, `golang-testing`
- **Rust 技能** - `rust-patterns`, `rust-testing`
- **通用技能** - `tdd-workflow`, `e2e-testing`, `api-design`, `backend-patterns`

**详细文档：** [03-skills/README.md](./03-skills/README.md)

## 项目结构

```
ecc-guide/
├── README.md                          # 项目主文档
├── ecc使用手册.md                      # ECC 完整使用手册
├── 学习路线图.md                       # 循序渐进学习路线
├── 01-commands/                       # 命令学习模块
│   ├── README.md                       # 命令总览
│   ├── 01-plan/                       # /plan 命令
│   ├── 02-tdd/                        # /tdd 命令
│   ├── 03-code-review/                # /code-review 命令
│   ├── 04-e2e/                        # /e2e 命令
│   ├── 05-build-fix/                  # /build-fix 命令
│   └── 06-learn/                      # /learn 命令
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
│   └── 05-kotlin/                     # Kotlin 开发技能
├── 04-practice/                       # 实践项目
│   └── java-springboot-demo/           # Java Spring Boot 实践项目
├── 05-scenarios/                      # 完整使用场景案例
│   ├── 01-new-feature-development/     # 新功能开发场景
│   ├── 02-bug-fix-memory-leak/        # Bug 修复场景
│   ├── 03-refactoring-api-client/      # 重构场景
│   ├── 04-multi-language-project/     # 多语言项目场景
│   ├── 05-technical-debt-cleanup/     # 技术债务清理场景
│   └── 06-team-code-review/           # 团队代码审查场景
├── 06-workflows/                      # 工作流程指南
│   └── README.md                      # 工作流程总览
└── 07-exercises/                      # 练习题
    └── README.md                      # 练习总览
```

## 学习路径

### 🚀 快速开始路径

适合有经验的开发者，快速上手 ECC：

1. 📖 阅读 [ECC 使用手册](./ecc使用手册.md)
2. 📖 阅读 [命令总览](./01-commands/README.md)
3. 📖 阅读 [代理总览](./02-agents/README.md)
4. 🛏️ 在 [Java Spring Boot Demo](./04-practice/java-springboot-demo/) 中实践
5. 🎬 查看 [完整场景示例](./05-scenarios/01-new-feature-development/)

### 🎓 初学者路径（推荐）

适合刚开始使用 ECC 的开发者：

#### 第一周：基础入门
- Day 1-2: 学习 ECC 核心概念
  - [ECC 使用手册](./ecc使用手册.md)
  - [命令总览](./01-commands/README.md)
  - [代理总览](./02-agents/README.md)
  - [技能总览](./03-skills/README.md)

- Day 3-4: 学习核心命令
  - [/plan 命令](./01-commands/01-plan/README.md)
  - [/tdd 命令](./01-commands/02-tdd/README.md)

- Day 5-6: Java 实践环境
  - [Java Spring Boot Demo](./04-practice/java-springboot-demo/README.md)
  - 搭建开发环境，运行示例项目

- Day 7: 初级练习
  - 完成初级练习题
  - 在 Java 项目中使用 `/plan` 命令

#### 第二周：代理与技能
- Day 8-9: 深入学习代理
  - [planner 代理](./02-agents/01-planner/README.md)
  - [tdd-guide 代理](./02-agents/02-tdd-guide/README.md)
  - [code-reviewer 代理](./02-agents/03-code-reviewer/README.md)

- Day 10-11: Java 技能学习
  - [Java 编码标准](./03-skills/01-java/java-coding-standards/README.md)
  - [Spring Boot 模式](./03-skills/01-java/springboot-patterns/README.md)
  - [Spring Boot TDD](./03-skills/01-java/springboot-tdd/README.md)

- Day 12-13: TDD 实践
  - 在 Java 项目中完整实现一个新功能
  - 使用完整的 ECC 开发周期

- Day 14: 中级练习
  - 完成中级练习题
  - 复盘本周学到的内容

#### 第三周：完整工作流
- 学习完整的 ECC 开发工作流
- 理解不同场景下的最佳实践
- 学习高级命令和功能

#### 第四周：场景应用
- 学习 [场景案例](./05-scenarios/)
- 在真实项目中应用 ECC
- 完成高级练习

### 💻 Java 开发者路径

适合主要使用 Java 的开发者：

1. 📖 学习 Java 技能
   - [Java 编码标准](./03-skills/01-java/java-coding-standards/README.md)
   - [Spring Boot 模式](./03-skills/01-java/springboot-patterns/README.md)
   - [Spring Boot TDD](./03-skills/01-java/springboot-tdd/README.md)

2. 🛏️ 在 Java Spring Boot Demo 中实践
     - [项目说明](./04-practice/java-springboot-demo/README.md)
   - 完成项目中的所有练习任务

3. 📝 完成练习题
   - 初级练习
   - 中级练习
   - 高级练习

4. 🎬 学习 Java 相关场景
   - [新功能开发场景](./05-scenarios/01-new-feature-development/)
   - [Bug 修复场景](./05-scenarios/02-bug-fix-memory-leak/)

## 典型工作流

ECC 的典型开发工作流：

```
1. /plan          → 创建实现计划
2. /tdd           → 测试驱动实现
3. /code-review   → 代码审查
4. /e2e           → 端到端测试
5. git commit     → 提交代码
```

### 工作流示例

**场景：为任务管理系统添加标签功能**

```bash
# 步骤 1: 规划
用户: /plan 我需要为任务添加标签功能，支持为任务添加多个标签，可以按标签筛选任务

# 步骤 2: 实现
用户: /tdd 实现任务标签功能

# 步骤 3: 审查
用户: /code-review 审查任务标签功能的代码

# 步骤 4: 测试
用户: /e2e 生成并运行端到端测试

# 步骤 5: 提交
用户: git commit -m "feat: add task tags feature"
```

## 实践项目

### 主要实践项目：Java Spring Boot Demo

**项目名称**：任务管理系统 (Task Management API)

**技术栈：**
- Java 17+
- Spring Boot 3.x
- Spring Data JPA
- H2 Database
- Maven/Gradle
- JUnit 5

**项目功能：**
- 用户管理（创建、查询、更新、删除）
- 任务管理（创建、查询、更新、删除）
- 评论功能
- RESTful API

**项目详情：** [04-practice/java-springboot-demo/README.md](./04-practice/java-springboot-demo/README.md)

### 其他实践项目

- **Python Django Demo** - Python Django 实践项目（参考）
- **Go API Demo** - Go API 实践项目（参考）
- **Rust API Demo** - Rust API 实践项目（参考）

**实践项目总览：** [04-practice/README.md](./04-practice/README.md)

## 场景案例

本指南包含 6 个完整的使用场景案例：

1. **新功能开发** - [01-new-feature-development/](./05-scenarios/01-new-feature-development/)
   - 用户认证功能
   - 数据导出功能
   - 实时通知功能

2. **Bug 修复** - [02-bug-fix-memory-leak/](./05-scenarios/02-bug-fix-memory-le-leak/)
   - 内存泄漏定位和修复
   - 使用 ECC 进行系统化调试

3. **重构** - [03-refactoring-api-client/](./05-scenarios/03-refactoring-api-client/)
   - API 客户端重构
   - 保持行为一致性的重构策略

4. **多语言项目** - [04-multi-language-project/](./05-scenarios/04-multi-language-project/)
   - 在多语言项目中使用 ECC
   - 不同技术栈的最佳实践

5. **技术债务清理** - [05-technical-debt-cleanup/](./05-scenarios/05-technical-debt-cleanup/)
   - 系统化技术债务清理
   - 优先级排序和执行策略

6. **团队代码审查** - [06-team-code-review/](./05-scenarios/06-team-code-review/)
   - 使用 ECC 进行团队代码审查
   - 建立代码审查标准

## 学习资源

### 📚 文档资源

- [ECC 使用手册](./ecc使用手册.md) - 完整的功能参考
- [学习路线图](./学习路线图.md) - 循序渐进学习指南
- [命令总览](./01-commands/README.md) - 所有命令的详细说明
- [代理总览](./02-agents/README.md) - 所有代理的详细说明
- [技能总览](./03-skills/README.md) - 所有技能的详细说明

### 🛠️ 实践资源

- [Java Spring Boot Demo](./04-practice/java-springboot-demo/) - 主要实践项目
- [场景案例](./05-scenarios/) - 6 个完整的使用场景
- [练习题集](./07-exercises/) - 初级/中级/高级练习

### 💡 最佳实践

- [工作流程指南](./06-workflows/) - 不同场景的工作流程
- [代码审查最佳实践](./02-agents/03-code-reviewer/) - 代码审查指南
- [TDD 最佳实践](./01-commands/02-tdd/) - 测试驱动开发指南

## 贡献指南

欢迎为 ECC 学习工程贡献内容！

### 如何贡献

1. Fork 本项目
2. 创建你的学习笔记和示例
3. 提交 Pull Request

### 贡献类型

- 📝 完善文档内容
- 🛠️ 添加新的实践项目
- 🎬 添加新的场景案例
- 💡 分享使用心得和最佳实践
- 🐛 修复文档中的错误

## 常见问题

### Q: 我需要什么基础才能开始学习 ECC？

A: 基本的编程知识和对软件开发的了解即可。如果你熟悉 Java、Python、Go 或 Rust 中的一种语言，会更容易上手。

### Q: 学习 ECC 需要多长时间？

A: 取决于你的经验和学习速度。快速开始路径大约需要 1-2 天，完整的初学者路径大约需要 4 周。

### Q: 我必须使用 Java 吗？

A: 不是必须的。你可以根据自己的主要语言选择相应的实践项目和技能。但本学习工程以 Java 为主要示例语言。

### Q: ECC 可以用于哪些类型的项目？

A: ECC 可以用于大多数软件开发项目，包括 Web 应用、API 服务、移动应用、微服务架构等。支持多种编程语言和框架。

### Q: 如何获取帮助？

A:
1. 📖 查阅相关文档
2. 🎬 查看场景案例
3. 📝 尝试练习题
4. 💡 在你的项目中实践

## 许可证

本项目采用 MIT 许可证。

## 开始学习

准备好开始学习了吗？选择一个起点：

- 🚀 [从学习路线图开始](./学习路线图.md) - 循序渐进学习
- 📖 [从命令学习开始](./01-commands/) - 学习核心命令
- 🤖 [从代理学习开始](./02-agents/) - 理解 ECC 代理
- 📚 [从技能学习开始](./03-skills/) - 掌握领域知识
- 🛏️ [从实践项目开始](./04-practice/java-springboot-demo/) - 动手实践
- 🎬 [从场景案例开始](./05-scenarios/01-new-feature-development/) - 学习完整流程
- 📝 [从练习开始](./07-exercises/) - 完成练习题

祝你学习愉快！🎉

---

**最后更新：** 2026-03-20
**项目版本：** 1.0.0

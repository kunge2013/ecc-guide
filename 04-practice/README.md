# 实践项目

## 概述

本目录包含 ECC 学习工程中的实践项目，用于在真实项目中应用 ECC 的各种功能。

## 实践项目结构

```
04-practice/
├── README.md                              # 本文件
├── java-springboot-demo/                  # Java Spring Boot 实践项目（主要）
│   ├── README.md                          # 项目说明
│   ├── pom.xml                            # Maven 配置
│   ├── build.gradle                       # Gradle 配置
│   └── src/                               # 源代码
├── python-django-demo/                    # Python Django 实践项目（参考）
├── go-api-demo/                           # Go API 实践项目（参考）
└── rust-api-demo/                         # Rust API 实践项目（参考）
```

## 主要实践项目：Java Spring Boot Demo

### 项目概述

**项目名称**: 任务管理系统 (Task Management API)

**技术栈**:
- Java 17+
- Spring Boot 3.x
- Spring Data JPA
- H2 Database (开发环境)
- Maven/Gradle
- JUnit 5

**项目功能**:
- 用户管理（创建、查询、更新、删除）
- 任务管理（创建、查询、更新、删除）
- 评论功能
- RESTful API

### 为什么选择这个项目？

1. **简洁而完整**: 涵盖了 Web 应用的核心功能
2. **分层架构**: 清晰的分层结构，易于理解
3. **适合实践**: 每个功能都可以使用 ECC 完整开发
4. **可扩展**: 可以逐步添加新功能进行练习

### 项目结构

```
java-springboot-demo/
├── README.md                              # 项目说明和练习任务
├── pom.xml                                # Maven 配置
├── build.gradle                           # Gradle 配置
└── src/
    ├── main/
    │   ├── java/com/ecc/demo/
    │   │   ├── EccDemoApplication.java    # 主应用类
    │   │   ├── model/                     # 领域模型
    │   │   │   ├── User.java
    │   │   │   ├── Task.java
    │   │   │   └── Comment.java
    │   │   ├── repository/                # 数据访问层
    │   │   │   ├── UserRepository.java
    │   │   │   └── TaskRepository.java
    │   │   ├── service/                   # 业务逻辑层
    │   │   │   ├── UserService.java
    │   │   │   └── TaskService.java
    │   │   └── controller/                # REST API 层
    │   │       ├── UserController.java
    │   │       └── TaskController.java
    │   └── resources/
    │       └── application.yml             # 应用配置
    └── test/
        └── java/com/ecc/demo/
            ├── service/                   # 单元测试
            │   └── TaskServiceTest.java
            └── controller/                # 集成测试
                └── TaskControllerIntegrationTest.java
```

## 练习方法

### 第一步：环境准备

1. **安装开发工具**:
   - JDK 17+
   - Maven 或 Gradle
   - IDE（IntelliJ IDEA 推荐）

2. **克隆或创建项目**:
   ```bash
   cd 04-practice/java-springboot-demo
   ```

3. **构建项目**:
   ```bash
   # 使用 Maven
   mvn clean install

   # 或使用 Gradle
   ./gradlew build
   ```

4. **运行项目**:
   ```bash
   # 使用 Maven
   mvn spring-boot:run

   # 或使用 Gradle
   ./gradlew bootRun
   ```

### 第二步：学习现有代码

1. 阅读 `README.md` 了解项目结构
2. 查看 `model/` 目录理解领域模型
3. 查看 `service/` 目录理解业务逻辑
4. 查看 `controller/` 目录理解 API 设计
5. 查看 `test/` 目录理解测试方法

### 第三步：完成练习任务

项目 `README.md` 中包含了一系列练习任务，按顺序完成：

1. **练习 1**: 使用 `/plan` 计划新功能
2. **练习 2**: 使用 `/tdd` 实现功能
3. **练习 3**: 使用 `/code-review` 审查代码
4. **练习 4**: 使用 `/build-fix` 修复错误
5. **练习 5**: 使用 `/e2e` 生成测试

### 第四步：自主实践

在熟悉基本流程后，你可以：

1. 添加新功能（如任务标签功能）
2. 重构现有代码
3. 优化性能
4. 添加安全功能
5. 完善测试覆盖

## 其他语言实践项目

### Python Django Demo

如果你主要使用 Python，可以参考 `python-django-demo/` 目录。

**技术栈**:
- Python 3.x
- Django 4.x
- Django REST Framework
- pytest

### Go API Demo

如果你主要使用 Go，可以参考 `go-api-demo/` 目录。

**技术栈**:
- Go 1.21+
- Gin Web Framework
- GORM
- testify

### Rust API Demo

如果你主要使用 Rust，可以参考 `rust-api-demo/` 目录。

**技术栈**:
- Rust 1.70+
- Actix-web
- Diesel ORM
- cargo test

## ECC 在实践项目中的应用

### 典型开发流程

```
1. 需求分析
   ↓
2. /plan          → 制定实现计划
   ↓
3. /tdd           → 测试驱动实现
   ↓
4. /code-review   → 代码审查
   ↓
5. /e2e           → 端到端测试
   ↓
6. git commit     → 提交代码
```

### 具体示例

假设你要为任务管理系统添加一个新功能：**任务标签功能**

#### 步骤 1: 规划

```bash
用户: /plan 我需要为任务添加标签功能，支持为任务添加多个标签，可以按标签筛选任务

ECC: [生成详细的实现计划]
- 创建 Tag 实体
- 创建 Task 和 Tag 的多对多关系
- 更新 TaskService 支持标签操作
- 更新 TaskController 添加标签 API
- 编写测试
```

#### 步骤 2: 实现

```bash
用户: /tdd 实现任务标签功能

ECC: [调用 tdd-guide 代理]
- 指导先写测试（RED）
- 实现功能（GREEN）
- 重构优化（IMPROVE）
```

#### 步骤 3: 审查

```bash
用户: /code-review 审查任务标签功能的代码

ECC: [调用 code-reviewer 代理]
- 检查代码质量
- 识别安全问题
- 验证最佳实践
```

#### 步骤 4: 测试

```bash
用户: /e2e 生成并运行端到端测试

ECC: [调用 e2e-runner 代理]
- 生成测试脚本
- 使用浏览器运行测试
- 验证完整用户流程
```

## 学习建议

1. **从简单开始**: 先完成基本的 CRUD 练习
2. **循序渐进**: 逐步完成更复杂的练习
3. **记录过程**: 记录每个步骤的问题和解决方案
4. **对比学习**: 对比不使用 ECC 和使用 ECC 的差异
5. **持续实践**: 在真实项目中应用所学

## 常见问题

**Q: 我必须使用 Java 吗？**

A: 不是必须的。你可以根据自己的主要语言选择相应的实践项目。但本学习工程以 Java 为主，因为 Java 是企业级开发的主流语言。

**Q: 练习任务必须按顺序完成吗？**

A: 建议按顺序完成，因为后面的练习依赖于前面的知识。但如果你有经验，也可以跳过一些基础练习。

**Q: 完成所有练习需要多久？**

A: 取决于你的经验和学习速度。一般来说，完成所有练习需要 2-4 周。

**Q: 遇到问题怎么办？**

A: 可以参考项目文档、查看示例代码、或者查阅相关技能文档。

## 下一步

- 开始使用 [Java Spring Boot Demo](./java-springboot-demo/)
- 或查看 [学习路线图](../学习路线图.md)

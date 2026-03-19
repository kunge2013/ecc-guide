# 技能 (Skills)

## 什么是 ECC 技能？

ECC 技能（Skills）是包含领域知识和最佳实践的文档。技能会被 Claude 自动加载，提供特定技术栈或领域的指导。

## 技能 vs 命令 vs 代理

| 组件 | 触发方式 | 用途 | 示例 |
|-------|---------|------|------|
| **命令** | 用户输入 `/command` | 启动特定的开发工作流 | `/tdd`、`/plan`、`/code-review` |
| **代理** | 由命令或其他工具自动调用 | 执行特定的自动化任务 | `tdd-guide`、`planner`、`code-reviewer` |
| **技能** | 被 Claude 自动加载 | 提供领域知识和最佳实践指导 | `python-patterns`、`springboot-patterns` |

## 技能分类

### 1. Java 开发技能
- **[java-coding-standards](./01-java/java-coding-standards/README.md)** - Java 编码规范
- **[springboot-patterns](./01-java/springboot-patterns/README.md)** - Spring Boot 架构模式
- **[springboot-tdd](./01-java/springboot-tdd/README.md)** - Spring Boot TDD 实践
- **[springboot-verification](./01-java/springboot-verification/README.md)** - Spring Boot 验证循环

### 2. Python 开发技能
- **[python-patterns](./02-python/python-patterns/README.md)** - Python 编码模式
- **[python-testing](./02-python/python-testing/README.md)** - Python 测试策略

### 3. Go 开发技能
- **[golang-patterns](./03-go/golang-patterns/README.md)** - Go 编码模式
- **[golang-testing](./03-go/golang-testing/README.md)** - Go 测试模式

### 4. Rust 开发技能
- **[rust-patterns](./04-rust/rust-patterns/README.md)** - Rust 编码模式
- **[rust-testing](./04-rust/rust-testing/README.md)** - Rust 测试模式

### 5. Kotlin 开发技能
- **[kotlin-patterns](./05-kotlin/kotlin-patterns/README.md)** - Kotlin 编码模式
- **[kotlin-testing](./05-kotlin/kotlin-testing/README.md)** - Kotlin 测试模式
- **[kotlin-coroutines-flows](./05-kotlin/kotlin-coroutines-flows/README.md)** - Kotlin 协程和 Flow
- **[kotlin-ktor-patterns](./05-kotlin/kotlin-ktor-patterns/README.md)** - Ktor 服务器模式
- **[kotlin-exposed-patterns](./05-kotlin/kotlin-exposed-patterns/README.md)** - Exposed ORM 模式
- **[compose-multiplatform-patterns](./05-kotlin/compose-multiplatform-patterns/README.md)** - Compose Multiplatform 模式
- **[android-clean-architecture](./05-kotlin/android-clean-architecture/README.md)** - Android 清洁架构

### 6. C++ 开发技能
- **[cpp-coding-standards](./06-cpp/cpp-coding-standards/README.md)** - C++ 编码标准
- **[cpp-testing](./06-cpp/cpp-testing/README.md)** - C++ 测试策略

### 7. Django 开发技能
- **[django-patterns](./07-django/django-patterns/README.md)** - Django 架构模式
- **[django-tdd](./07-django/django-tdd/README.md)** - Django TDD 策略
- **[django-verification](./07-django/django-verification/README.md)** - Django 验证循环

### 8. Laravel 开发技能
- **[laravel-patterns](./08-laravel/laravel-patterns/README.md)** - Laravel 架构模式
- **[laravel-tdd](./08-laravel/laravel-tdd/README.md)** - Laravel TDD 策略
- **[laravel-verification](./08-laravel/laravel-verification/README.md)** - Laravel 验证循环

### 9. 后端开发技能（通用）
- **[backend-patterns](./09-backend/backend-patterns/README.md)** - 后端架构模式
- **[api-design](./09-backend/api-design/README.md)** - REST API 设计模式
- **[docker-patterns](./09-backend/docker-patterns/README.md)** - Docker 模式
- **[deployment-patterns](./09-backend/deployment-patterns/README.md)** - 部署模式

### 10. 数据库技能
- **[postgres-patterns](./10-database/postgres-patterns/README.md)** - PostgreSQL 模式
- **[database-migrations](./10-database/database-migrations/README.md)** - 数据库迁移最佳实践

### 11. 安全技能
- **[security-review](./11-security/security-review/README.md)** - 安全审查
- **[security-scan](./11-security/security-scan/README.md)** - 安全扫描

### 12. AI 和机器学习技能
- **[claude-api](./12-ai/claude-api/README.md)** - Claude API 模式
- **[ai-regression-testing](./12-ai/ai-regression-testing/README.md)** - AI 回归测试
- **[ai-first-engineering](./12-ai/ai-first-engineering/README.md)** - AI 优先工程
- **[cost-aware-llm-pipeline](./12-ai/cost-aware-llm-pipeline/README.md)** - 成本感知 LLM 流水线

### 13. 测试技能（通用）
- **[tdd-workflow](./13-testing/tdd-workflow/README.md)** - 测试驱动开发工作流
- **[e2e-testing](./13-testing/e2e-testing/README.md)** - 端到端测试

### 14. 文档和内容技能
- **[article-writing](./14-docs/article-writing/README.md)** - 文章写作
- **[content-engine](./14-docs/content-engine/README.md)** - 内容引擎

## 技能工作原理

### 自动加载机制

技能通过以下方式被加载和使用：

1. **项目检测** - ECC 检测项目技术栈（如检测到 `pom.xml` 则加载 Java 技能）
2. **上下文匹配** - 当相关话题出现时，相应的技能被自动激活
3. **知识提供** - 技能提供特定领域的知识和最佳实践
4. **指导建议** - 根据技能内容，提供专业的建议和指导

### 技能激活场景

```
用户在 Java 项目中工作
    ↓
ECC 检测到 pom.xml 或 .java 文件
    ↓
自动加载 Java 相关技能
    ↓
用户编写代码时
    ↓
Java 技能提供编码规范建议
    ↓
用户编写测试时
    ↓
Spring Boot TDD 技能提供测试指导
```

## 核心技能详解

### 1. java-coding-standards

**用途**: 提供 Java 编码规范和最佳实践

**包含内容**:
- 命名约定
- 代码组织
- 异常处理
- 集合使用
- 并发编程
- 性能优化

**何时激活**: 在 Java 项目中编写代码时

### 2. springboot-patterns

**用途**: 提供 Spring Boot 架构模式

**包含内容**:
- 分层架构（Controller → Service → Repository）
- 依赖注入模式
- 配置管理
- 安全最佳 practices
- 数据访问模式

**何时激活**: 在 Spring Boot 项目中工作时

### 3. springboot-tdd

**用途**: 指导 Spring Boot 项目的测试驱动开发

**包含内容**:
- 单元测试策略
- 集成测试策略
- Mock 使用模式
- 测试覆盖率要求
- 测试组织

**何时激活**: 在 Spring Boot 项目中编写测试时

### 4. python-patterns

**用途**: 提供 Python 编码模式和最佳实践

**包含内容**:
- PEP 8 编码规范
- Pythonic 习语
- 类型提示使用
- 异步编程
- 性能优化

**何时激活**: 在 Python 项目中编写代码时

## 学习路径

根据你的主要开发语言，选择相应的学习路径：

### Java 开发者路径

1. **[java-coding-standards](./01-java/java-coding-standards/README.md)** - Java 编码规范
2. **[springboot-patterns](./01-java/springboot-patterns/README.md)** - Spring Boot 模式
3. **[springboot-tdd](./01-java/springboot-tdd/README.md)** - Spring Boot TDD

### Python 开发者路径

1. **[python-patterns](./02-python/python-patterns/README.md)** - Python 编码模式
2. **[python-testing](./02-python/python-testing/README.md)** - Python 测试策略

### Go 开发者路径

1. **[golang-patterns](./03-go/golang-patterns/README.md)** - Go 编码模式
2. **[golang-testing](./03-go/golang-testing/README.md)** - Go 测试模式

### 多语言开发者路径

1. 选择主要语言的技能
2. 学习通用技能：
   - **[tdd-workflow](./13-testing/tdd-workflow/README.md)**
   - **[e2e-testing](./13-testing/e2e-testing/README.md)**
   - **[api-design](./09-backend/api-design/README.md)**
   - **[backend-patterns](./09-backend/backend-patterns/README.md)**

## 技能最佳实践

### 1. 理解技能范围
每个技能专注于特定的领域或技术栈，了解其范围有助于更好地利用它们。

### 2. 信任技能建议
技能是基于行业最佳实践编写的，通常它们的建议是值得采纳的。

### 3. 结合实践学习
阅读技能文档后，在实际项目中应用所学知识。

### 4. 定期回顾
技术不断发展，定期回顾技能文档，更新你的知识。

## 技能与代理的关系

技能和代理相互配合：

```
技能提供知识
    ↓
代理执行任务
    ↓
代理使用技能知识
    ↓
提供更准确的建议
```

例如：
- `tdd-guide` 代理会使用 `springboot-tdd` 技能的知识来提供 Spring Boot 特定的 TDD 指导
- `java-reviewer` 代理会使用 `java-coding-standards` 技能的标准来审查 Java 代码

## 实用使用场景



### 1. Spring Boot REST API 开发

**场景**: 使用 Spring Boot 快速构建符合最佳实践的 REST API

**涉及技能**:
- `java-coding-standards` - Java 编码规范
- `springboot-patterns` - Spring Boot 架构模式
- `springboot-tdd` - Spring Boot TDD 实践

**完整示例**: [Spring Boot REST API 实战](./examples/springboot-rest-api.md)

---

### 2. Python FastAPI 微服务

**场景**: 使用 Python FastAPI 快速构建高性能微服务

**涉及技能**:
- `python-patterns` - Python 编码模式
- `python-testing` - Python 测试策略
- `backend-patterns` - 后端架构模式
- `docker-patterns` - Docker 模式

**完整示例**: [FastAPI 微服务实战](./examples/fastapi-microservice.md)

---

### 3. Go API 网关

**场景**: 使用 Go 构建高性能、生产级的 API 网关

**涉及技能**:
- `golang-patterns` - Go 编码模式
- `golang-testing` - Go 测试模式
- `backend-patterns` - 后端架构模式
- `api-design` - API 设计模式

**完整示例**: [Go API 网关实战](./examples/go-api-gateway.md)

---

### 4. Rust 数据处理管道

**场景**: 使用 Rust 构建安全、高性能的数据处理管道

**涉及技能**:
- `rust-patterns` - Rust 编码模式
- `rust-testing` - Rust 测试模式
- `backend-patterns` - 后端架构模式

**完整示例**: [Rust 数据处理管道实战](./examples/rust-data-pipeline.md)

---

### 5. 多语言微服务架构

**场景**: 构建包含多个技术栈的完整微服务架构

**涉及技能**:
- Java + Spring Boot
- Python + FastAPI
- Go + Chi Router
- Rust + Tokio
- Kotlin + Ktor

**完整示例**: [多语言微服务架构实战](./examples/multilanguage-microservices.md)

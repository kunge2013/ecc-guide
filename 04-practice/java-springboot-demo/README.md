# Java Spring Boot Demo - 任务管理系统

## 项目概述

这是一个基于 Spring Boot 的任务管理系统 REST API，用于演示和练习 ECC 的各种功能。

## 技术栈

- **Java**: 17+
- **Spring Boot**: 3.2.x
- **Spring Data JPA**: 数据访问
- **H2 Database**: 开发环境数据库
- **Maven**: 项目管理
- **JUnit 5**: 测试框架
- **AssertJ**: 断言库

## 项目功能

### 当前功能

1. **用户管理**
   - 创建用户
   - 查询用户
   - 更新用户
   - 删除用户

2. **任务管理**
   - 创建任务
   - 查询任务
   - 更新任务
   - 删除任务
   - 按状态筛选任务

3. **评论功能**
   - 为任务添加评论
   - 查询任务评论

### 待实现功能（用于练习）

1. **任务标签功能** ⭐
   - 创建、编辑、删除标签
   - 为任务添加多个标签
   - 按标签筛选任务

2. **用户认证** ⭐⭐
   - 用户注册和登录
   - JWT Token 认证
   - 权限管理

3. **任务统计** ⭐⭐
   - 任务统计 API
   - 按时间段统计
   - 按用户统计

4. **任务通知** ⭐⭐⭐
   - 任务状态变更通知
   - 任务截止提醒
   - 邮件通知

## 项目结构

```
src/
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

## 快速开始

### 前置条件

- JDK 17+
- Maven 3.6+

### 构建项目

```bash
# 克隆项目
cd 04-practice/java-springboot-demo

# 构建项目
mvn clean install

# 跳过测试构建
mvn clean install -DskipTests
```

### 运行项目

```bash
# 使用 Maven
mvn spring-boot:run

# 或先打包后运行
mvn package
java -jar target/ecc-demo-0.0.1-SNAPSHOT.jar
```

### 运行测试

```bash
# 运行所有测试
mvn test

# 运行特定测试
mvn test -Dtest=TaskServiceTest
```

## API 端点

### 用户 API

| 方法 | 端点 | 说明 |
|-----|------|------|
| POST | /api/users | 创建用户 |
| GET | /api/users/{id} | 获取用户 |
| GET | /api/users | 获取所有用户 |
| PUT | /api/users/{id} | 更新用户 |
| DELETE | /api/users/{id} | 删除用户 |

### 任务 API

| 方法 | 端点 | 说明 |
|-----|------|------|
| POST | /api/tasks | 创建任务 |
| GET | /api/tasks/{id} | 获取任务 |
| GET | /api/tasks | 获取所有任务 |
| GET | /api/tasks?status={status} | 按状态筛选任务 |
| PUT | /api/tasks/{id} | 更新任务 |
| DELETE | /api/tasks/{id} | 删除任务 |

### 评论 API

| 方法 | 端点 | 说明 |
|-----|------|------|
| POST | /api/tasks/{taskId}/comments | 添加评论 |
| GET | /api/tasks/{taskId}/comments | 获取任务评论 |

## 练习任务

### 练习 1: 使用 /plan 规划任务标签功能 ⭐

**目标**: 学习如何使用 `/plan` 命令规划新功能

**步骤**:
1. 阅读需求（见上方"待实现功能"）
2. 执行 `/plan` 命令
3. 审查生成的计划
4. 根据需要修改计划
5. 确认计划

**预期结果**:
- 成功使用 `/plan` 命令
- 获得详细的实现计划
- 理解规划流程

**参考**:
- [/plan 命令说明](../../01-commands/01-plan/README.md)
- [/plan 练习](../../01-commands/01-plan/exercise.md)

### 练习 2: 使用 /tdd 实现任务标签功能 ⭐⭐

**目标**: 学习如何使用 `/tdd` 命令实现功能

**步骤**:
1. 使用 `/tdd` 命令
2. 按照指导编写测试（RED）
3. 实现功能（GREEN）
4. 重构优化（IMPROVE）
5. 验证测试覆盖率

**预期结果**:
- 成功使用 `/tdd` 命令
- 实现 Tag 实体和 Repository
- 实现 TagService 和 TagController
- 编写和通过所有测试
- 达到 80%+ 测试覆盖率

**参考**:
- [/tdd 命令说明](../../01-commands/02-tdd/README.md)
- [/tdd 练习](../../01-commands/02-tdd/exercise.md)
- [Spring Boot TDD 技能](../../03-skills/01-java/springboot-tdd/README.md)

### 练习 3: 使用 /code-review 审查代码 ⭐⭐

**目标**: 学习如何使用 `/code-review` 命令审查代码

**步骤**:
1. 使用 `/code-review` 命令
2. 审查任务标签功能的代码
3. 根据建议修复问题
4. 再次审查直到满意

**预期结果**:
- 成功使用 `/code-review` 命令
- 识别并修复代码问题
- 提高代码质量

**参考**:
- [/code-review 命令说明](../../01-commands/03-code-review/README.md)
- [code-reviewer 代理](../../02-agents/03-code-reviewer/README.md)

### 练习 4: 使用 /build-fix 修复错误 ⭐

**目标**: 学习如何使用 `/build-fix` 命令修复错误

**步骤**:
1. 故意引入一个编译错误
2. 运行 `mvn compile`
3. 使用 `/build-fix` 命令
4. 验证错误被修复

**预期结果**:
- 成功使用 `/build-fix` 命令
- 理解错误修复流程

**参考**:
- [/build-fix 命令说明](../../01-commands/05-build-fix/README.md)

### 练习 5: 使用 /e2e 生成端到端测试 ⭐⭐⭐

**目标**: 学习如何使用 `/e2e` 命令生成和运行端到端测试

**步骤**:
1. 使用 `/e2e` 命令
2. 生成任务标签功能的 E2E 测试
3. 运行测试验证功能

**预期结果**:
- 成功使用 `/e2e` 命令
- 生成并运行 E2E 测试
- 验证完整用户流程

**参考**:
- [/e2e 命令说明](../../01-commands/04-e2e/README.md)
- [e2e-runner 代理](../../02-agents/04-e2e-runner/README.md)

### 练习 6: 完整功能开发周期 ⭐⭐⭐

**目标**: 完整实践 ECC 工作流

**需求**: 实现任务统计功能（见上方"待实现功能"）

**步骤**:
1. `/plan` - 规划实现
2. `/tdd` - 测试驱动实现
3. `/code-review` - 代码审查
4. `/e2e` - 端到端测试
5. `git commit` - 提交代码

**预期结果**:
- 完整实践 ECC 工作流
- 实现任务统计功能
- 理解每个步骤的重要性

**参考**:
- [完整场景示例](../../05-scenarios/01-new-feature-development/)

## 学习路径

### 第一周：基础练习

1. **Day 1-2**: 完成 [练习 1](#练习-1-使用--plan-规划任务标签功能-)
2. **Day 3-4**: 完成 [练习 2](#练习-2-使用--tdd-实现任务标签功能-)
3. **Day 5**: 完成 [练习 3](#练习-3-使用--code-review-审查代码-)

### 第二周：进阶练习

1. **Day 1**: 完成 [练习 4](#练习-4-使用--build-fix-修复错误-)
2. **Day 2-3**: 完成 [练习 5](#练习-5-使用--e2e-生成端到端测试-)
3. **Day 4-5**: 完成 [练习 6](#练习-6-完整功能开发周期-)

## 开发规范

### 代码规范

- 遵循 [Java 编码标准](../../03-skills/01-java/java-coding-standards/README.md)
- 使用 4 空格缩进
- 方法不超过 50 行
- 类不超过 800 行
- 使用有意义的命名

### 测试规范

- 测试单元必须命名清晰
- 使用 AAA 模式（Arrange, Act, Assert）
- 测试覆盖率要求 80%+
- 遵循 [Spring Boot TDD 技能](../../03-skills/01-java/springboot-tdd/README.md)

### Git 规范

- 使用 Conventional Commits 格式
- 提交前确保测试通过
- 提交前使用 `/code-review` 审查代码

## 常见问题

**Q: 项目使用 Gradle 还是 Maven？**

A: 本项目使用 Maven。如果你更喜欢 Gradle，可以配置 Gradle 构建。

**Q: 数据库是什么？**

A: 开发环境使用 H2 内存数据库。生产环境可以切换到 PostgreSQL、MySQL 等。

**Q: 如何配置数据库？**

A: 修改 `src/main/resources/application.yml` 中的数据库配置。

**Q: 测试数据如何管理？**

A: 使用 `@DataJpaTest` 和 `@SpringBootTest` 注解，配合 `@BeforeEach` 设置测试数据。

**Q: 如何添加新的 API 端点？**

A: 参考现有的 Controller 实现，遵循相同的模式。

## 扩展建议

完成基础练习后，可以尝试：

1. **实现用户认证** - 参考 Spring Security 文档
2. **添加 Redis 缓存** - 优化查询性能
3. **集成消息队列** - 实现异步通知
4. **添加 Actuator** - 监控和管理
5. **配置 Docker** - 容器化部署

## 参考资源

- [Spring Boot 官方文档](https://spring.io/projects/spring-boot)
- [Spring Data JPA 文档](https://spring.io/projects/spring-data-jpa)
- [JUnit 5 文档](https://junit.org/junit5/)
- [ECC 学习路线图](../../学习路线图.md)

## 下一步

- 开始 [练习 1](#练习-1-使用--plan-规划任务标签功能-)
- 或查看 [学习路线图](../../学习路线图.md)

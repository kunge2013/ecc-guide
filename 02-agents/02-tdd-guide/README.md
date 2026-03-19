# tdd-guide 代理

## 概述

tdd-guide 代理是 ECC 的核心代理之一，专门负责指导测试驱动开发（TDD）流程。当需要编写新功能或修复bug时，tdd-guide 代理会被自动调用，确保代码通过TDD流程开发。

## 代理职责

tdd-guide 代理的主要职责包括：

1. **指导测试先行** - 在编写实现代码前先编写测试
2. **监控 TDD 循环** - 确保 RED → GREEN → IMPROVE 流程
3. **提供测试建议** - 提供测试用例编写建议
4. **验证测试覆盖** - 确保测试覆盖率达到要求
5. **重构指导** - 在测试通过后指导代码重构
6. **强制测试先行** - 拒绝直接编写实现代码的行为

## 触发条件

tdd-guide 代理在以下情况下被触发：

1. **用户执行 `/tdd` 命令** - 最常见的触发方式
2. **新功能开发** - 当用户要求添加新功能时自动触发
3. **Bug 修复** - 修复bug时自动触发
4. **planner 完成后** - planner 代理完成计划后自动调用

## 工作流程

### 标准 TDD 循环

```
1. RED（测试先行）
   ↓
2. 编写失败的测试
   ↓
3. 运行测试（应该失败）
   ↓
4. GREEN（最小实现）
   ↓
5. 编写最小实现代码
   ↓
6. 运行测试（应该通过）
   ↓
7. IMPROVE（重构优化）
   ↓
8. 重构代码，保持测试通过
   ↓
9. 重复循环
```

### 详细步骤

#### 步骤 1: RED 阶段 - 测试先行

**目标**: 编写一个失败的测试

**tdd-guide 指导**:
- 识别需要测试的功能点
- 设计测试用例（正常情况、边界情况、异常情况）
- 先写测试代码，不写实现代码
- 确保测试能够运行但失败

**示例场景**: 实现用户登录功能

**tdd-guide 建议的测试用例**:
```java
// UserServiceTest.java
@Test
void shouldLoginUserWithValidCredentials() {
    // Given
    String username = "testuser";
    String password = "password123";
    User user = new User(username, password);
    userRepository.save(user);

    // When
    LoginResult result = userService.login(username, password);

    // Then
    assertThat(result.isSuccess()).isTrue();
    assertThat(result.getUser()).isNotNull();
}

@Test
void shouldFailLoginWithInvalidPassword() {
    // Given
    String username = "testuser";
    String correctPassword = "password123";
    User user = new User(username, correctPassword);
    userRepository.save(user);

    // When
    LoginResult result = userService.login(username, "wrongpassword");

    // Then
    assertThat(result.isSuccess()).isFalse();
    assertThat(result.getErrorMessage()).contains("Invalid credentials");
}

@Test
void shouldFailLoginForNonexistentUser() {
    // When
    LoginResult result = userService.login("nonexistent", "password");

    // Then
    assertThat(result.isSuccess()).isFalse();
    assertThat(result.getErrorMessage()).contains("User not found");
}
```

**运行测试（RED）**:
```bash
mvn test -Dtest=UserServiceTest
# 结果: 3 tests failed (as expected - implementation doesn't exist yet)
```

#### 步骤 2: GREEN 阶段 - 最小实现

**目标**: 编写刚好能让测试通过的最小实现

**tdd-guide 指导**:
- 不要过度设计
- 只写能通过当前测试的代码
- 使用简单的硬编码值（如果有帮助）
- 保持代码简单直接

**示例实现**:
```java
// UserService.java
public LoginResult login(String username, String password) {
    Optional<User> userOpt = userRepository.findByUsername(username);

    if (userOpt.isEmpty()) {
        return LoginResult.failure("User not found");
    }

    User user = userOpt.get();
    if (!user.getPassword().equals(password)) {
        return LoginResult.failure("Invalid credentials");
    }

    return LoginResult.success(user);
}
```

**运行测试（GREEN）**:
```bash
mvn test -Dtest=UserServiceTest
# 结果: 3 tests passed ✓
```

#### 步骤 3: IMPROVE 阶段 - 重构优化

**目标**: 在保持测试通过的前提下优化代码

**tdd-guide 指导**:
- 提取重复代码
- 改善命名
- 应用设计模式
- 优化性能（如有需要）
- 保持测试绿色

**重构示例**:
```java
// UserServiceImpl - 重构后
public LoginResult login(String username, String password) {
    return userRepository.findByUsername(username)
        .map(user -> validateCredentials(user, password))
        .orElse(LoginResult.failure("User not found"));
}

private LoginResult validateCredentials(User user, String password) {
    if (!passwordEncoder.matches(password, user.getPassword())) {
        return LoginResult.failure("Invalid credentials");
    }
    return LoginResult.success(user);
}
```

**运行测试（IMPROVE 后）**:
```bash
mvn test -Dtest=UserServiceTest
# 结果: 3 tests passed ✓
```

## 使用场景

### 场景 1: 新功能开发

**用户输入**:
```
我需要添加用户注册功能
```

**tdd-guide 响应**:
```markdown
好的！让我们使用 TDD 方法实现用户注册功能。

## TDD 循环开始

### 阶段 1: RED - 编写测试

我先为用户注册编写测试用例。测试应该包括：
1. 有效数据注册成功
2. 重复用户名注册失败
3. 密码不符合要求失败
4. 邮箱格式验证

你准备好了吗？我将编写后运行测试（应该失败）。
```

### 场景 2: Bug 修复

**用户输入**:
```
用户报告一个 bug：当用户名包含特殊字符时，登录会失败
```

**tdd-guide 响应**:
```markdown
好的！让我们使用 TDD 方法修复这个 bug。

## TDD 循环开始

### 阶段 1: RED - 编写回归测试

首先，我需要编写一个测试来重现这个 bug：
- 测试用例：用户名包含特殊字符应该能正常登录

这确保我们：
1. 理解 bug 的具体表现
2. 有测试来防止回归
3. 有明确的修复目标

让我先编写这个测试...
```

### 场景 3: 重构现有代码

**用户输入**:
```
我想重构 OrderService 类，它变得太复杂了
```

**tdd-guide 响应**:
```markdown
好的！重构前确保有测试保护。

## 重构前检查

1. 检查现有测试覆盖率
2. 识别缺失的测试用例
3. 编写缺失的测试（如果需要）

让我们先运行现有测试，确保基线稳定...
```

## 代理特性

### 强制测试先行

tdd-guide 代理会：

- **检测直接编码** - 检测用户是否直接写实现代码
- **拒绝并指导** - 拒绝直接编码，指导先写测试
- **监控文件创建** - 确保测试文件先于实现文件创建

### 智能测试建议

tdd-guide 代理会：

- **识别测试边界** - 自动建议边界测试用例
- **异常处理测试** - 提醒编写异常场景测试
- **集成测试建议** - 根据架构建议需要集成测试
- **覆盖率目标** - 监控测试覆盖率是否达标

### 持续验证

tdd-guide 代理会：

- **验证测试运行** - 确保测试能够运行
- **验证 RED 状态** - 确保新测试初始失败
- **验证 GREEN 状态** - 确保实现后测试通过
- **验证重构安全** - 重构后确保测试仍然通过

## 与其他代理的协作

### 与 planner 协作

```
planner 生成实现计划
    ↓
用户确认计划
    ↓
tdd-guide 接手，指导 TDD 实现
```

### 与 code-reviewer 协作

```
tdd-guide 指导完成 TDD 实现
    ↓
code-reviewer 审查代码（包括测试质量）
```

### 与 language-reviewer 协作

```
tdd-guide 指导测试编写
    ↓
language-reviewer 审查测试代码质量（如 python-reviewer）
```

## 最佳实践

### 对于用户

1. **遵循 TDD 循环** - 严格按照 RED → GREEN → IMPROVE 执行
2. **相信 tdd-guide** - 代理的建议基于最佳实践
3. **不要跳过测试** - 每个功能都需要测试
4. **保持测试简单** - 测试应该简单、专注、快速
5. **及时重构** - 在测试保护下重构是安全的

### 测试覆盖率要求

- **最低要求**: 80% 测试覆盖率
- **推荐目标**: 90%+ 测试覆盖率
- **核心业务逻辑**: 100% 测试覆盖率

### 测试类型

tdd-guide 代理会指导编写以下测试类型：

1. **单元测试** - 测试单个函数/方法
2. **集成测试** - 测试组件间交互
3. **端到端测试** - 测试完整用户流程（配合 e2e-runner）

## 常见问题

### Q: 我可以先写实现代码再写测试吗？

A: **不可以**。tdd-guide 会强制测试先行，这是 TDD 的核心原则。

### Q: 测试失败但我知道怎么实现，可以跳过吗？

A: **不可以**。必须先看到测试失败（RED），然后编写实现让它通过（GREEN），这样才能验证测试的有效性。

### Q: 现有代码没有测试怎么办？

A: 使用 TDD 方法逐步添加测试。先为新功能写测试，然后为旧代码添加测试。

### Q: 测试很慢，可以不运行全部测试吗？

A: 可以只运行相关的测试，但最终要确保所有测试通过。使用测试框架的选择性运行功能。

## 学习资源

- [/tdd 命令说明](../../01-commands/02-tdd/README.md)
- [测试最佳实践](../../01-commands/02-tdd/testing-best-practices.md)
- [示例：用户注册 TDD 实现](./examples/user-registration-tdd.md)
- [示例：Bug 修复 TDD 流程](./examples/bug-fix-tdd.md)

## 下一步

- 查看 [用户注册 TDD 实现示例](./examples/user-registration-tdd.md)
- 或返回 [代理总览](../README.md)

# 示例：Bug 修复的 TDD 流程

这个示例展示如何使用 TDD 方法修复一个已知的 bug，确保不引入回归问题。

## Bug 描述

**Bug 报告**: 当用户名包含特殊字符（如 `@`、`#`、`$`）时，用户登录功能失败，即使用户名和密码都正确。

**重现步骤**:
1. 创建用户名为 `user@domain` 的用户
2. 使用正确的用户名和密码登录
3. 登录失败，显示 "用户名或密码错误"

**预期结果**: 应该允许特殊字符的用户名进行登录。

## TDD 循环 1: 编写回归测试

### RED 阶段

首先编写一个测试来重现这个 bug。这个测试将作为回归测试，防止 bug 再次出现。

```java
// UserServiceTest.java
@Test
@DisplayName("应该支持包含特殊字符的用户名登录")
void shouldLoginUserWithSpecialCharacters() {
    // Given
    String username = "user@domain";
    String password = "SecurePass123!";

    // 先创建用户
    User user = new User(username, passwordEncoder.encode(password), "user@domain.com");
    userRepository.save(user);

    // When
    LoginResult result = userService.login(username, password);

    // Then - 这个测试当前会失败（重现 bug）
    assertThat(result.isSuccess())
        .withFailMessage("用户 %s 应该能够登录", username)
        .isTrue();

    assertThat(result.getUser().getUsername())
        .isEqualTo(username);
}
```

**运行测试（RED）**:
```bash
$ mvn test -Dtest=UserServiceTest#shouldLoginUserWithSpecialCharacters

Tests run: 1, Failures: 1, Errors: 0, Skipped: 0

[ERROR] shouldLoginUserWithSpecialCharacters
[ERROR]   Expected: true
[ERROR]   Actual: false

✓ 测试失败，成功重现了 bug
```

### 分析失败原因

查看测试失败的原因，理解 bug 的根本原因：

```java
// UserServiceImpl.java（当前实现）
public LoginResult login(String username, String password) {
    String query = "SELECT * FROM users WHERE username = '" + username + "'";
    //                                ^^^^^^^^^^^^^^^^^^^^
    //                                这里直接拼接用户名

    User user = jdbcTemplate.queryForObject(query, userRowMapper);

    if (user == null) {
        return LoginResult.failure("用户名或密码错误");
    }

    if (!passwordEncoder.matches(password, user.getPassword())) {
        return LoginResult.failure("用户名或密码错误");
    }

    return LoginResult.success(user);
}
```

**问题分析**:
- 用户名中的 `@` 字符在 SQL 中有特殊含义
（虽然这里不是注入问题，但可能导致查询失败）
- 更重要的是，这种写法有 SQL 注入风险

### GREEN 阶段

修复 bug，使用参数化查询：

```java
// UserServiceImpl.java（修复后）
public LoginResult login(String username, String password) {
    // 使用参数化查询
    String query = "SELECT * FROM users WHERE username = ?";

    User user = jdbcTemplate.queryForObject(
        query,
        userRowMapper,
        username  // 作为参数传递，避免特殊字符问题
    );

    if (user == null) {
        return LoginResult.failure("用户名或密码错误");
    }

    if (!passwordEncoder.matches(password, user.getPassword())) {
        return LoginResult.failure("用户名或密码错误");
    }

    return LoginResult.success(user);
}
```

**运行测试（GREEN）**:
```bash
$ mvn test -Dtest=UserServiceTest#shouldLoginUserWithSpecialCharacters

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

✓ 测试通过，bug 已修复
```

### IMPROVE 阶段

重构代码，提高可读性和安全性：

```java
// UserServiceImpl.java（重构后）
public LoginResult login(String username, String password) {
    return userRepository.findByUsername(username)
        .map(user -> validateCredentials(user, password))
        .orElse(LoginResult.failure("用户名或密码错误"));
}

private LoginResult validateCredentials(User user, String password) {
    if (!passwordEncoder.matches(password, user.getPassword())) {
        return LoginResult.failure("用户名或密码错误");
    }
    return LoginResult.success(user);
}
```

**运行测试（IMPROVE 后）**:
```bash
$ mvn test -Dtest=UserServiceTest#shouldLoginUserWithSpecialCharacters

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

✓ 测试依然通过
```

## TDD 循环 2: 添加更多边界测试

虽然 bug 修复了，但我们应该添加更多测试用例，确保其他边界情况也被覆盖。

### RED 阶段

添加更多特殊字符测试：

```java
@Test
@DisplayName("应该支持各种特殊字符的用户名")
void shouldSupportVariousSpecialCharacters() {
    // Given
    List<String> usernames = Arrays.asList(
        "user@domain",
        "user#tag",
        "user$special",
        "user%test",
        "user&symbol",
        "user*star",
        "user/forward",
        "user\\backslash",
        "user|pipe",
        "user.question",
        "user.exclamation!",
        "user(parenthesis)"
    );

    String password = "SecurePass123!";

    for (String username : usernames) {
        // Given
        User user = new User(
            username,
            passwordEncoder.encode(password),
            username + "@example.com"
        );
        userRepository.save(user);

        // When
        LoginResult result = userService.login(username, password);

        // Then
        assertThat(result.isSuccess())
            .withFailMessage("用户 %s 应该能够登录", username)
            .isTrue();
    }
}

@Test
@DisplayName("应该拒绝 SQL 注入尝试")
void shouldRejectSQLInjectionAttempts() {
    // Given
    String maliciousUsername = "admin' OR '1'='1";
    String password = "anyPassword";

    // When
    LoginResult result = userService.login(maliciousUsername, password);

    // Then - 应该安全地拒绝，而不是执行注入的 SQL
    assertThat(result.isSuccess()).isFalse();
    assertThat(result.getErrorMessage()).contains("用户名或密码错误");
}
```

**运行测试（RED）**:
```bash
$ mvn test -Dtest=UserServiceTest

Tests run: 3, Failures: 2, Errors: 0, Skipped: 0

[ERROR] shouldSupportVariousSpecialCharacters
[ERROR]   Expected: all usernames should be supported
[ERROR]   Actual: some usernames still fail

[ERROR] shouldRejectSQLInjectionAttempts
[ERROR]   Expected: false (should fail)
[ERROR]   Actual: true (login succeeded - injection possible!)

✓ 发现了新问题：某些特殊字符仍然有问题，而且存在 SQL 注入风险！
```

### GREEN 阶段

彻底修复，确保安全性和完整性：

```java
// UserServiceImpl.java（彻底修复）
public LoginResult login(String username, String password) {
    // 验证用户名不为空
    if (username == null || username.trim().isEmpty()) {
        return LoginResult.failure("用户名不能为空");
    }

    // 限制用户名长度
    if (username.length() > 100) {
        return LoginResult.failure("用户名过长");
    }

    // 使用 JPA Repository（自动防止 SQL 注入）
    return userRepository.findByUsername(username)
        .map(user -> validateCredentials(user, password))
        .orElse(LoginResult.failure("用户名或密码错误"));
}

// UserRepository.java（使用 JPA，自动参数化查询）
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);

    // JPA 会自动将这个方法转换为参数化查询：
    // SELECT * FROM users WHERE username = ?
}
```

**运行测试（GREEN）**:
```bash
$ mvn test -Dtest=UserServiceTest

Tests run: 3, Failures: 0, Errors: 0, Skipped: 0

✓ 所有测试通过！
```

### IMPROVE 阶段

添加更多安全验证：

```java
// UserServiceImpl.java（增强安全性）
public LoginResult login(String username, String password) {
    // 输入验证
    List<String> validationErrors = validateLoginInput(username, password);
    if (!validationErrors.isEmpty()) {
        return LoginResult.failure(String.join(", ", validationErrors));
    }

    // 查找用户
    return userRepository.findByUsername(username)
        .map(user -> validateCredentials(user, password))
        .orElse(LoginResult.failure("用户名或密码错误"));
}

private List<String> validateLoginInput(String username, String password) {
    List<String> errors = new ArrayList<>();

    if (username == null || username.trim().isEmpty()) {
        errors.add("用户名不能为空");
    }

    if (password == null || password.trim().isEmpty()) {
        errors.add("密码不能为空");
    }

    if (username != null && username.length() > 100) {
        errors.add("用户名过长");
    }

    if (password != null && password.length() > 200) {
        errors.add("密码过长");
    }

    return errors;
}

private LoginResult validateCredentials(User user, String password) {
    // 使用安全的密码比较（防止时序攻击）
    if (!passwordEncoder.matches(password, user.getPassword())) {
        return LoginResult.failure("用户名或密码错误");
    }

    // 检查账户是否被锁定
    if (user.isLocked()) {
        return LoginResult.failure("账户已被锁定");
    }

    // 检查账户是否过期
    if (user.isExpired()) {
        return LoginResult.failure("账户已过期");
    }

    return LoginResult.success(user);
}
```

**运行测试（IMPROVE 后）**:
```bash
$ mvn test -Dtest=UserServiceTest

Tests run: 3, Failures: 0, Errors: 0, Skipped: 0

✓ 所有测试依然通过
```

## TDD 循环 3: 添加相关功能的测试

修复这个 bug 时，我们还应该检查相关功能是否也受影响。

### RED 阶段

测试用户创建和更新功能：

```java
@Test
@DisplayName("应该能够创建包含特殊字符的用户")
void shouldCreateUserWithSpecialCharacters() {
    // Given
    UserRegistrationRequest request = new UserRegistrationRequest(
        "user@domain",
        "SecurePass123!",
        "user@domain.com"
    );

    // When
    User user = userService.createUser(request);

    // Then
    assertThat(user).isNotNull();
    assertThat(user.getUsername()).isEqualTo("user@domain");
    assertThat(user.getEmail()).isEqualTo("user@domain.com");
}

@Test
@DisplayName("应该能够更新包含特殊字符的用户名")
void shouldUpdateUserWithSpecialCharacters() {
    // Given
    User user = new User("olduser", "password", "olduser@example.com");
    user = userRepository.save(user);

    // When
    user.setUsername("user@newdomain");
    User updatedUser = userRepository.save(user);

    // Then
    assertThat(updatedUser.getUsername()).isEqualTo("user@newdomain");

    // 验证可以用新用户名登录
    LoginResult result = userService.login("user@newdomain", "password");
    assertThat(result.isSuccess()).isTrue();
}
```

**运行测试（RED）**:
```bash
$ mvn test -Dtest=UserServiceTest

Tests run: 5, Failures: 1, Errors: 0, Skipped: 0

[ERROR] shouldUpdateUserWithSpecialCharacters
[ERROR]   Expected: true
[ERROR]   Actual: false

✓ 发现了另一个 bug：更新用户名后无法登录
```

### GREEN 阶段

修复更新功能的问题：

```java
// 问题分析：可能是因为用户名更新后索引没有正确更新

// 解决方案：确保 JPA 实体的正确配置
@Entity
@Table(name = "users", indexes = {
    @Index(name = "idx_username", columnList = "username", unique = true)
})
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "username", unique = true, nullable = false, length = 100)
    private String username;

    // ... 其他字段

    // 确保 equals 和 hashCode 正确实现
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof User)) return false;
        User user = (User) o;
        return id != null && id.equals(user.id);
    }

    @Override
    public int hashCode() {
        return getClass().hashCode();
    }
}
```

**运行测试（GREEN）**:
```bash
$ mvn test -Dtest=UserServiceTest

Tests run: 5, Failures: 0, Errors: 0, Skipped: 0

✓ 所有测试通过
```

## 最终结果

所有测试通过，bug 完全修复：

```bash
$ mvn test -Dtest=UserServiceTest

[INFO] Tests run: 5, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO] Results:
[INFO]
[INFO] Tests run: 5, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO] BUILD SUCCESS
```

## 总结

### Bug 修复的 TDD 方法总结

1. **编写回归测试** - 先写一个失败的测试来重现 bug
2. **修复 bug** - 让测试通过
3. **添加更多测试** - 确保边界情况也被覆盖
4. **重构代码** - 在测试保护下改善代码质量
5. **发现相关问题** - 测试揭示了其他潜在问题
6. **彻底修复** - 确保所有相关功能都正常

### TDD 修复 Bug 的优势

| 优势 | 说明 |
|------|------|
| **防止回归** | 测试会持续运行，防止 bug 再次出现 |
| **理解问题** | 编写测试帮助深入理解 bug 的根本原因 |
| **发现相关问题** | 测试可能揭示其他相关的问题 |
| **安全重构** | 有测试保护，可以安全地重构代码 |
| **文档价值** | 测试即文档，展示了期望的行为 |

### 关键要点

1. **永远先写测试** - 即使是修复 bug，也要先写测试
2. **测试应该失败** - 验证测试确实重现了 bug
3. **最小修复** - 先让测试通过，然后考虑重构
4. **全面测试** - 不要只测试 bug 场景，也要测试边界情况
5. **持续验证** - 修复后运行所有相关测试，确保没有引入新问题

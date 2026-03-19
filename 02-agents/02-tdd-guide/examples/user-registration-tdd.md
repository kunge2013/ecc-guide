# 示例：用户注册功能的 TDD 实现

这个示例展示如何使用 tdd-guide 代理实现一个完整的用户注册功能，遵循严格的 TDD 流程。

## 场景描述

为任务管理系统添加用户注册功能：
- 用户可以注册新账户
- 需要验证用户名、密码、邮箱格式
- 防止重复用户名和邮箱
- 注册成功后发送欢迎邮件

## TDD 循环 1：验证用户名

### RED 阶段

首先编写测试用例来验证用户名规则：

```java
// UserRegistrationServiceTest.java
@SpringBootTest
class UserRegistrationServiceTest {

    @Autowired
    private UserRegistrationService registrationService;

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private EmailService emailService;

    @Test
    @DisplayName("应该成功注册有效的用户")
    void shouldRegisterValidUser() {
        // Given
        RegistrationRequest request = new RegistrationRequest(
            "john_doe",
            "SecurePass123!",
            "john@example.com"
        );

        // When
        RegistrationResult result = registrationService.register(request);

        // Then
        assertThat(result.isSuccess()).isTrue();
        assertThat(result.getUser().getUsername()).isEqualTo("john_doe");

        // 验证用户被保存
        Optional<User> savedUser = userRepository.findByUsername("john_doe");
        assertThat(savedUserUser).isPresent();
    }

    @Test
    @DisplayName("应该拒绝空用户名")
    void shouldRejectEmptyUsername() {
        // Given
        RegistrationRequest request = new RegistrationRequest(
            "",  // 空用户名
            "SecurePass123!",
            "john@example.com"
        );

        // When
        RegistrationResult result = registrationService.register(request);

        // Then
        assertThat(result.isSuccess()).isFalse();
        assertThat(result.getErrors()).contains("用户名不能为空");
    }

    @Test
    @DisplayName("应该拒绝过短的用户名")
    void shouldRejectShortUsername() {
        // Given
        RegistrationRequest request = new RegistrationRequest(
            "ab",  // 少于3个字符
            "SecurePass123!",
            "john@example.com"
        );

        // When
        RegistrationResult result = registrationService.register(request);

        // Then
        assertThat(result.isSuccess()).isFalse();
        assertThat(result.getErrors()).contains("用户名至少需要3个字符");
    }

    @Test
    @DisplayName("应该拒绝过长的用户名")
    void shouldRejectLongUsername() {
        // Given
        String longUsername = "a".repeat(31);  // 超过30个字符
        RegistrationRequest request = new RegistrationRequest(
            longUsername,
            "SecurePass123!",
            "john@example.com"
        );

        // When
        RegistrationResult result = registrationService.register(request);

        // Then
        assertThat(result.isSuccess()).isFalse();
        assertThat(result.getErrors()).contains("用户名不能超过30个字符");
    }

    @Test
    @DisplayName("应该拒绝无效的用户名格式")
    void shouldRejectInvalidUsernameFormat() {
        // Given
        RegistrationRequest request = new RegistrationRequest(
            "user@name!",  // 包含无效字符
            "SecurePass123!",
            "john@example.com"
        );

        // When
        RegistrationResult result = registrationService.register(request);

        // Then
        assertThat(result.isSuccess()).isFalse();
        assertThat(result.getErrors()).contains("用户名只能包含字母、数字和下划线");
    }
}
```

**运行测试（RED）**:
```bash
$ mvn test -Dtest=UserRegistrationServiceTest

Tests run: 5, Failures: 5, Errors: 0, Skipped: 0

[INFO] UserRegistrationServiceTest
[INFO]   shouldRegisterValidUser ... FAILED
[INFO]   shouldRejectEmptyUsername ... FAILED
[INFO]   shouldRejectShortUsername ... FAILED
[INFO]   shouldRejectLongUsername ... FAILED
[INFO]   shouldRejectInvalidUsernameFormat ... FAILED
```

### GREEN 阶段

编写最小实现来通过测试：

```java
// UserRegistrationServiceImpl.java
@Service
public class UserRegistrationServiceImpl implements UserRegistrationService {

    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    private final EmailService emailService;

    // 构造函数注入
    public UserRegistrationServiceImpl(
            UserRepository userRepository,
            PasswordEncoder passwordEncoder,
            EmailService emailService) {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
        this.emailService = emailService;
    }

    @Override
    public RegistrationResult register(RegistrationRequest request) {
        List<String> errors = new ArrayList<>();

        // 验证用户名
        errors.addAll(validateUsername(request.getUsername()));

        if (!errors.isEmpty()) {
            return RegistrationResult.failure(errors);
        }

        // 创建用户
        User user = new User(
            request.getUsername(),
            passwordEncoder.encode(request.getPassword()),
            request.getEmail()
        );
        user.setCreatedAt(LocalDateTime.now());

        // 保存用户
        User savedUser = userRepository.save(user);

        // 发送欢迎邮件
        emailService.sendWelcomeEmail(savedUser);

        return RegistrationResult.success(savedUser);
    }

    private List<String> validateUsername(String username) {
        List<String> errors = new ArrayList<>();

        if (username == null || username.isEmpty()) {
            errors.add("用户名不能为空");
            return errors;
        }

        if (username.length() < 3) {
            errors.add("用户名至少需要3个字符");
        }

        if (username.length() > 30) {
            errors.add("用户名不能超过30个字符");
        }

        // 用户名只能包含字母、数字和下划线
        if (!username.matches("^[a-zA-Z0-9_]+$")) {
            errors.add("用户名只能包含字母、数字和下划线");
        }

        return errors;
    }
}
```

**运行测试（GREEN）**:
```bash
$ mvn test -Dtest=UserRegistrationServiceTest

Tests run: 5, Failures: 0, Errors: 0, Skipped: 0

[INFO] UserRegistrationServiceTest
[INFO]   shouldRegisterValidUser ... PASSED
[INFO]   shouldRejectEmptyUsername ... PASSED
[INFO]   shouldRejectShortUsername ... PASSED
[INFO]   shouldRejectLongUsername ... PASSED
[INFO]   shouldRejectInvalidUsernameFormat ... PASSED
```

### IMPROVE 阶段

重构代码，改善可读性和可维护性：

```java
// 重构后的版本
@Service
public class UserRegistrationServiceImpl implements UserRegistrationService {

    private static final int MIN_USERNAME_LENGTH = 3;
    private static final int MAX_USERNAME_LENGTH = 30;
    private static final Pattern USERNAME_PATTERN = Pattern.compile("^[a-zA-Z0-9_]+$");

    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    private final EmailService emailService;

    public UserRegistrationServiceImpl(
            UserRepository userRepository,
            PasswordEncoder passwordEncoder,
            EmailService emailService) {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
        this.emailService = emailService;
    }

    @Override
    public RegistrationResult register(RegistrationRequest request) {
        ValidationErrors errors = validateRegistrationRequest(request);

        if (errors.hasErrors()) {
            return RegistrationResult.failure(errors.getMessages());
        }

        User user = createUserFromRequest(request);
        User savedUser = userRepository.save(user);
        emailService.sendWelcomeEmail(savedUser);

        return RegistrationResult.success(savedUser);
    }

    private ValidationErrors validateRegistrationRequest(RegistrationRequest request) {
        ValidationErrors errors = new ValidationErrors();

        errors.addAll(validateUsername(request.getUsername()));
        errors.addAll(validatePassword(request.getPassword()));
        errors.addAll(validateEmail(request.getEmail()));

        return errors;
    }

    private List<String> validateUsername(String username) {
        if (isBlank(username)) {
            return List.of("用户名不能为空");
        }

        List<String> errors = new ArrayList<>();

        if (username.length() < MIN_USERNAME_LENGTH) {
            errors.add(String.format("用户名至少需要%d个字符", MIN_USERNAME_LENGTH));
        }

        if (username.length() > MAX_USERNAME_LENGTH) {
            errors.add(String.format("用户名不能超过%d个字符", MAX_USERNAME_LENGTH));
        }

        if (!USERNAME_PATTERN.matcher(username).matches()) {
            errors.add("用户名只能包含字母、数字和下划线");
        }

        return errors;
    }

    private User createUserFromRequest(RegistrationRequest request) {
        User user = new User(
            request.getUsername(),
            passwordEncoder.encode(request.getPassword()),
            request.getEmail()
        );
        user.setCreatedAt(LocalDateTime.now());
        return user;
    }
}
```

**运行测试（IMPROVE 后）**:
```bash
$ mvn test -Dtest=UserRegistrationServiceTest

Tests run: 5, Failures: 0, Errors: 0, Skipped: 0

所有测试依然通过 ✓
```

## TDD 循环 2：验证密码

### RED 阶段

添加密码验证测试：

```java
@Test
@DisplayName("应该拒绝空密码")
void shouldRejectEmptyPassword() {
    // Given
    RegistrationRequest request = new RegistrationRequest(
        "john_doe",
        "",  // 空密码
        "john@example.com"
    );

    // When
    RegistrationResult result = registrationService.register(request);

    // Then
    assertThat(result.isSuccess()).isFalse();
    assertThat(result.getErrors()).contains("密码不能为空");
}

@Test
@DisplayName("应该拒绝弱密码（少于8个字符）")
void shouldRejectWeakPasswordTooShort() {
    // Given
    RegistrationRequest request = new RegistrationRequest(
        "john_doe",
        "Short1!",  // 少于8个字符
        "john@example.com"
    );

    // When
    RegistrationResult result = registrationService.register(request);

    // Then
    assertThat(result.isSuccess()).isFalse();
    assertThat(result.getErrors()).contains("密码至少需要8个字符");
}

@Test
@DisplayName("应该拒绝弱密码（无数字）")
void shouldRejectWeakPasswordNoNumber() {
    // Given
    RegistrationRequest request = new RegistrationRequest(
        "john_doe",
        "Password!",  // 无数字
        "john@example.com"
    );

    // When
    RegistrationResult result = registrationService.register(request);

    // Then
    assertThat(result.isSuccess()).isFalse();
    assertThat(result.getErrors()).contains("密码必须包含至少一个数字");
}

@Test
@DisplayName("应该拒绝弱密码（无特殊字符）")
void shouldRejectWeakPasswordNoSpecialChar() {
    // Given
    RegistrationRequest request = new RegistrationRequest(
        "john_doe",
        "Password123",  // 无特殊字符
        "john@example.com"
    );

    // When
    RegistrationResult result = registrationService.register(request);

    // Then
    assertThat(result.isSuccess()).isFalse();
    assertThat(result.getErrors()).contains("密码必须包含至少一个特殊字符");
}
```

**运行测试（RED）**:
```bash
$ mvn test -Dtest=UserRegistrationServiceTest

Tests run: 9, Failures: 4, Errors: 0, Skipped: 0
```

### GREEN 阶段

实现密码验证：

```java
private List<String> validatePassword(String password) {
    if (isBlank(password)) {
        return List.of("密码不能为空");
    }

    List<String> errors = new ArrayList<>();

    if (password.length() < 8) {
        errors.add("密码至少需要8个字符");
    }

    if (!password.matches(".*\\d.*")) {
        errors.add("密码必须包含至少一个数字");
    }

    if (!password.matches(".*[!@#$%^&*(),.?\":{}|<>].*")) {
        errors.add("密码必须包含至少一个特殊字符");
    }

    return errors;
}
```

**运行测试（GREEN）**:
```bash
$ mvn test -Dtest=UserRegistrationServiceTest

Tests run: 9, Failures: 0, Errors: 0, Skipped: 0
```

## TDD 循环 3：验证邮箱

### RED 阶段

添加邮箱验证测试：

```java
@Test
@DisplayName("应该拒绝无效邮箱格式")
void shouldRejectInvalidEmailFormat() {
    // Given
    RegistrationRequest request = new RegistrationRequest(
        "john_doe",
        "SecurePass123!",
        "invalid-email"  // 无效邮箱
    );

    // When
    RegistrationResult result = registrationService.register(request);

    // Then
    assertThat(result.isSuccess()).isFalse();
    assertThat(result.getErrors()).contains("邮箱格式无效");
}

@Test
@DisplayName("应该接受有效的邮箱格式")
void shouldAcceptValidEmailFormats() {
    // Given
    List<String> validEmails = List.of(
        "test@example.com",
        "user.name@domain.co.uk",
        "user+tag@example.org"
    );

    for (String email : validEmails) {
        RegistrationRequest request = new RegistrationRequest(
            "john_doe_" + email.hashCode(),
            "SecurePass123!",
            email
        );

        // When
        RegistrationResult result = registrationService.register(request);

        // Then
        assertThat(result.isSuccess())
            .withFailMessage("邮箱 %s 应该有效", email)
            .isTrue();
    }
}
```

**运行测试（RED）**:
```bash
$ mvn test -Dtest=UserRegistrationServiceTest

Tests run: 11, Failures: 3, Errors: 0, Skipped: 0
```

### GREEN 阶段

实现邮箱验证：

```java
private List<String> validateEmail(String email) {
    if (isBlank(email)) {
        return List.of("邮箱不能为空");
    }

    // 使用正则表达式验证邮箱格式
    String emailRegex = "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$";
    if (!email.matches(emailRegex)) {
        return List.of("邮箱格式无效");
    }

    return Collections.emptyList();
}
```

**运行测试（GREEN）**:
```bash
$ mvn test -Dtest=UserRegistrationServiceTest

Tests run: 11, Failures: 0, Errors: 0, Skipped: 0
```

## TDD 循环 4：防止重复

### RED 阶段

添加重复用户名和邮箱测试：

```java
@Test
@DisplayName("应该拒绝重复的用户名")
void shouldRejectDuplicateUsername() {
    // Given
    RegistrationRequest firstRequest = new RegistrationRequest(
        "john_doe",
        "SecurePass123!",
        "john1@example.com"
    );
    registrationService.register(firstRequest);

    RegistrationRequest secondRequest = new RegistrationRequest(
        "john_doe",  // 重复的用户名
        "AnotherPass123!",
        "john2@example.com"
    );

    // When
    RegistrationResult result = registrationService.register(secondRequest);

    // Then
    assertThat(result.isSuccess()).isFalse();
    assertThat(result.getErrors()).contains("用户名已被使用");
}

@Test
@DisplayName("应该拒绝重复的邮箱")
void shouldRejectDuplicateEmail() {
    // Given
    RegistrationRequest firstRequest = new RegistrationRequest(
        "john_doe_1",
        "SecurePass123!",
        "john@example.com"
    );
    registrationService.register(firstRequest);

    RegistrationRequest secondRequest = new RegistrationRequest(
        "john_doe_2",
        "AnotherPass123!",
        "john@example.com"  // 重复的邮箱
    );

    // When
    RegistrationResult result = registrationService.register(secondRequest);

    // Then
    assertThat(result.isSuccess()).isFalse();
    assertThat(result.getErrors()).contains("邮箱已被使用");
}
```

**运行测试（RED）**:
```bash
$ mvn test -Dtest=UserRegistrationServiceTest

Tests run: 13, Failures: 2, Errors: 0, Skipped: 0
```

### GREEN 阶段

实现重复检查：

```java
@Override
public RegistrationResult register(RegistrationRequest request) {
    ValidationErrors errors = validateRegistrationRequest(request);

    if (errors.hasErrors()) {
        return RegistrationResult.failure(errors.getMessages());
    }

    // 检查重复
    errors.addAll(checkForDuplicates(request));

    if (errors.hasErrors()) {
        return RegistrationResult.failure(errors.getMessages());
    }

    User user = createUserFromRequest(request);
    User savedUser = userRepository.save(user);
    emailService.sendWelcomeEmail(savedUser);

    return RegistrationResult.success(savedUser);
}

private List<String> checkForDuplicates(RegistrationRequest request) {
    List<String> errors = new ArrayList<>();

    if (userRepository.existsByUsername(request.getUsername())) {
        errors.add("用户名已被使用");
    }

    if (userRepository.existsByEmail(request.getEmail())) {
        errors.add("邮箱已被使用");
    }

    return errors;
}
```

**运行测试（GREEN）**:
```bash
$ mvn test -Dtest=UserRegistrationServiceTest

Tests run: 13, Failures: 0, Errors: 0, Skipped: 0
```

## 最终结果

所有测试通过，功能完整实现：

```bash
$ mvn test -Dtest=UserRegistrationServiceTest

[INFO] Tests run: 13, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO] Results:
[INFO]
[INFO] Tests run: 13, Failures: 0, Errors: 0, Skipped: 0
[INFO]
[INFO] BUILD SUCCESS
```

### 测试覆盖率

```bash
$ mvn jacoco:report

[INFO] UserRegistrationServiceImpl
[INFO]   Line coverage: 95%
[INFO]   Branch coverage: 88%
[INFO]   Method coverage: 100%
```

## 关键要点

1. **严格的 RED → GREEN → IMPROVE 循环** - 每次只实现一个功能点
2. **测试先行** - 永远先写测试，再写实现
3. **小步前进** - 每次只处理一个验证规则
4. **持续重构** - 在测试保护下安全重构
5. **高覆盖率** - 通过 TDD 实现接近 100% 的测试覆盖率

## 与传统方法的对比

| 维度 | 传统方法 | TDD 方法 |
|-------|---------|----------|
| 测试编写时机 | 功能完成后 | 功能之前 |
| 发现 bug 时机 | 集成/测试阶段 | 开发阶段 |
| 重构信心 | 低（怕破坏功能） | 高（有测试保护） |
| 文档价值 | 无（需要额外文档） | 测试即文档 |
| 代码质量 | 依赖开发者经验 | 通过测试驱动质量 |

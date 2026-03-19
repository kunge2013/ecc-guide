# 示例：Spring Boot 代码审查

这个示例展示 code-reviewer 如何审查 Spring Boot 应用代码。

## 审查的代码

### 1. UserController.java

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @Autowired
    private UserService userService;  // 字段注入

    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.findById(id);  // 可能返回 null
    }

    @PostMapping
    public User createUser(@RequestBody User user) {
        return userService.save(user);  // 没有验证
    }

    @DeleteMapping("/{id}")
    public void deleteUser(@PathVariable Long id) {
        userService.deleteById(id);  // 没有权限检查
    }
}
```

### 2. UserService.java

```java
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    public User findById(Long id) {
        return userRepository.findById(id).orElse(null);  // 返回 null
    }

    public List<User> findByDepartment(Long departmentId) {
        List<User> users = new ArrayList<>();
        for (User user : userRepository.findAll()) {  // 低效查询
            if (user.getDepartmentId().equals(departmentId)) {
                users.add(user);
            }
        }
        return users;
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void updateUserWithLog(User user) {
        userRepository.save(user);
        logUserUpdate(user);  // 新事务可能导致数据不一致
    }
}
```

### 3. UserRepository.java

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    @Query("SELECT u FROM User u WHERE u.username = :username")
    User findByUsername(@Param("username") String username);  // 返回 User 而非 Optional
}
```

## 审查报告

# Spring Boot 代码审查报告

## 概览
- 审查的文件：3 个
- 发现的 CRITICAL 问题：2 个
- 发现的 HIGH 问题：5 个
- 发现的 MEDIUM 问题：3 个
- 发现的 LOW 问题：2 个

---

## 🔴 CRITICAL 问题

### 1. 缺少权限检查
**文件**: `UserController.java:23`
**等级**: CRITICAL
**问题**: 删除用户操作没有权限检查

```java
@DeleteMapping("/{id}")
public void deleteUser(@PathVariable Long id) {
    userService.deleteById(id);  // 任何人都可以删除用户！
}
```

**风险**: 未经授权的用户可以删除系统中的任何用户账户

**建议**: 添加权限检查：

```java
@DeleteMapping("/{id}")
@PreAuthorize("hasRole('ADMIN') or @userService.isCurrentUser(#id)")
public void deleteUser(@PathVariable Long id) {
    userService.deleteById(id);
}

// 或在 Service 层检查
@Service
public class UserService {
    public void deleteById(Long id, Long currentUserId) {
        User currentUser = findById(currentUserId);
        User userToDelete = findById(id);

        // 只有管理员或用户本人可以删除
        if (!currentUser.isAdmin() && !currentUser.getId().equals(id)) {
            throw new AccessDeniedException("无权删除此用户");
        }

        userRepository.deleteById(id);
    }
}
```

---

### 2. SQL 注入风险
**文件**: `UserRepository.java:8`
**等级**: CRITICAL
**问题**: 虽然使用了参数化查询，但返回类型不安全

```java
@Query("SELECT u FROM User u WHERE u.username = :username")
User findByUsername(@Param("username") String username);  // 返回 null 时会导致 NPE
```

**风险**: 如果用户不存在，返回 null 可能导致 NullPointerException

**建议**: 返回 Optional：

```java
@Query("SELECT u FROM User u WHERE u.username = :username")
Optional<User> findByUsername(@Param("username") String username);

// 使用时
userRepository.findByUsername(username)
    .ifPresentOrElse(
        user -> processUser(user),
        () -> throw new UserNotFoundException(username)
    );
```

---

## 🟠 HIGH 问题

### 1. 字段注入
**文件**: `UserController.java:7`
**等级**: HIGH
**问题**: 使用字段注入而非构造器注入

```java
@Autowired
private UserService userService;
```

**问题**:
- 无法设置 final 字段
- 难以编写单元测试（无法 mock）
- 依赖关系不明确
- 可能导致循环依赖

**建议**: 使用构造器注入：

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    // 或者使用 Lombok
    @RequiredArgsConstructor
    public class UserController {
        private final UserService userService;
    }
}
```

---

### 2. 可能返回 null
**文件**: `UserController.java:13`
**等级**: HIGH
**问题**: 端点可能返回 null

```java
@GetMapping("/{id}")
public User getUser(@PathVariable Long id) {
    return userService.findById(id);  // 如果用户不存在，返回 null
}
```

**问题**: 客户端收到 null 时可能出错

**建议**: 返回 ResponseEntity 或使用异常处理：

```java
@GetMapping("/{id}")
public ResponseEntity<User> getUser(@PathVariable Long id) {
    return userService.findById(id)
        .map(ResponseEntity::ok)
        .orElse(ResponseEntity.notFound().build());
}

// 或使用全局异常处理
@GetMapping("/{id}")
public User getUser(@PathVariable Long id) {
    return userService.findById(id)
        .orElseThrow(() -> new UserNotFoundException(id));
}

@ResponseStatus(HttpStatus.NOT_FOUND)
class UserNotFoundException extends RuntimeException {
    public UserNotFoundException(Long id) {
        super("User not found: " + id);
    }
}
```

---

### 3. 缺少输入验证
**文件**: `UserController.java:17`
**等级**: HIGH
**问题**: 创建用户时没有验证输入

```java
@PostMapping
public User createUser(@RequestBody User user) {
    return userService.save(user);  // 没有验证 user 的内容
}
```

**问题**: 可能创建无效的用户数据

**建议**: 使用 Bean Validation：

```java
@PostMapping
public ResponseEntity<User> createUser(@Valid @RequestBody User user) {
    User savedUser = userService.save(user);
    return ResponseEntity.status(HttpStatus.CREATED).body(savedUser);
}

// User 实体中添加验证注解
@Entity
public class User {
    @NotBlank(message = "用户名不能为空")
    @Size(min = 3, max = 50, message = "用户名长度必须在3-50之间")
    private String username;

    @NotBlank(message = "密码不能为空")
    @Size(min = 8, message = "密码至少8个字符")
    private String password;

    @Email(message = "邮箱格式无效")
    @NotBlank(message = "邮箱不能为空")
    private String email;
}

// 全局异常处理器
@ControllerAdvice
public class ValidationExceptionHandler {
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationException(
            MethodArgumentNotValidException ex) {

        List<String> errors = ex.getBindingResult()
            .getFieldErrors()
            .stream()
            .map(error -> error.getField() + ": " + error.getDefaultMessage())
            .collect(Collectors.toList());

        ErrorResponse response = new ErrorResponse(
            "验证失败",
            errors
        );
        return ResponseEntity.badRequest().body(response);
    }
}
```

---

### 4. 低效的查询
**文件**: `UserService.java:15`
**等级**: HIGH
**问题**: 加载所有用户然后在内存中过滤

```java
public List<User> findByDepartment(Long departmentId) {
    List<User> users = new ArrayList<>();
    for (User user : userRepository.findAll()) {  // 加载所有用户！
        if (user.getDepartmentId().equals(departmentId)) {
            users.add(user);
        }
    }
    return users;
}
```

**问题**:
- 性能差（加载所有数据）
- 内存消耗大
- 扩展性差

**建议**: 使用数据库查询：

```java
// 在 Repository 中添加查询方法
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    @Query("SELECT u FROM User u WHERE u.departmentId = :departmentId")
    List<User> findByDepartmentId(@Param("departmentId") Long departmentId);

    // 或使用方法名查询
    List<User> findByDepartmentId(Long departmentId);
}

// Service 中使用
public List<User> findByDepartment(Long departmentId) {
    return userRepository.findByDepartmentId(departmentId);
}
```

---

### 5. 事务传播配置错误
**文件**: `UserService.java:28`
**等级**: HIGH
**问题**: 使用 REQUIRES_NEW 导致数据不一致

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void updateUserWithLog(User user) {
    userRepository.save(user);
    logUserUpdate(user);  // 在新事务中记录日志
}
```

**问题**: 如果日志记录失败，用户更新已经提交，但日志丢失

**建议**: 使用默认传播级别或处理异常：

```java
// 选项 1: 使用默认传播级别（REQUIRED）
@Transactional
public void updateUserWithLog(User user) {
    userRepository.save(user);
    logUserUpdate(user);  // 在同一事务中
}

// 选项 2: 使用 REQUIRES_NEW 但处理异常
public void updateUserWithLog(User user) {
    userRepository.save(user);  // 在当前事务中

    try {
        logUserUpdateAsync(user);  // 异步记录日志
    } catch (Exception e) {
        // 日志失败不影响主业务流程
        log.error("Failed to log user update", e);
    }
}

@Transactional(propagation = Propagation.REQUIRES_NEW)
public void logUserUpdateAsync(User user) {
    // 异步日志记录
}
```

---

## 🟡 MEDIUM 问题

### 1. 缺少审计日志
**文件**: `UserService.java`
**等级**: MEDIUM
**问题**: 用户的创建、更新、删除操作没有审计日志

**建议**: 使用 Spring Data JPA Auditing：

```java
@Configuration
@EnableJpaAuditing
public class JpaAuditingConfig {
}

@Entity
@EntityListeners(AuditingEntityListener.class)
public class User {
    @CreatedDate
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;

    @CreatedBy
    @Column(name = "created_by", nullable = false, updatable = false)
    private String createdBy;

    @LastModifiedDate
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;

    @LastModifiedBy
    @Column(name = "updated_by")
    private String updatedBy;
}
```

---

### 2. 缺少缓存
**文件**: `UserService.java`
**等级**: MEDIUM
**问题**: 频繁查询的用户数据没有缓存

**建议**: 添加 Spring Cache：

```java
@Service
public class UserService {

    @Cacheable(value = "users", key = "#id")
    public User findById(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));
    }

    @CacheEvict(value = "users", key = "#user.id")
    public User save(User user) {
        return userRepository.save(user);
    }

    @CacheEvict(value = "users", key = "#id")
    public void deleteById(Long id) {
        userRepository.deleteById(id);
    }
}
```

---

### 3. 缺少分页支持
**文件**: `UserController.java`
**等级**: MEDIUM
**问题**: 列表查询没有分页支持

**建议**: 使用 Spring Data 分页：

```java
@GetMapping
public Page<User> getUsers(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size,
        @RequestParam(defaultValue = "id") String sortBy) {

    Pageable pageable = PageRequest.of(page, size, Sort.by(sortBy));
    return userService.findAll(pageable);
}

// Service
@Service
public class UserService {
    public Page<User> findAll(Pageable pageable) {
        return userRepository.findAll(pageable);
    }
}
```

---

## 总体评分: 5.5/10

## 优先修复建议

### 立即修复（CRITICAL）
1. ✅ 添加删除用户的权限检查
2. ✅ 修复 UserRepository 的返回类型

### 高优先级（HIGH）
1. ✅ 将字段注入改为构造器注入
2. ✅ 添加输入验证
3. ✅ 修复查询性能问题
4. ✅ 正确处理 null 返回值
5. ✅ 修复事务传播配置

### 中优先级（MEDIUM）
1. 添加审计日志
2. 添加缓存支持
3. 添加分页支持

## 测试覆盖率建议

| 类名 | 当前覆盖率 | 建议覆盖率 |
|------|-----------|-----------|
| UserController | 45% | 80% |
| UserService | 60% | 85% |
| UserRepository | 30% | 75% |

## 依赖项安全检查

| 依赖项 | 当前版本 | 最新版本 | 漏洞 |
|--------|----------|----------|------|
| spring-boot-starter-web | 2.5.0 | 2.7.0 | CVE-2022-22965 (HIGH) |
| spring-boot-starter-data-jpa | 2.5.0 | 2.7.0 | CVE-2022-22965 (HIGH) |
| spring-security-core | 5.5.0 | 5.7.0 | 无 |

**建议**: 升级 Spring Boot 到 2.7.0 或更高版本

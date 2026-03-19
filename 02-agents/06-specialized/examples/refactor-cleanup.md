# 示例：代码重构和清理

这个示例展示 refactor-cleaner 如何识别和清理死代码。

## 项目现状

### 目录结构

```
src/
├── main/
│   ├── java/
│   │   └── com/
│   │       └── example/
│   │           ├── controller/
│   │           │   ├── UserController.java
│   │           │   ├── TaskController.java
│   │           │   └── OldController.java  # 已废弃
│   │           ├── service/
│   │           │   ├── UserService.java
│   │           │   ├── TaskService.java
│   │           │   └── LegacyService.java  # 已废弃
│   │           ├── repository/
│   │           │   ├── UserRepository.java
│   │           │   └── TaskRepository.java
│   │           ├── util/
│   │           │   ├── StringUtils.java
│   │           │   ├── DateUtils.java
│   │           │   ├── UnusedUtils.java  # 未使用
│   │           │   └── DuplicateUtils.java  # 重复代码
│   │           └── constants/
│   │               └── Constants.java
│   └── resources/
│       ├── application.properties
│       └── legacy-config.xml  # 未使用
└── test/
    └── java/
        └── com/
            └── example/
                ├── UserControllerTest.java
                ├── TaskControllerTest.java
                └── OldTest.java  # 测试已废弃的代码
```

### 代码分析报告

# 代码重构和清理报告

## 执行概览
- 分析时间: 2024-03-19 17:00:00
- 扫描的文件: 25 个
- 识别的问题: 18 个
- 清理优先级: 🟠 HIGH

---

## 🔴 死代码

### 1. 未使用的类
**问题 ID**: DEAD-001
**文件**: `OldController.java`
**严重性**: CRITICAL
**类型**: 死代码

**问题描述**:
`OldController` 类没有任何引用，已经完全废弃。

**代码片段**:
```java
@RestController
@RequestMapping("/api/old")
public class OldController {
    // 这个类没有任何引用
}
```

**引用分析**:
- ✗ 没有其他类引用此类
- ✗ 没有测试类测试此类
- ✗ 没有配置文件引用此类

**建议**: 删除此文件

**删除命令**:
```bash
rm src/main/java/com/example/controller/OldController.java
```

---

### 2. 未使用的方法
**问题 ID**: DEAD-002
**文件**: `StringUtils.java:45`
**严重性**: HIGH
**类型**: 死代码

**问题描述**:
`StringUtils` 类中的 `reverseString()` 方法从未被使用。

**代码片段**:
```java
public class StringUtils {
    public static String reverseString(String str) {
        // 这个方法从未被调用
        return new StringBuilder(str).reverse().toString();
    }

    // 其他方法...
}
```

**引用分析**:
- ✗ 没有任何地方调用 `StringUtils.reverseString()`
- ✓ 类本身仍在使用

**建议**: 删除此方法或添加 `@Deprecated` 注解

**删除前考虑**:
- 是否在未来可能使用？
- 是否应该添加文档说明废弃原因？

---

### 3. 未使用的字段
**问题 ID**: DEAD-003
**文件**: `UserService.java:18`
**严重性**: MEDIUM
**类型**: 死代码

**问题描述**:
`UserService` 类中的 `legacyField` 字段从未被使用。

**代码片段**:
```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;

    private String legacyField;  // 从未使用

    // ...
}
```

**建议**: 删除此字段

---

### 4. 未使用的导入
**问题 ID**: DEAD-004
**文件**: `TaskController.java:5`
**严重性**: LOW
**类型**: 死代码

**问题描述**:
存在从未使用的导入语句。

**代码片段**:
```java
import com.example.util.StringUtils;  // 未使用
import com.example.util.DateUtils;
import java.util.List;
```

**建议**: 删除未使用的导入

**自动修复工具**:
- IntelliJ IDEA: `Ctrl + Alt + O` (优化导入）
- Eclipse: `Ctrl + Shift + O`
- VS Code: 使用插件 "Organize Imports"

---

### 5. 未使用的参数
**问题 ID**: DEAD-005
**文件**: `TaskService.java:32`
**严重性**: MEDIUM
**类型**: 死代码

**问题描述**:
`processTask()` 方法的 `deprecatedParam` 参数从未被使用。

**代码片段**:
```java
public void processTask(Long taskId, String deprecatedParam) {
    Task task = taskRepository.findById(taskId).orElseThrow(
        () -> new TaskNotFoundException(taskId)
    );

    // deprecatedParam 从未被使用

    task.setStatus("IN_PROGRESS");
    taskRepository.save(task);
}
```

**建议**: 删除此参数或使用 `@Deprecated` 标记

---

### 6. 废弃的测试
**问题 ID**: DEAD-006
**文件**: `OldTest.java`
**严重性**: HIGH
**类型**: 死代码

**问题描述**:
`OldTest` 测试类测试已废弃的 `OldController`。

**代码片段**:
```java
@SpringBootTest
class OldTest {
    @Test
    void testOldController() {
        // 测试已废弃的代码
    }
}
```

**建议**: 删除此测试类（与 `OldController` 一起删除）

---

## 🟠 重复代码

### 7. 重复的日期格式化
**问题 ID**: DUPL-001
**严重性**: MEDIUM
**类型**: 代码重复

**问题描述**:
日期格式化逻辑在多个地方重复。

**重复位置 1**:
```java
// UserService.java:25
private String formatDate(LocalDateTime date) {
    DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
    return date.format(formatter);
}
```

**重复位置 2**:
```java
// TaskService.java:40
private String formatDate(LocalDateTime date) {
    DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
    return date.format(formatter);
}
```

**重复位置 3**:
```java
// ReportService.java:60
private String formatDate(LocalDateTime date) {
    DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
    return date.format(formatter);
}
```

**建议**: 提取到公共工具类

**优化方案**:
```java
// DateUtils.java
public class DateUtils {
    private static final DateTimeFormatter DEFAULT_FORMATTER =
        DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

    public static String format(LocalDateTime date) {
        return date.format(DEFAULT_FORMATTER);
    }
}

// 使用
String formattedDate = DateUtils.format(date);
```

---

### 8. 重复的验证逻辑
**问题 ID**: DUPL-002
**严重性**: MEDIUM
**类型**: 代码重复

**问题描述**:
邮箱验证逻辑在多个地方重复。

**重复位置 1**:
```java
// UserController.java:15
private boolean isValidEmail(String email) {
    String emailRegex = "^[A-Za-z0-9+_.-]+@(.+\\.)+[A-Za-z]{2,}$";
    Pattern pattern = Pattern.compile(emailRegex);
    return pattern.matcher(email).matches();
}
```

**重复位置 2**:
```java
// TaskController.java:20
private boolean isValidEmail(String email) {
    String emailRegex = "^[A-Za-z0-9+_.-]+@(.+\\.)+[A-Za-z]{2,}$";
    Pattern pattern = Pattern.compile(emailRegex);
    return pattern.matcher(email).matches();
}
```

**建议**: 提取到公共工具类或使用 Bean Validation

**优化方案 1: 提取到工具类**
```java
// ValidationUtils.java
public class ValidationUtils {
    private static final Pattern EMAIL_PATTERN = Pattern.compile(
        "^[A-Za-z0-9+_.-]+@(.+\\.)+[A-Za-z]{2,}$"
    );

    public static boolean isValidEmail(String email) {
        return EMAIL_PATTERN.matcher(email).matches();
    }
}
```

**优化方案 2: 使用 Bean Validation**
```java
public class User {
    @Email(message = "邮箱格式无效")
    private String email;
}

public class Task {
    @Email(message = "邮箱格式无效")
    private String assigneeEmail;
}
```

---

## 🟡 复杂度问题

### 9. 方法复杂度过高
**问题 ID**: CPLX-001
**文件**: `TaskService.java:50`
**严重性**: HIGH
**类型**: 代码复杂度

**问题描述**:
`processComplexTask()` 方法的圈复杂度为 15，超过推荐的 10。

**代码片段**:
```java
public void processComplexTask(Long taskId, ComplexOptions options) {
    Task task = taskRepository.findById(taskId).orElseThrow(
        () -> new TaskNotFoundException(taskId)
    );

    if (task == null) {
        throw new TaskNotFoundException(taskId);
    }

    if (task.getStatus().equals("COMPLETED")) {
        return;
    }

    if (task.getStatus().equals("CANCELLED")) {
        return;
    }

    if (options.getNotifyAssignee()) {
        notifyUser(task.getAssigneeId());
    }

    if (options.getNotifyCreator()) {
        notifyUser(task.getCreatorId());
    }

    if (options.getNotifyTeam()) {
        notifyTeam(task.getProjectId());
    }

    if (options.getArchive()) {
        archiveTask(task);
    }

    if (options.getBackup()) {
        backupTask(task);
    }

    if (options.getAudit()) {
        auditTask(task);
    }

    // ... 更多条件
}
```

**复杂度分析**:
- 圈复杂度: 15
- 认知复杂度: 12
- 方法长度: 45 行

**建议**: 拆分为多个方法

**优化方案**:
```java
public void processComplexTask(Long taskId, ComplexOptions options) {
    Task task = findTaskOrThrow(taskId);
    checkTaskStatus(task);
    processNotifications(task, options);
    processTaskActions(task, options);
}

private Task findTaskOrThrow(Long taskId) {
    return taskRepository.findById(taskId)
        .orElseThrow(() -> new TaskNotFoundException(taskId));
}

private void checkTaskStatus(Task task) {
    String status = task.getStatus();

    if (status.equals("COMPLETED") || status.equals("CANCELLED")) {
        return;
    }
}

private void processNotifications(Task task, ComplexOptions options) {
    if (options.getNotifyAssignee()) {
        notifyUser(task.getAssigneeId());
    }

    if (options.getNotifyCreator()) {
        notifyUser(task.getCreatorId());
    }

    if (options.getNotifyTeam()) {
        notifyTeam(task.getProjectId());
    }
}

private void processTaskActions(Task task, ComplexOptions options) {
    if (options.getArchive()) {
        archiveTask(task);
    }

    if (options.getBackup()) {
        backupTask(task);
    }

    if (options.getAudit()) {
        auditTask(task);
    }
}
```

**优化后指标**:
| 指标 | 优化前 | 优化后 |
|------|--------|--------|
| 圈复杂度 | 15 | 3（主方法） |
| 方法长度 | 45 行 | 12 行（主方法） |
| 可测试性 | 低 | 高 |
| 可维护性 | 低 | 高 |

---

### 10. 类复杂度过高
**问题 ID**: CPLX-002
**文件**: `TaskController.java`
**严重性**: MEDIUM
**类型**: 代码复杂度

**问题描述**:
`TaskController` 类有 1500 行代码，超过推荐的 500 行。

**建议**: 拆分为多个控制器

**优化方案**:
```
TaskController (主控制器）
├── TaskQueryController (查询相关）
├── TaskCommandController (命令相关）
├── TaskAssignmentController (分配相关）
└── TaskCommentController (评论相关）
```

---

## 🟢 设计问题

### 11. 违反单一职责原则
**问题 ID**: SRP-001
**文件**: `UserService.java`
**严重性**: MEDIUM
**类型**: 设计问题

**问题描述**:
`UserService` 类同时负责用户管理、邮件发送和审计日志。

**代码片段**:
```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;

    @Autowired
    private EmailService emailService;

    @Autowired
    private AuditService auditService;

    // 用户管理
    public User createUser(User user) {
        // ...
    }

    // 邮件发送（应该分离）
    public void sendWelcomeEmail(User user) {
        // ...
    }

    // 审计日志（应该分离）
    public void auditUserAction(String action, User user) {
        // ...
    }
}
```

**建议**: 拆分为多个服务

**优化方案**:
```
UserService (用户管理）
├── UserManagementService (CRUD 操作）
├── UserNotificationService (邮件发送）
└── UserAuditService (审计日志）
```

---

### 12. 硬编码的值
**问题 ID**: HARD-001
**文件**: `TaskService.java:15`
**严重性**: LOW
**类型**: 设计问题

**问题描述**:
存在硬编码的字符串和数字。

**代码片段**:
```java
public class TaskService {
    private static final int MAX_TASKS_PER_USER = 10;  // 应该在配置中

    public void assignTask(Long userId, Long taskId) {
        int taskCount = taskRepository.countByUserId(userId);

        if (taskCount >= 10) {  // 硬编码
            throw new TaskLimitExceededException();
        }
    }

    public String generateTaskId() {
        return "TASK-" + System.currentTimeMillis();  // 硬编码前缀
    }
}
```

**建议**: 使用常量或配置

**优化方案**:
```java
@Value("${task.max-per-user:10}")
private int maxTasksPerUser;

@Value("${task.id.prefix:TASK-}")
private String taskIdPrefix;

public void assignTask(Long userId, Long taskId) {
    int taskCount = taskRepository.countByUserId(userId);

    if (taskCount >= maxTasksPerUser) {
        throw new TaskLimitExceededException();
    }
}

public String generateTaskId() {
    return taskIdPrefix + System.currentTimeMillis();
}
```

---

## 🔵 性能问题

### 13. 不必要的对象创建
**问题 ID**: PERF-001
**文件**: `DateUtils.java:8`
**严重性**: LOW
**类型**: 性能问题

**问题描述**:
在循环中重复创建 `SimpleDateFormat` 对象。

**代码片段**:
```java
public class DateUtils {
    public List<String> formatDates(List<Date> dates) {
        List<String> formatted = new ArrayList<>();

        for (Date date : dates) {
            DateFormat formatter = new SimpleDateFormat("yyyy-MM-dd");
            formatted.add(formatter.format(date));
        }

        return formatted;
    }
}
```

**问题**:
- `SimpleDateFormat` 不是线程安全的
- 每次循环都创建新对象

**建议**: 使用线程安全的格式化器或重用对象

**优化方案**:
```java
public class DateUtils {
    private static final DateTimeFormatter FORMATTER =
        DateTimeFormatter.ofPattern("yyyy-MM-dd");

    public List<String> formatDates(List<LocalDateTime> dates) {
        return dates.stream()
            .map(FORMATTER::format)
            .collect(Collectors.toList());
    }
}
```

---

## 清理优先级

### 第一优先级（立即清理）
1. ✅ 删除未使用的类 (`OldController.java`)
2. ✅ 删除废弃的测试 (`OldTest.java`)
3. ✅ 删除未使用的导入

### 第二优先级（本周清理）
1. ✅ 删除未使用的方法
2. ✅ 删除未使用的字段
3. ✅ 删除未使用的参数
4. ✅ 提取重复代码

### 第三优先级（本月清理）
1. 拆分高复杂度方法
2. 拆分大类
3. 应用单一职责原则
4. 移除硬编码值

## 自动清理工具

### Maven 插件

```xml
<plugin>
    <groupId>com.thoughtworks.plexus</groupId>
    <artifactId>plexus-compiler-api</artifactId>
    <version>2.8.8</version>
</plugin>

<!-- 死代码检测 -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-checkstyle-plugin</artifactId>
    <version>3.2.0</version>
    <configuration>
        <configLocation>google_checks.xml</configLocation>
    </configuration>
</plugin>

<!-- 重复代码检测 -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-pmd-plugin</artifactId>
    <version>3.19.0</version>
    <configuration>
        <rulesets>
            <ruleset>/rulesets/java/quickstart.xml</ruleset>
        </rulesets>
    </configuration>
</plugin>
```

### 运行清理命令

```bash
# 检测未使用代码（需要 IDE 或专用工具）
mvn clean install

# Checkstyle（代码风格检查）
mvn checkstyle:check

# PMD（重复代码检测）
mvn pmd:check

# SpotBugs（Bug 检测）
mvn spotbugs:check

# Jacoco（测试覆盖率）
mvn jacoco:report
```

## 清理后预期改进

| 指标 | 清理前 | 清理后 | 改进 |
|------|--------|--------|------|
| 总文件数 | 25 | 20 | -5 |
| 总代码行数 | 8,500 | 7,200 | -15% |
| 重复代码率 | 8.5% | 2.3% | -73% |
| 平均方法复杂度 | 6.2 | 4.1) | -34% |
| 未使用导入 | 15 | 0 | -100% |
| 测试覆盖率 | 65% | 78% | +20% |

## 后续步骤

1. **立即执行第一优先级清理**
2. **运行完整测试套件**
3. **审查并执行第二优先级清理**
4. **更新文档**
5. **建立代码审查流程防止技术债务累积**

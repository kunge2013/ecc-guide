# 使用场景：使用 Spring Boot 快速构建 REST API

这个示例展示如何结合使用 ECC 的技能和代理来快速构建一个符合最佳实践的 Spring Boot REST API。

## 场景描述

**任务**: 构建一个任务管理系统的 REST API

**功能要求**:
- 任务的 CRUD 操作
- 用户认证和授权
- 输入验证
- 错误处理
- 分页和排序
- 完整的测试覆盖

**技术栈**: Spring Boot 3.2 + Java 17 + PostgreSQL + JPA

## ECC 辅助开发流程

### 步骤 1: 项目初始化

```
用户: 创建一个新的 Spring Boot 项目，包含任务管理功能

ECC 检测到:
- 项目类型: Spring Boot (pom.xml)
- 激活技能: springboot-patterns, java-coding-standards

ECC 使用 springboot-patterns 技能建议:
```

根据 `springboot-patterns` 技能的建议，项目结构如下：

```
src/main/java/com/example/taskmanager/
├── controller/
│   ├── TaskController.java
│   └── UserController.java
├── service/
│   ├── TaskService.java
│   └── UserService.java
├── repository/
│   ├── TaskRepository.java
│   └── UserRepository.java
├── model/
│   ├── Task.java
│   └── User.java
├── dto/
│   ├── TaskDTO.java
│   ├── TaskCreateRequest.java
│   └── TaskUpdateRequest.java
├── exception/
│   ├── GlobalExceptionHandler.java
│   ├── ResourceNotFoundException.java
│   └── ValidationException.java
├── config/
│   ├── SecurityConfig.java
│   └── DatabaseConfig.java
└── TaskManagerApplication.java

src/main/resources/
├── application.yml
└── db/migration/
    └── V1__create_tables.sql

src/test/java/com/example/taskmanager/
├── controller/
│   ├── TaskControllerTest.java
│   └── UserControllerTest.java
└── service/
    ├── TaskServiceTest.java
    └── UserServiceTest.java
```

---

### 步骤 2: 实体类设计

根据 `java-coding-standards` 技能的指导：

```java
// Task.java
package com.example.taskmanager.model;

import jakarta.persistence.*;
import lombok.*;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.data.domain.Auditable;

@Entity
@Table(name = "tasks", indexes = {
    @Index(name = "idx_user_id", columnList = "user_id"),
    @Index(name = "idx_status", columnList = "status"),
    @Index(name = "idx_priority", columnList = "priority")
})
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Task {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "user_id", nullable = false)
    private Long userId;

    @Column(nullable = false, length = 200)
    private String title;

    @Column(columnDefinition = "TEXT")
    private String description;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 20)
    private TaskStatus status;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 10)
    private TaskPriority priority;

    @Column(name = "due_date")
    private LocalDate dueDate;

    @CreatedDate
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    @Column(name = "updated_at", nullable = false)
    private LocalDateTime updatedAt;
}

public enum TaskStatus {
    TODO, IN_PROGRESS, COMPLETED, CANCELLED
}

public enum TaskPriority {
    LOW, MEDIUM, HIGH, URGENT
}
```

**java-coding-standards 技能提供的建议**:
- ✅ 使用 Lombok 减少样板代码
- ✅ 添加数据库索引优化查询
- ✅ 使用枚举类型限制值范围
- ✅ 使用审计字段记录创建和更新时间
- ✅ 添加适当的列约束

---

### 步骤 3: Repository 层

根据 `springboot-patterns` 技能的指导：

```java
// TaskRepository.java
package com.example.taskmanager.repository;

import com.example.taskmanager.model.Task;
import com.example.taskmanager.model.TaskStatus;
import org.springframework.data.data.domain.Page;
import org.springframework.data.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

import java.time.LocalDate;

public interface TaskRepository extends JpaRepository<Task, Long> {

    // 基础查询方法（Spring Data JPA 自动生成）
    Page<Task> findByUserId(Long userId, Pageable pageable);
    Page<Task> findByUserIdAndStatus(
        Long userId,
        TaskStatus status,
        Pageable pageable
    );

    // 自定义查询（使用 @Query）
    @Query("""
        SELECT t FROM Task t
        WHERE t.userId = :userId
        AND (:status IS NULL OR t.status = :status)
        AND (:priority IS NULL OR t.priority = :priority)
        AND (:dueDateFrom IS NULL OR t.dueDate >= :dueDateFrom)
        AND (:dueDateTo IS NULL OR t.dueDate <= :dueDateTo)
        ORDER BY
            CASE WHEN :priority IS NULL THEN 0 ELSE 1 END,
            CASE t.priority
                WHEN URGENT THEN 1
                WHEN HIGH THEN 2
                WHEN MEDIUM THEN 3
                WHEN LOW THEN 4
            END
    """)
    Page<Task> findTasksWithFilters(
        @Param("userId") Long userId,
        @Param("status") TaskStatus status,
        @Param("priority") TaskPriority priority,
        @Param("dueDateFrom") LocalDate dueDateFrom,
        @Param("dueDateTo") LocalDate dueDateTo,
        Pageable pageable
    );

    // 统计查询
    long countByUserId(Long userId);
    long countByUserIdAndStatus(Long userId, TaskStatus status);
}
```

**springboot-patterns 技能提供的建议**:
- ✅ 使用 Spring Data JPA 方法命名约定
- ✅ 使用 @Query 编写复杂查询
- ✅ 支持分页和排序
- ✅ 使用参数化查询防止 SQL 注入

---

### 步骤 4: Service 层

结合 `springboot-patterns` 和 `java-coding-standards` 技能：

```java
// TaskService.java
package com.example.taskmanager.service;

import com.example.taskmanager.dto.*;
import com.example.taskmanager.exception.ResourceNotFoundException;
import com.example.taskmanager.model.Task;
import com.example.taskmanager.repository.TaskRepository;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.cache.annotation.*;
import org.springframework.data.data.domain.Page;
import org.springframework.data.data.domain.Pageable;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.time.LocalDate;

@Service
@RequiredArgsConstructor
@Slf4j
public class TaskService {

    private final TaskRepository taskRepository;

    @Cacheable(value = "tasks", key = "#id")
    public TaskDTO getTaskById(Long id) {
        log.debug("Fetching task with id: {}", id);

        Task task = taskRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Task not found: " + id));

        return mapToDTO(task);
    }

    @Cacheable(value = "userTasks", key = "#userId")
    public Page<TaskDTO> getTasksByUserId(
            Long userId,
            TaskFilterRequest filter,
            Pageable pageable) {

        log.debug("Fetching tasks for user: {}", userId);

        Page<Task> tasks = taskRepository.findTasksWithFilters(
            userId,
            filter.getStatus(),
            filter.getPriority(),
            filter.getDueDateFrom(),
            filter.getDueDateTo(),
            pageable
        );

        return tasks.map(this::mapToDTO);
    }

    @CacheEvict(value = "tasks", key = "#result.id")
    @CacheEvict(value = "userTasks", allEntries = true)
    @Transactional
    public TaskDTO createTask(Long userId, TaskCreateRequest request) {
        log.info("Creating task for user: {}", userId);

        Task task = Task.builder()
            .userId(userId)
            .title(request.getTitle())
            .description(request.getDescription())
            .status(TaskStatus.TODO)
            .priority(request.getPriority() != null ?
                request.getPriority() : TaskPriority.MEDIUM)
            .dueDate(request.getDueDate())
            .build();

        Task savedTask = taskRepository.save(task);
        log.info("Task created with id: {}", savedTask.getId());

        return mapToDTO(savedTask);
    }

    @CacheEvict(value = "tasks", key = "#id")
    @CacheEvict(value = "userTasks", allEntries = true)
    @Transactional
    public TaskDTO updateTask(Long id, Long userId, TaskUpdateRequest request) {
        log.info("Updating task: {} for user: {}", id, userId);

        Task task = taskRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Task not found: " + id));

        // 验证任务所有权
        if (!task.getUserId().equals(userId)) {
            throw new SecurityException("Unauthorized access to task: " + id);
        }

        // 更新字段
        if (request.getTitle() != null) {
            task.setTitle(request.getTitle());
        }
        if (request.getDescription() != null) {
            task.setDescription(request.getDescription());
        }
        if (request.getStatus() != null) {
            task.setStatus(request.getStatus());
        }
        if (request.getPriority() != null) {
            task.setPriority(request.getPriority());
        }
        if (request.getDueDate() != null) {
            task.setDueDate(request.getDueDate());
        }

        Task updatedTask = taskRepository.save(task);
        log.info("Task updated: {}", id);

        return mapToDTO(updatedTask);
    }

    @CacheEvict(value = "tasks", key = "#id")
    @CacheEvict(value = "userTasks", allEntries = true)
    @Transactional
    public void deleteTask(Long id, Long userId) {
        log.info("Deleting task: {} for user: {}", id, userId);

        Task task = taskRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("Task not found: " + id));

        // 验证任务所有权
        if (!task.getUserId().equals(userId)) {
            throw new SecurityException("Unauthorized access to task: " + id);
        }

        taskRepository.delete(task);
        log.info("Task deleted: {}", id);
    }

    private TaskDTO mapToDTO(Task task) {
        return TaskDTO.builder()
            .id(task.getId())
            .userId(task.getUserId())
            .title(task.getTitle())
            .description(task.getDescription())
            .status(task.getStatus())
            .priority(task.getPriority())
            .dueDate(task.getDueDate())
            .createdAt(task.getCreatedAt())
            .updatedAt(task.getUpdatedAt())
            .build();
    }
}
```

**技能建议**:
- ✅ 使用 @RequiredArgsConstructor 进行构造器注入
- ✅ 使用 @Slf4j 添加日志
- ✅ 使用 @Cacheable 添加缓存
- ✅ 使用 @Transactional 管理事务
- ✅ 验证数据所有权
- ✅ 使用 Builder 模式创建对象
- ✅ 添加适当的日志级别

---

### 步骤 5: Controller 层

根据 `springboot-patterns` 技能：

```java
// TaskController.java
package com.example.taskmanager.controller;

import com.example.taskmanager.dto.*;
import com.example.taskmanager.service.TaskService;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.data.data.domain.Page;
import org.springframework.data.data.domain.Pageable;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.security.core.annotation.AuthenticationPrincipal;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/v1/tasks")
@RequiredArgsConstructor
@CrossOrigin(origins = "*", maxAge = 3600)
public class TaskController {

    private final TaskService taskService;

    @GetMapping("/{id}")
    public ResponseEntity<TaskDTO> getTask(
            @PathVariable Long id,
            @AuthenticationPrincipal UserDetails userDetails) {

        Long userId = extractUserId(userDetails);
        TaskDTO task = taskService.getTaskById(id);

        // 验证所有权
        if (!task.getUserId().equals(userId)) {
            return ResponseEntity.status(HttpStatus.FORBIDDEN).build();
        }

        return ResponseEntity.ok(task);
    }

    @GetMapping
    public ResponseEntity<Page<TaskDTO>> getTasks(
            @RequestParam(required = false) TaskStatus status,
            @RequestParam(required = false) TaskPriority priority,
            @RequestParam(required = false) LocalDate dueDateFrom,
            @RequestParam(required = false) LocalDate dueDateTo,
            Pageable pageable,
            @AuthenticationPrincipal UserDetails userDetails) {

        Long userId = extractUserId(userDetails);

        TaskFilterRequest filter = TaskFilterRequest.builder()
            .status(status)
            .priority(priority)
            .dueDateFrom(dueDateFrom)
            .dueDateTo(dueDateTo)
            .build();

        Page<TaskDTO> tasks = taskService.getTasksByUserId(userId, filter, pageable);
        return ResponseEntity.ok(tasks);
    }

    @PostMapping
    public ResponseEntity<TaskDTO> createTask(
            @Valid @RequestBody TaskCreateRequest request,
            @AuthenticationPrincipal UserDetails userDetails) {

        Long userId = extractUserId(userDetails);
        TaskDTO createdTask = taskService.createTask(userId, request);

        return ResponseEntity
            .status(HttpStatus.CREATED)
            .body(createdTask);
    }

    @PutMapping("/{id}")
    public ResponseEntity<TaskDTO> updateTask(
            @PathVariable Long id,
            @Valid @RequestBody TaskUpdateRequest request,
            @AuthenticationPrincipal UserDetails userDetails) {

        Long userId = extractUserId(userDetails);
        TaskDTO updatedTask = taskService.updateTask(id, userId, request);

        return ResponseEntity.ok(updatedTask);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteTask(
            @PathVariable Long id,
            @AuthenticationPrincipal UserDetails userDetails) {

        Long userId = extractUserId(userDetails);
        taskService.deleteTask(id, userId);

        return ResponseEntity.noContent().build();
    }

    private Long extractUserId(UserDetails userDetails) {
        // 从 JWT token 中提取用户 ID
        return Long.parseLong(userDetails.getUsername());
    }
}
```

**springboot-patterns 技能提供的建议**:
- ✅ 使用 @Valid 进行 Bean Validation
- ✅ 使用 @AuthenticationPrincipal 获取当前用户
- ✅ 使用 RESTful HTTP 方法
- ✅ 返回适当的 HTTP 状态码
- ✅ 使用 @CrossOrigin 支持 CORS
- ✅ 支持 JSON 和 XML 响应

---

### 步骤 6: 使用 TDD 技能编写测试

```
用户: 我需要为 TaskService 编写测试

ECC 自动激活: springboot-tdd 技能
```

根据 `springboot-tdd` 技能的指导：

```java
// TaskServiceTest.java
package com.example.taskmanager.service;

import com.example.taskmanager.dto.*;
import com.example.taskmanager.exception.ResourceNotFoundException;
import com.example.taskmanager.model.Task;
import com.example.taskmanager.model.TaskStatus;
import com.example.taskmanager.model.TaskPriority;
import com.example.taskmanager.repository.TaskRepository;
import org.junit.jupiter.api.*;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import org.springframework.data.data.domain.Page;
import org.springframework.data.data.domain.PageImpl;
import org.springframework.data.data.domain.PageRequest;
import org.springframework.data.data.domain.Pageable;

import java.time.LocalDate;
import java.time.LocalDateTime;
import java.util.Collections;

import static org.assertj.core.api.Assertions.*;
import static org.mockito.ArgumentMatchers.*;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class TaskServiceTest {

    @Mock
    private TaskRepository taskRepository;

    @InjectMocks
    private TaskService taskService;

    private Task testTask;
    private Long testUserId = 1L;
    private Long testTaskId = 1L;

    @BeforeEach
    void setUp() {
        testTask = Task.builder()
            .id(testTaskId)
            .userId(testUserId)
            .title("Test Task")
            .description("Test Description")
            .status(TaskStatus.TODO)
            .priority(TaskPriority.MEDIUM)
            .dueDate(LocalDate.now().plusWeeks(1))
            .createdAt(LocalDateTime.now())
            .updatedAt(LocalDateTime.now())
            .build();
    }

    @Test
    @DisplayName("应该成功获取任务")
    void shouldGetTaskById() {
        // Given
        when(taskRepository.findById(testTaskId))
            .thenReturn(java.util.Optional.of(testTask));

        // When
        TaskDTO result = taskService.getTaskById(testTaskId);

        // Then
        assertThat(result).isNotNull();
        assertThat(result.getId()).isEqualTo(testTaskId);
        assertThat(result.getTitle()).isEqualTo("Test Task");
        assertThat(result.getUserId()).isEqualTo(testUserId);
        verify(taskRepository, times(1)).findById(testTaskId);
    }

    @Test
    @DisplayName("当任务不存在时应该抛出异常")
    void shouldThrowExceptionWhenTaskNotFound() {
        // Given
        when(taskRepository.findById(anyLong()))
            .thenReturn(java.util.Optional.empty());

        // When & Then
        assertThatThrownBy(() -> taskService.getTaskById(testTaskId))
            .isInstanceOf(ResourceNotFoundException.class)
            .hasMessageContaining("Task not found");
    }

    @Test
    @DisplayName("应该成功创建任务")
    void shouldCreateTask() {
        // Given
        TaskCreateRequest request = TaskCreateRequest.builder()
            .title("New Task")
            .description("New Description")
            .priority(TaskPriority.HIGH)
            .dueDate(LocalDate.now().plusDays(7))
            .build();

        Task createdTask = Task.builder()
            .id(2L)
            .userId(testUserId)
            .title(request.getTitle())
            .description(request.getDescription())
            .status(TaskStatus.TODO)
            .priority(request.getPriority())
            .dueDate(request.getDueDate())
            .createdAt(LocalDateTime.now())
            .updatedAt(LocalDateTime.now())
            .build();

        when(taskRepository.save(any(Task.class)))
            .thenReturn(createdTask);

        // When
        TaskDTO result = taskService.createTask(testUserId, request);

        // Then
        assertThat(result).isNotNull();
        assertThat(result.getId()).isEqualTo(2L);
        assertThat(result.getTitle()).isEqualTo("New Task");
        assertThat(result.getStatus()).isEqualTo(TaskStatus.TODO);
        verify(taskRepository, times(1)).save(any(Task.class));
    }

    @Test
    @DisplayName("应该成功更新任务")
    void shouldUpdateTask() {
        // Given
        when(taskRepository.findById(testTaskId))
            .thenReturn(java.util.Optional.of(testTask));

        TaskUpdateRequest request = TaskUpdateRequest.builder()
            .title("Updated Title")
            .status(TaskStatus.IN_PROGRESS)
            .build();

        Task updatedTask = Task.builder()
            .id(testTaskId)
            .userId(testUserId)
            .title(request.getTitle())
            .description(testTask.getDescription())
            .status(request.getStatus())
            .priority(testTask.getPriority())
            .dueDate(testTask.getDueDate())
            .createdAt(testTask.getCreatedAt())
            .updatedAt(LocalDateTime.now())
            .build();

        when(taskRepository.save(any(Task.class)))
            .thenReturn(updatedTask);

        // When
        TaskDTO result = taskService.updateTask(testTaskId, testUserId, request);

        // Then
        assertThat(result).isNotNull();
        assertThat(result.getTitle()).isEqualTo("Updated Title");
        assertThat(result.getStatus()).isEqualTo(TaskStatus.IN_PROGRESS);
        verify(taskRepository, times(1)).save(any(Task.class));
    }

    @Test
    @DisplayName("应该成功删除任务")
    void shouldDeleteTask() {
        // Given
        when(taskRepository.findById(testTaskId))
            .thenReturn(java.util.Optional.of(testTask));

        doNothing().when(taskRepository).delete(any(Task.class));

        // When
        taskService.deleteTask(testTaskId, testUserId);

        // Then
        verify(taskRepository, times(1)).delete(testTask);
    }

    @Test
    @DisplayName("应该分页获取任务")
    void shouldGetTasksByPage() {
        // Given
        TaskFilterRequest filter = new TaskFilterRequest();
        Pageable pageable = PageRequest.of(0, 10);
        Page<Task> taskPage = new PageImpl<>(
            Collections.singletonList(testTask),
            pageable,
            1
        );

        when(taskRepository.findTasksWithFilters(
            eq(testUserId),
            isNull(),
            isNull(),
            isNull(),
            isNull(),
            eq(pageable)
        )).thenReturn(taskPage);

        // When
        Page<TaskDTO> result = taskService.getTasksByUserId(
            testUserId, filter, pageable
        );

        // Then
        assertThat(result).isNotNull();
        assertThat(result.getContent()).hasSize(1);
        assertThat(result.getContent().get(0).getId()).isEqualTo(testTaskId);
        verify(taskRepository, times(1)).findTasksWithFilters(
            eq(testUserId),
            isNull(),
            isNull(),
            isNull(),
            isNull(),
            eq(pageable)
        );
    }
}
```

**springboot-tdd 技能提供的建议**:
- ✅ 使用 JUnit 5 和 AssertJ
- ✅ 使用 @Mock 和 @InjectMocks
- ✅ 使用 Given-When-Then 模式
- ✅ 测试所有分支和边界条件
- ✅ 验证 mock 调用
- ✅ 使用有意义的测试名称

---

### 步骤 7: 自动代码审查

```
用户: 审查我编写的代码

ECC 自动调用: code-reviewer 代理
ECC 使用技能: java-coding-standards, springboot-patterns
```

**code-reviewer 使用技能后的审查报告**:

```markdown
# Spring Boot 代码审查报告

## ✅ 符合规范的部分

### 1. 依赖注入
- ✓ 使用构造器注入（@RequiredArgsConstructor）
- ✓ 没有使用字段注入

### 2. 事务管理
- ✓ Service 方法正确使用 @Transactional
- ✓ 避免了长事务

### 3. 异常处理
- ✓ 使用自定义异常类
- ✓ 提供了有意义的错误消息

### 4. 缓存策略
- ✓ 使用 @Cacheable 和 @CacheEvict
- ✓ 缓存键设计合理

### 5. 日志记录
- ✓ 使用 SLF4J
- ✓ 日志级别使用正确
- ✓ 日志消息包含关键信息

## ⚠️ 需要改进的部分

### 1. 分页查询优化
**文件**: TaskController.java:45

建议添加最大页面大小限制：
```java
@GetMapping
public ResponseEntity<Page<TaskDTO>> getTasks(
        ...
        Pageable pageable,
        ...) {

    // 限制最大页面大小
    if (pageable.getPageSize() > 100) {
        pageable = PageRequest.of(
            pageable.getPageNumber(),
            100,
            pageable.getSort()
        );
    }

    ...
}
```

### 2. 输入验证
**文件**: TaskController.java:75

建议添加更多验证规则：
```java
@NotBlank(message = "标题不能为空")
@Size(min = 1, max = 200, message = "标题长度必须在1-200之间")
private String title;

@Size(max = 1000, message = "描述长度不能超过1000字符")
private String description;
```

### 3. 安全性
**文件**: TaskController.java:35

建议添加速率限制：
```java
@RateLimiter(name = "taskApi", fallbackMethod = "rateLimitFallback")
@GetMapping("/{id}")
public ResponseEntity<TaskDTO> getTask(...) {
    ...
}
```

## 总体评分: 8.5/10

代码质量优秀，遵循了 Spring Boot 最佳实践！
```

---

## 使用 ECC 技能和代理的优势

| 传统开发 | 使用 ECC |
|----------|---------|
| 手动搭建项目结构 | springboot-patterns 自动生成结构 |
| 查阅文档编写代码 | java-coding-standards 实时提供指导 |
| 手动编写测试 | springboot-tdd 生成测试模板 |
| 提交代码后人工审查 | code-reviewer 自动审查代码 |
| 迭代次数多 | 一次完成质量代码 |

## 关键要点

1. **技能自动激活** - ECC 检测项目类型自动加载相关技能
2. **最佳实践指导** - 技能提供行业最佳实践
3. **代理协作** - 多个代理协同工作
4. **持续改进** - 自动化审查和学习
5. **提高效率** - 减少重复工作，专注业务逻辑

## 扩展场景

### 添加更多功能

```
用户: 添加任务评论功能

ECC 自动:
1. 激活 springboot-patterns 技能
2. 使用 java-coding-standards 指导
3. 生成 CommentController、CommentService、CommentRepository
4. 生成测试模板
```

### 性能优化

```
用户: 优化查询性能

ECC 使用 skills:
- springboot-patterns: 建议使用 @EntityGraph
- postgres-patterns: 建议添加索引
```

### 安全加固

```
用户: 添加安全措施

ECC 使用 security-review 代理:
- 检测 CSRF 保护
- 建议使用 JWT
- 审计权限检查
```
